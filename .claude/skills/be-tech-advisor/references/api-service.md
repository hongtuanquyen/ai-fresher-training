# REST / GraphQL API Service — Tech Stack Reference

## Đặc điểm archetype
- CRUD là core: tạo, đọc, cập nhật, xóa entities
- Business logic ở tầng service, không phải DB stored procedure
- Auth + authorization cho mọi endpoint
- Validation input chặt chẽ
- API documentation (OpenAPI / GraphQL schema)
- Relational data model chiếm đa số
- Deploy thường là container hoặc PaaS
- Client: web app, mobile app, hoặc cả hai

## Stack combinations đã curate

### Combo 1: NestJS + PostgreSQL + Prisma (Node.js — khuyến nghị mặc định)
- **Runtime**: Node.js 22 LTS
- **Framework**: NestJS 11 — opinionated, DI, modular, decorator-based
- **Language**: TypeScript strict
- **Database**: PostgreSQL 17
- **ORM**: Prisma 6 — type-safe, migration, studio GUI
- **Cache**: Redis 7 (ioredis)
- **Auth**: Passport.js + JWT + refresh token
- **Validation**: class-validator + class-transformer (DTO)
- **API doc**: @nestjs/swagger (auto-gen OpenAPI)
- **Queue**: BullMQ (Redis-based)
- **Logging**: Pino (structured JSON)
- **Testing**: Vitest + Supertest + testcontainers
- **Deploy**: Docker → Railway / Render / AWS ECS
- **Khi nào**: team Node/TS, dự án vừa-lớn, cần structure rõ, enterprise-grade
- **Khi nào không**: team không quen decorator pattern, dự án nhỏ (overkill)

### Combo 2: Fastify + PostgreSQL + Drizzle (Node.js — lightweight)
- **Runtime**: Node.js 22 LTS
- **Framework**: Fastify 5 — nhanh nhất Node.js, plugin system, schema-based validation
- **Language**: TypeScript
- **Database**: PostgreSQL 17
- **ORM**: Drizzle — SQL-like syntax, type-safe, lightweight, no codegen
- **Cache**: Redis (ioredis)
- **Auth**: @fastify/jwt + @fastify/cookie
- **Validation**: Zod + fastify-type-provider-zod (auto OpenAPI)
- **API doc**: @fastify/swagger (auto-gen)
- **Queue**: BullMQ hoặc pg-boss (PostgreSQL-based, không cần Redis)
- **Logging**: Pino (built-in Fastify)
- **Testing**: Vitest + Fastify inject()
- **Deploy**: Docker → VPS / Railway / Fly.io
- **Khi nào**: team Node/TS, cần performance cao hơn Express/NestJS, ít boilerplate, dự án nhỏ-vừa
- **Khi nào không**: cần DI container phức tạp, team thích opinionated framework

### Combo 3: Hono + D1/Turso + Drizzle (Edge / Serverless)
- **Runtime**: Cloudflare Workers / Bun / Node.js
- **Framework**: Hono — ultralight, web-standard, multi-runtime
- **Language**: TypeScript
- **Database**: Cloudflare D1 (SQLite edge) hoặc Turso (distributed SQLite)
- **ORM**: Drizzle
- **Cache**: Cloudflare KV / Upstash Redis
- **Auth**: Hono middleware + JWT
- **Validation**: Zod + @hono/zod-openapi
- **API doc**: auto-gen OpenAPI từ Zod schema
- **Queue**: Cloudflare Queues / Upstash QStash
- **Logging**: structured console (Workers) / Axiom
- **Testing**: Vitest + miniflare
- **Deploy**: Cloudflare Workers (wrangler) / Vercel Edge
- **Khi nào**: latency thấp global, pay-per-request, cold start gần 0, budget thấp, API đơn giản-vừa
- **Khi nào không**: cần PostgreSQL feature (JSONB, full-text, complex JOIN), heavy background job, WebSocket persistent

