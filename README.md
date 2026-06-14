<h1 align="center">Hi, I'm Bhavin Patil</h1>
<h3 align="center">Full Stack Engineer · AWS Certified Solutions Architect · System Design</h3>

<p align="center">
  <a href="https://www.github.com/hackzbhavin" target="_blank">GitHub</a> ·
  <a href="https://www.linkedin.com/in/bhavin-patil-413a601a0" target="_blank">LinkedIn</a> ·
  <a href="mailto:bhavinpatil28@gmail.com">Email</a>
</p>

***

### About me

Full Stack Engineer with 4+ years building scalable, high-performance cloud-native SaaS applications using React, TypeScript, NestJS, Go, and AWS.

I focus on the gap between **working code and production-grade architecture** — system design, performance under load, observability, and cost-efficient infrastructure.

- Designed and shipped production systems for global SaaS platforms.
- Improved API performance through query optimization, caching, and better data models.
- Reduced AWS infrastructure costs and led microservices and event-driven backend initiatives.

***

### What I work on

- **Scalable backends & APIs** — NestJS/Node.js and Go services on AWS (Lambda, EC2, S3, DynamoDB, SQS, SNS).
- **System design & distributed systems** — microservices, event-driven architecture, async queues, high-availability design.
- **Load engineering & observability** — k6 load testing, Prometheus, Grafana, diagnosing bottlenecks at the DB/cache/app layer.
- **Performance & cost optimization** — query tuning, caching strategies, connection pool management, infra right-sizing.
- **Web & mobile frontends** — React, Next.js, and React Native for SaaS dashboards and cross-platform apps.

***

### Architecture Case Studies

> Repos built to study real problems. Not tutorials — actual systems with load tests, failure modes, and architectural trade-offs.

---

#### 📉 [error-logger-naive-to-production](https://github.com/hackzbhavin/error-logger-naive-to-production)
A synchronous error logging system — built the way most engineers write their first version.  
Intentionally designed to break under load to study **DB pool exhaustion, race conditions, and synchronous hot path bottlenecks**.  
Benchmarked with k6. Observed in Grafana. p95 hits ~1800ms at 500 VUs before the system falls over.

`NestJS` `MySQL` `TypeORM` `k6` `Prometheus` `Grafana`

#### ⚡ [high-throughput-error-ingestion](https://github.com/hackzbhavin/high-throughput-error-ingestion)
Async error deduplication pipeline built as the architected answer to the same problem.  
Redis handles atomic counting. BullMQ decouples the DB write from the request path. Cron syncs counts to MySQL in batch.  
Same load test: p95 drops to ~8ms at 500 VUs.

`NestJS` `Redis` `BullMQ` `MySQL` `TypeORM` `k6` `Prometheus` `Grafana` `cAdvisor`

> **Takeaway:** the gap between these two repos is not about more code — it is about understanding which parts of a system need to be fast and which parts just need to eventually be correct.

---

#### 🔒 [go-throttle](https://github.com/hackzbhavin/go-throttle)
Production-grade, per-entity throttling written in Go — built after studying how Shopify and Netflix actually handle this at scale.

The hard constraint: no API gateway. Everything enforced at the app layer, per `entity_id`, so one customer being throttled has zero effect on others.

The interesting problem was the **multi-server fallback** — when Redis goes down, two in-memory stores diverge and silently double your effective limit. The fix is a 3-mode state machine:

```
REDIS ──(3 failures)──► PEER SYNC ──(5 failures)──► LOCAL / NODE_COUNT
  ▲                          │                              │
  └──────── Redis ping ok ───┴──────────────────────────────┘
```

Three modes, each with a clear job:

- **REDIS** — Lua atomic token bucket. One source of truth. No race between servers.
- **PEER** — Nodes gossip snapshots every 100ms over TCP. Minimum token count wins. Max overshoot window: one gossip cycle.
- **LOCAL** — Both Redis and peers are unreachable. Each node enforces `limit / node_count`. Accepts halved capacity in exchange for zero downtime. Flushes to MySQL every 5s so counters survive restarts.

The token bucket itself is straightforward Go — a mutex-protected map, refill on read, no goroutines needed per entity:

```go
type Bucket struct {
    mu         sync.Mutex
    limit      int
    refillRate float64 // tokens per second
    buckets    map[string]*entry
}

type entry struct {
    tokens     float64
    lastRefill time.Time
}

func (b *Bucket) Consume(entityID string, cost int) (allowed bool, remaining int) {
    b.mu.Lock()
    defer b.mu.Unlock()

    now := time.Now()
    e, ok := b.buckets[entityID]
    if !ok {
        e = &entry{tokens: float64(b.limit), lastRefill: now}
        b.buckets[entityID] = e
    }

    elapsed := now.Sub(e.lastRefill).Seconds()
    e.tokens = math.Min(float64(b.limit), e.tokens+elapsed*b.refillRate)
    e.lastRefill = now

    if e.tokens < float64(cost) {
        return false, int(e.tokens)
    }

    e.tokens -= float64(cost)
    return true, int(e.tokens)
}
```

