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

#### 🔒 [nestjs-throttling-mastery](https://github.com/hackzbhavin/nestjs-throttling-mastery)
Production-grade, per-entity throttling — built after studying how Shopify and Netflix actually do it at scale.

The hard constraint: no API gateway. Everything enforced at the app layer, per `entity_id`, so one customer being throttled has zero effect on others.

The tricky part was the **2-server same-DC fallback** — when Redis goes down, two in-memory stores diverge and silently double your effective limit. Solved with a 3-mode state machine:

```
REDIS ──(3 failures)──► PEER SYNC ──(5 failures)──► LOCAL / NODE_COUNT
  ▲                          │                              │
  └──────── Redis ping ok ───┴──────────────────────────────┘
```

- **REDIS mode** — Lua atomic token bucket. Single source of truth. No race conditions across servers.
- **PEER mode** — Both servers gossip snapshots every 100ms. Minimum token count wins. Overshoot window: 100ms max.
- **LOCAL mode** — Redis + peer both unreachable. Each node enforces `limit / node_count`. Accepts capacity halving in exchange for zero downtime.
- **MySQL flush** — Active only in LOCAL mode. Persists counters every 5s so limits survive server restarts during extended outages.

```go
// Same idea, different language — a Go sketch of the token bucket
func (b *Bucket) Consume(entityID string, cost int) (allowed bool, remaining int) {
    b.mu.Lock()
    defer b.mu.Unlock()

    now := time.Now()
    entry, ok := b.buckets[entityID]
    if !ok {
        entry = &BucketEntry{Tokens: float64(b.limit), LastRefill: now}
        b.buckets[entityID] = entry
    }

    // Refill based on elapsed time
    elapsed := now.Sub(entry.LastRefill).Seconds()
    entry.Tokens = math.Min(float64(b.limit), entry.Tokens + elapsed*b.refillRate)
    entry.LastRefill = now

    if entry.Tokens < float64(cost) {
        return false, int(entry.Tokens)
    }

    entry.Tokens -= float64(cost)
    return true, int(entry.Tokens)
}
```

Load tested with k6. Three concurrent entities — one behaving badly, two behaving normally. The two normal ones never see a 429.

`NestJS` `Redis Lua` `Bull` `MySQL` `Circuit Breaker` `Peer Sync` `k6`

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