### Combo 4: Django + PostgreSQL (Python)
- **Runtime**: Python 3.12+
- **Framework**: Django 5 + Django REST Framework (hoặc Django Ninja cho type-safe)
- **Language**: Python (type hints)
- **Database**: PostgreSQL 17
- **ORM**: Django ORM (built-in, migration tốt)
- **Cache**: Redis (django-redis)
- **Auth**: Django auth (built-in) + djangorestframework-simplejwt
- **Validation**: serializers (DRF) hoặc Pydantic (Django Ninja)
- **API doc**: drf-spectacular (auto OpenAPI)
- **Queue**: Celery + Redis/RabbitMQ
- **Logging**: structlog
- **Testing**: pytest + factory_boy + testcontainers
- **Deploy**: Docker → AWS / GCP / Railway / VPS (Gunicorn + Uvicorn)
- **Khi nào**: team Python, cần admin panel built-in (Django Admin rất mạnh), rapid prototyping, data-heavy app
- **Khi nào không**: cần high concurrency I/O-bound (Node/Go tốt hơn), team không biết Python

### Combo 5: FastAPI + PostgreSQL + SQLAlchemy (Python — async)
- **Runtime**: Python 3.12+
- **Framework**: FastAPI — async, type-safe, auto OpenAPI, modern Python
- **Language**: Python (type hints + Pydantic)
- **Database**: PostgreSQL 17
- **ORM**: SQLAlchemy 2.0 (async) + Alembic migration
- **Cache**: Redis (aioredis)
- **Auth**: FastAPI Security + JWT (python-jose)
- **Validation**: Pydantic v2 (built-in FastAPI)
- **API doc**: auto-gen OpenAPI + Swagger UI + ReDoc
- **Queue**: Celery hoặc arq (async)
- **Logging**: structlog
- **Testing**: pytest + httpx (async test client)
- **Deploy**: Docker (Uvicorn) → AWS / GCP / Fly.io
- **Khi nào**: team Python, cần async, auto API doc tốt nhất, ML/AI integration dễ (cùng ecosystem), type safety Python
- **Khi nào không**: cần admin panel (Django tốt hơn), team không quen async

### Combo 6: Go + PostgreSQL + Chi/Echo/Fiber (Go)
- **Runtime**: Go 1.23+
- **Framework**: Chi (minimal) / Echo (balanced) / Fiber (Express-like)
- **Language**: Go
- **Database**: PostgreSQL 17
- **ORM/Query**: sqlc (type-safe SQL → Go) hoặc GORM (ORM) hoặc Bun
- **Cache**: Redis (go-redis)
- **Auth**: golang-jwt + middleware
- **Validation**: go-playground/validator
- **API doc**: swaggo (comment-based OpenAPI)
- **Queue**: Asynq (Redis-based) hoặc River (PostgreSQL-based)
- **Logging**: slog (stdlib) hoặc zerolog
- **Testing**: Go testing + testify + testcontainers-go
- **Deploy**: Single binary → Docker / VPS / K8s / Lambda
- **Khi nào**: cần throughput cao (>50k RPM), low memory footprint, team biết Go, microservice, CLI tool cùng repo
- **Khi nào không**: rapid prototyping (Go verbose hơn), team không biết Go, cần ORM migration mạnh

### Combo 7: Spring Boot + PostgreSQL (Java/Kotlin)
- **Runtime**: JVM (Java 21 LTS / Kotlin)
- **Framework**: Spring Boot 3.x
- **Language**: Kotlin (khuyến nghị) hoặc Java
- **Database**: PostgreSQL 17
- **ORM**: Spring Data JPA + Hibernate / jOOQ (type-safe SQL)
- **Cache**: Spring Cache + Redis (Lettuce)
- **Auth**: Spring Security + OAuth2 Resource Server
- **Validation**: Jakarta Validation (built-in)
- **API doc**: springdoc-openapi (auto Swagger)
- **Queue**: Spring AMQP (RabbitMQ) / Spring Kafka
- **Logging**: SLF4J + Logback (structured)
- **Testing**: JUnit 5 + MockK (Kotlin) + Testcontainers
- **Deploy**: Docker (JVM) / GraalVM native image → K8s / AWS ECS
- **Khi nào**: enterprise, team Java/Kotlin, cần transaction phức tạp, integration pattern mạnh (Spring ecosystem), hiring dễ nhất cho enterprise
- **Khi nào không**: startup nhỏ cần đi nhanh, memory-constrained (JVM heavy), team không biết Java

