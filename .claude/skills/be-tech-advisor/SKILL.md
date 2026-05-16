---
name: be-tech-advisor
description: >
  Đọc yêu cầu dự án backend (từ chat hoặc file spec), phân tích nhu cầu, rồi đề xuất 2-3 option tech stack hoàn chỉnh để so sánh. Mỗi option gồm: danh sách công nghệ + lý do chọn + bảng pros/cons + folder structure + config files mẫu. Trigger skill này khi user hỏi về lựa chọn công nghệ cho backend mới, cần tư vấn tech stack server-side, muốn so sánh framework/database/infra, hỏi "nên dùng gì cho backend dự án X", hoặc đưa spec/requirement và muốn biết nên dùng stack nào cho phía server. Cũng trigger khi user đưa file PDF/doc/markdown chứa yêu cầu dự án và muốn đề xuất công nghệ backend. KHÔNG trigger cho câu hỏi cụ thể về cách dùng 1 framework đã chọn (ví dụ "cách viết middleware trong Express"), chỉ trigger khi cần TƯ VẤN CHỌN stack. KHÔNG trigger cho câu hỏi frontend — dùng fe-tech-advisor cho frontend.
---

# BE Tech Stack Advisor

Skill phân tích yêu cầu dự án backend và đề xuất 2-3 option tech stack hoàn chỉnh để so sánh.

## Khi nào dùng skill này

- User mô tả dự án backend mới và hỏi nên dùng công nghệ gì
- User upload file spec và muốn tư vấn stack server-side
- User muốn so sánh framework/database/queue/deploy cho dự án cụ thể
- User hỏi dạng: "nên dùng NestJS hay Fastify?", "database nào phù hợp?", "backend stack cho dự án X?"

## Workflow

### Bước 1 — Thu thập yêu cầu

Đọc input từ user (chat hoặc file). Trích xuất các tín hiệu sau:

```
PROJECT SIGNALS:
├── Loại dự án: REST API | GraphQL API | Realtime (WebSocket/SSE) | BaaS | Microservices | Monolith | Serverless
├── Domain: SaaS | E-commerce | FinTech | HealthTech | Social/Community | IoT | AI/ML serving | Internal tool | Marketplace
├── Quy mô dự kiến: nhỏ (<10 endpoint) | vừa (10-50) | lớn (>50 endpoint / multi-service)
├── Traffic estimate: thấp (<1k RPM) | vừa (1k-50k RPM) | cao (>50k RPM) | spike-heavy (event-driven bursts)
├── Team size: solo | small (2-5) | medium (5-15) | large (>15)
├── Timeline: gấp (<1 tháng) | bình thường (1-3 tháng) | dài hạn (>3 tháng)
├── Data model: relational (nhiều JOIN, transaction) | document (flexible schema) | graph | time-series | mixed
├── Auth: simple (JWT) | OAuth2/OIDC | multi-tenant | RBAC/ABAC phức tạp | SSO enterprise
├── Realtime: không | notification/webhook | chat/collaboration | live data feed
├── File handling: không | upload/download | media processing (image/video/audio) | large file (>1GB)
├── Search: không cần | basic (SQL LIKE) | full-text search | faceted/fuzzy search
├── Background job: không | simple cron | queue-based (email, PDF gen) | heavy pipeline (ETL, ML)
├── Third-party integration: ít (<3 API) | vừa (3-10) | nhiều (>10, aggregation)
├── Compliance: không đặc biệt | GDPR | HIPAA | PCI DSS | SOC 2
├── Deploy target: VPS (DigitalOcean, Hetzner) | Cloud managed (AWS/GCP/Azure) | Serverless (Vercel/Lambda) | Container (Docker/K8s) | Edge
├── Budget: bootstrap (free tier, self-host) | startup (vài trăm $/tháng) | enterprise (không giới hạn)
├── Team experience: Node.js | Python | Go | Java/Kotlin | Rust | Ruby | PHP | .NET | không preference
└── Ràng buộc: legacy integration | specific database required | on-premise | air-gapped | latency SLA
```

**Nếu thiếu thông tin quan trọng**: hỏi user TỐI ĐA 3 câu gộp. Không phỏng vấn dài. Thiếu → dùng default hợp lý, ghi rõ assumption.

**Nếu có file spec**: đọc file trước, trích xuất signal, chỉ hỏi những gì file không cover.

