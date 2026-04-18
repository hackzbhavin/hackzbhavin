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

> Two repos. Same problem. Two different engineering levels. Real load test results.

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

> The takeaway: the gap between these two repos is not about more code — it is about understanding **which parts of a system need to be fast and which parts just need to eventually be correct**.

***

### Highlighted Projects

- 🚀 **(In Progress) Flash Sale / Ticketing Backend** — high-concurrency backend using Go/NestJS, Redis, and Postgres to handle thousands of users buying limited tickets without overselling.
- 🧮 **Salary Calculator (Clean Architecture + TDD)** — Python/TypeScript service with 4-layer clean architecture and full test coverage for salary and metrics calculation.
- 📊 **SaaS Analytics Dashboard** — React + Next.js dashboard powered by microservices and GraphQL for real-time business metrics.
- 📱 **React Native App** — production mobile app with real-time communication and optimized MySQL queries for better performance.

***

### Tech Stack

**Languages**
JavaScript · TypeScript · Go · Python · PHP

**Frontend**
React.js · Next.js · React Native · Redux

**Backend**
Node.js · NestJS · Express.js · Django · Laravel

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