The Redis path uses a Lua script for the same reason — `GET` + `SET` is not atomic, `EVAL` is:

```go
var luaScript = redis.NewScript(`
    local key   = KEYS[1]
    local limit = tonumber(ARGV[1])
    local rate  = tonumber(ARGV[2])
    local cost  = tonumber(ARGV[3])
    local now   = tonumber(ARGV[4])

    local data  = redis.call("HMGET", key, "tokens", "last_refill")
    local tokens     = tonumber(data[1]) or limit
    local lastRefill = tonumber(data[2]) or now

    local elapsed = (now - lastRefill) / 1000
    tokens = math.min(limit, tokens + elapsed * rate)

    if tokens < cost then
        redis.call("HMSET", key, "tokens", tokens, "last_refill", now)
        redis.call("EXPIRE", key, 3600)
        return {0, math.floor(tokens)}
    end

    tokens = tokens - cost
    redis.call("HMSET", key, "tokens", tokens, "last_refill", now)
    redis.call("EXPIRE", key, 3600)
    return {1, math.floor(tokens)}
`)
```

The circuit breaker around Redis is just a counter — three consecutive errors flip the mode, a background ping flips it back. Nothing fancy, which is the point. More state = more things to debug at 2am.

Load tested with k6: three concurrent entities, one hammering at 10x the limit. The other two never see a 429.

`Go` `Redis Lua` `MySQL` `Circuit Breaker` `Peer Sync` `k6` `Prometheus`

---

#### 🎬 [cine-mcp](https://github.com/hackzbhavin/cine-mcp)
Full-stack movie ticket booking API with **Model Context Protocol (MCP)** support.

The idea: instead of building a frontend for everything, expose your backend as MCP tools so AI agents (Claude, GPT, custom agents) can query cinemas, search by name/city, check showtimes, and book seats — no REST wrappers, no glue code.

Explores two MCP primitives most people skip:
- **Sampling** — the server asks the AI to generate something mid-tool execution (e.g. generate a booking summary).
- **Elicitation** — the server pauses execution to ask the user a follow-up question from inside a tool.

`NestJS 11` `MySQL 8` `Redis 7` `BullMQ` `MCP SDK` `Prometheus` `Grafana` `k6`

---

#### 🚀 [rush-queue](https://github.com/hackzbhavin/rush-queue)
High-concurrency backend for flash sales and ticket drops — handles thousands of simultaneous buyers without overselling a single unit.

The core problem: at 10,000 concurrent requests, a simple `SELECT + UPDATE` races. You need atomic reservation without locking the entire inventory table.

`NestJS` `Redis` `BullMQ` `MySQL`

***

### Highlighted Projects

- 🧮 **Salary Calculator (Clean Architecture + TDD)** — Python/TypeScript service with 4-layer clean architecture and full test coverage for salary and metrics calculation.
- 📊 **SaaS Analytics Dashboard** — React + Next.js dashboard powered by microservices and GraphQL for real-time business metrics.
- 📱 **React Native App** — production mobile app with real-time communication and optimized MySQL queries.

***

### Tech Stack

**Languages**
JavaScript · TypeScript · Go · Python · PHP

**Frontend**
React.js · Next.js · React Native · Redux

**Backend**
Node.js · NestJS · Express.js · Go · Django · Laravel

**Cloud & DevOps**
AWS (Lambda, EC2, S3, DynamoDB, API Gateway, IAM, CloudFront, SNS, SQS) · Docker · Kubernetes · Terraform · GitHub Actions

**Data & Messaging**
MySQL · DynamoDB · Redis · Kafka · BullMQ

**Observability**
Prometheus · Grafana · cAdvisor · k6

***

### Connect

- 💼 LinkedIn: [bhavin-patil-413a601a0](https://www.linkedin.com/in/bhavin-patil-413a601a0)
- 💻 HackerRank: [hackzbhavin](https://www.hackerrank.com/hackzbhavin)
- 🧠 LeetCode: [hackzbhavin](https://leetcode.com/hackzbhavin)
- 📫 Email: **bhavinpatil28@gmail.com**

***

### Stats

<p align="left">
  <img src="https://github-readme-stats.vercel.app/api/top-langs?username=hackzbhavin&show_icons=true&locale=en&layout=compact" alt="Top languages" />
</p>

<h3>LeetCode</h3>

<p align="left">
  <img src="https://leetcard.jacoblin.cool/hackzbhavin?theme=dark&font=Roboto&ext=heatmap" alt="LeetCode stats" />
</p>