### Bước 2 — Phân loại dự án

Dựa trên tín hiệu, xác định **project archetype**. Đọc file reference tương ứng:

| Archetype | File reference | Khi nào |
|---|---|---|
| REST/GraphQL API | `references/api-service.md` | API server tiêu chuẩn cho web/mobile client, CRUD, business logic |
| Realtime & Event-driven | `references/realtime-event.md` | Chat, notification, live dashboard, IoT, event sourcing, CQRS |
| Data-intensive | `references/data-intensive.md` | ETL, analytics, ML serving, search platform, time-series, heavy aggregation |
| Platform / Microservices | `references/platform-microservices.md` | Multi-service, API gateway, service mesh, large team, domain-driven design |

Một dự án có thể match nhiều archetype → đọc nhiều file reference, ưu tiên archetype chính.

→ **Đọc file reference tương ứng** trước khi đề xuất.

### Bước 3 — Đề xuất 2-3 option

Mỗi option là **một bộ tech stack hoàn chỉnh** gồm tất cả layer cần thiết. Output theo template dưới đây.

**Nguyên tắc chọn option**:
- Option phải **khác nhau có ý nghĩa** (khác framework HOẶC khác database HOẶC khác architecture pattern — không phải chỉ đổi ORM)
- Mỗi option phải **giải quyết được yêu cầu**
- Ưu tiên stack có **ecosystem mạnh, maintained tốt, community lớn, hiring dễ**
- Nếu team đã có kinh nghiệm framework cụ thể → option đầu tiên dùng framework đó
- Ghi rõ **khi nào option này tốt hơn option kia**

### Bước 4 — Output theo template

Với **MỖI option**, output đầy đủ:

---

#### TEMPLATE OUTPUT (lặp cho mỗi option)

```markdown
## Option [A/B/C]: <Tên ngắn> (ví dụ: "NestJS + PostgreSQL + Redis + Docker")

### Tổng quan
<1-2 câu mô tả approach và khi nào nên chọn>

### Tech Stack

| Layer | Công nghệ | Version | Lý do chọn |
|---|---|---|---|
| Runtime | Node.js | 22 LTS | Async I/O, ecosystem lớn, team quen |
| Language | TypeScript | 5.x | Type safety, DX, catch bug sớm |
| Framework | NestJS | 11.x | Opinionated, DI, modular, enterprise-ready |
| Database | PostgreSQL | 17 | ACID, JSONB, full-text search, mature |
| ORM | Prisma | 6.x | Type-safe query, migration, introspection |
| Cache | Redis | 7.x | Session, rate limit, queue, pub/sub |
| Queue | BullMQ | 5.x | Job queue trên Redis, retry, priority |
| Auth | Passport.js + JWT | | Strategy pattern, extensible |
| Validation | class-validator + Zod | | DTO validation NestJS + shared schema |
| API doc | Swagger (auto-gen) | | OpenAPI từ decorator |
| Testing | Vitest + Supertest | | Unit + integration, nhanh |
| Logging | Pino | | Structured JSON log, nhanh |
| Monitoring | OpenTelemetry + Sentry | | Traces + error tracking |
| Container | Docker + docker-compose | | Dev parity, deploy consistent |
| CI/CD | GitHub Actions | | Free cho open-source, mature |
| Deploy | Railway / Render / VPS | | Tùy budget |

### Khi nào chọn option này
- <Điều kiện 1>
- <Điều kiện 2>

### Khi nào KHÔNG chọn
- <Điều kiện 1>
- <Điều kiện 2>

### Folder Structure

\```
project-root/
├── src/
│   ├── app.module.ts
│   ├── main.ts
│   ├── common/                 # Shared: guards, pipes, filters, interceptors
│   │   ├── guards/
│   │   ├── pipes/
│   │   ├── filters/
│   │   ├── interceptors/
│   │   └── decorators/
│   ├── config/                 # Configuration module
│   │   └── config.module.ts
│   ├── modules/
│   │   ├── auth/
│   │   │   ├── auth.module.ts
│   │   │   ├── auth.controller.ts
│   │   │   ├── auth.service.ts
│   │   │   ├── strategies/
│   │   │   └── dto/
│   │   ├── users/
│   │   │   ├── users.module.ts
│   │   │   ├── users.controller.ts
│   │   │   ├── users.service.ts
│   │   │   ├── users.repository.ts
│   │   │   ├── entities/
│   │   │   └── dto/
│   │   └── <feature>/
│   ├── database/
│   │   ├── migrations/
│   │   └── seeds/
│   └── jobs/                   # Background jobs
│       └── email.processor.ts
├── prisma/
│   └── schema.prisma
├── test/
│   ├── unit/
│   ├── integration/
│   └── e2e/
├── docker/
│   ├── Dockerfile
│   └── docker-compose.yml
├── .env.example
├── nest-cli.json
├── tsconfig.json
├── vitest.config.ts
└── package.json
\```

### Config Files mẫu

<Tạo file thật — xem quy tắc bên dưới>
```