### Combo 8: Laravel + MySQL/PostgreSQL (PHP)
- **Runtime**: PHP 8.3+
- **Framework**: Laravel 12
- **Language**: PHP
- **Database**: MySQL 8 hoặc PostgreSQL 17
- **ORM**: Eloquent (built-in, Active Record)
- **Cache**: Redis (predis/phpredis)
- **Auth**: Laravel Sanctum (SPA/mobile) / Laravel Passport (OAuth2)
- **Validation**: built-in (rất mạnh, declarative)
- **API doc**: Scribe hoặc l5-swagger
- **Queue**: Laravel Queue (Redis/DB/SQS driver)
- **Logging**: Monolog (built-in)
- **Testing**: Pest PHP / PHPUnit + database factory
- **Deploy**: Laravel Forge / Docker / VPS (PHP-FPM + Nginx)
- **Khi nào**: team PHP, rapid development, cần built-in everything (auth, queue, mail, scheduler, broadcasting), hosting rẻ (shared hosting cũng chạy được)
- **Khi nào không**: high concurrency I/O (PHP blocking), team không biết PHP, microservices

## Layer bổ sung thường gặp

| Nhu cầu | Khuyến nghị |
|---|---|
| API Gateway | Kong / Traefik / AWS API Gateway / Nginx |
| Rate limiting | Redis sliding window / Upstash Ratelimit / framework built-in |
| Full-text search | PostgreSQL tsvector (simple) / Typesense / MeiliSearch / Elasticsearch |
| File storage | S3 / Cloudflare R2 / MinIO (self-host) |
| Email | Resend / SendGrid / Amazon SES / Postmark |
| SMS | Twilio / Vonage |
| Payment | Stripe / PayPal / local gateway |
| PDF generation | Puppeteer / Gotenberg / WeasyPrint (Python) |
| Image processing | Sharp (Node) / Pillow (Python) / Imgproxy |
| Monitoring | OpenTelemetry + Grafana + Prometheus / Datadog / New Relic |
| Error tracking | Sentry |
| Secrets management | Vault / AWS Secrets Manager / Infisical / Doppler |
| Feature flag | Unleash / LaunchDarkly / PostHog |
| API versioning | URL prefix (/v1/) hoặc header-based |
| WebSocket | Socket.io (Node) / Django Channels / Gorilla (Go) / Spring WebSocket |
| GraphQL | Apollo Server / Mercurius (Fastify) / Strawberry (Python) / gqlgen (Go) |
| gRPC | @grpc/grpc-js (Node) / grpcio (Python) / google.golang.org/grpc (Go) |

## Anti-patterns

- Dùng NoSQL (MongoDB) mặc định vì "flexible" → relational data cần PostgreSQL, flexible schema dùng JSONB column
- Express cho project mới không có lý do cụ thể → Fastify nhanh hơn, Hono nhẹ hơn, NestJS structured hơn
- Không có migration tool → schema drift giữa env
- Raw SQL ở tầng controller → injection risk, khó maintain
- JWT không có refresh token → UX kém (user bị logout đột ngột)
- Logging console.log → không có structured data, không query được
- Không có health check endpoint → deploy blind
- Monolith nhưng deploy như microservice (mỗi module 1 container) → worst of both worlds
- Microservice cho team <5 người → coordination overhead lớn hơn benefit
- Tự implement auth từ đầu → dùng library/service (Passport, Spring Security, Django Auth)