---

#### QUY TẮC CONFIG FILES

Tạo **file thật** (dùng create_file) cho mỗi option, đặt trong `/mnt/user-data/outputs/option-[a|b|c]/`:

**Bắt buộc tạo** (mọi option):
1. `package.json` — dependencies đầy đủ, scripts (dev, build, start, test, lint, migrate, seed)
2. `tsconfig.json` — strict, path alias, target phù hợp
3. Config framework chính (vd `nest-cli.json`, `drizzle.config.ts`)
4. `.env.example` — tất cả env vars cần thiết (DB_URL, REDIS_URL, JWT_SECRET, v.v.)
5. `Dockerfile` — multi-stage build, production-ready
6. `docker-compose.yml` — dev environment (app + DB + Redis + ...)
7. `.eslintrc.json` hoặc `eslint.config.js`

**Tạo nếu có trong stack**:
8. `prisma/schema.prisma` hoặc `drizzle/schema.ts` (schema mẫu với 2-3 model cơ bản: User, Session, + 1 domain entity)
9. `vitest.config.ts` hoặc `jest.config.ts`
10. `.github/workflows/ci.yml` (CI cơ bản: lint + test + build)

**Không tạo** (quá chi tiết, user tự setup):
- Terraform, Pulumi, CDK
- Kubernetes manifests
- Business logic code

### Bước 5 — Bảng so sánh tổng hợp

Sau khi output tất cả option, tạo **1 bảng so sánh duy nhất**:

```markdown
## So sánh tổng hợp

| Tiêu chí | Option A | Option B | Option C |
|---|---|---|---|
| Learning curve | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ |
| Performance (throughput) | ... | ... | ... |
| Type safety | ... | ... | ... |
| Ecosystem / library | ... | ... | ... |
| Hiring / community | ... | ... | ... |
| Scalability vertical | ... | ... | ... |
| Scalability horizontal | ... | ... | ... |
| Ops complexity | ... | ... | ... |
| Time to MVP | ... | ... | ... |
| Cost (infra + dev) | ... | ... | ... |
| Testing DX | ... | ... | ... |
| Production readiness | ... | ... | ... |

### Đề xuất của tôi
<Dựa trên yêu cầu cụ thể, chọn 1 option và giải thích ngắn tại sao>
```

## Lưu ý quan trọng

1. **Luôn search web** cho version mới nhất trước khi đề xuất. Không dùng version cũ từ training data.
2. **Config file phải chạy được** — không placeholder. User clone về phải `docker compose up` được (ít nhất cho dev).
3. **Không đề xuất stack deprecated** hoặc maintenance mode (ví dụ: Express 4 cho project mới khi Express 5 đã stable, Sequelize khi có Prisma/Drizzle tốt hơn, trừ khi user chỉ định).
4. **Database choice phải justify** — không mặc định PostgreSQL cho mọi thứ. MongoDB phù hợp cho document-heavy, Redis cho cache, ClickHouse cho analytics. Giải thích tại sao chọn DB đó.
5. **Nếu user đã chỉ định framework**: option đầu dùng framework đó, option 2 là alternative có giải thích.
6. **Ghi rõ assumption** nếu thiếu thông tin.
7. **Tiếng Việt**: output bằng tiếng Việt trừ khi user viết tiếng Anh.
8. **Không mix quá nhiều ngôn ngữ trong 1 option**: mỗi option nên nhất quán về runtime/language (Node+TS, Python, Go, v.v.) — mix chỉ khi có lý do rõ (vd: Python cho ML service + Go cho API gateway).
