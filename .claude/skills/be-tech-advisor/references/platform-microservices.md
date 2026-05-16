# Platform / Microservices — Tech Stack Reference

## Đặc điểm archetype
- Nhiều service giao tiếp với nhau (sync + async)
- Team lớn, mỗi team sở hữu 1-2 service (Conway's Law)
- Domain-driven design (bounded context)
- API gateway / BFF (Backend for Frontend)
- Service discovery, load balancing
- Distributed tracing, centralized logging
- Database per service (hoặc shared với strict boundary)
- Deploy independent, CI/CD per service
- Monitoring phức tạp (nhiều failure point)

## Stack combinations đã curate

### Combo 1: Node.js Microservices + NestJS + NATS + PostgreSQL (khuyến nghị team Node)
- **Framework**: NestJS (mỗi service 1 NestJS app, hoặc NestJS Microservice transport)
- **Communication sync**: REST / gRPC (cho internal service-to-service)
- **Communication async**: NATS JetStream (message broker, nhẹ hơn Kafka)
- **API Gateway**: Kong / custom NestJS gateway
- **Database**: PostgreSQL per service (Prisma)
- **Cache**: Redis (shared hoặc per service)
- **Auth**: centralized auth service + JWT propagation
- **Tracing**: OpenTelemetry → Jaeger / Grafana Tempo
- **Logging**: Pino → Fluentd / Vector → Loki / Elasticsearch
- **Monitoring**: Prometheus + Grafana
- **Deploy**: Docker → Kubernetes (Helm charts)
- **Monorepo**: Turborepo hoặc Nx (shared libs: types, utils, proto)
- **Khi nào**: team >10 Node.js dev, domain phức tạp (e-commerce platform, SaaS multi-module), cần deploy independent
- **Khi nào không**: team <5 (monolith tốt hơn), product chưa tìm được market fit

### Combo 2: Go Microservices + gRPC + Kafka + PostgreSQL (khuyến nghị high-performance)
- **Language**: Go
- **Framework**: mỗi service dùng Chi/Echo (HTTP) + gRPC server (internal)
- **Communication sync**: gRPC (Protobuf, code-gen, fast)
- **Communication async**: Kafka (event log, replay-able)
- **API Gateway**: Envoy / Kong / Traefik
- **Database**: PostgreSQL per service (sqlc hoặc Bun)
- **Cache**: Redis
- **Auth**: centralized auth + JWT / mTLS (service-to-service)
- **Tracing**: OpenTelemetry → Jaeger
- **Logging**: zerolog / slog → Vector → Loki
- **Monitoring**: Prometheus + Grafana
- **Deploy**: Docker → Kubernetes
- **Monorepo**: Go workspace (go.work) + Buf (Protobuf management)
- **Khi nào**: throughput cực cao, team Go, cần low latency giữa service, resource efficient
- **Khi nào không**: rapid prototyping, team không biết Go + Protobuf

### Combo 3: Java/Kotlin + Spring Cloud + Kafka + PostgreSQL (Enterprise)
- **Language**: Kotlin (hoặc Java 21)
- **Framework**: Spring Boot 3 + Spring Cloud
- **Communication sync**: Spring Cloud OpenFeign / REST / gRPC
- **Communication async**: Kafka (Spring Kafka)
- **Service discovery**: Eureka / Consul / Kubernetes DNS
- **API Gateway**: Spring Cloud Gateway / Kong
- **Database**: PostgreSQL per service (Spring Data JPA)
- **Cache**: Spring Cache + Redis (Lettuce)
- **Auth**: Spring Security + OAuth2 / Keycloak
- **Tracing**: Micrometer + OpenTelemetry → Zipkin / Jaeger
- **Logging**: SLF4J + Logback → ELK
- **Config**: Spring Cloud Config / Consul KV
- **Deploy**: Docker → Kubernetes / AWS ECS
- **Khi nào**: enterprise lớn, team Java/Kotlin, cần mature ecosystem (Spring Cloud đã battle-tested), compliance strict
- **Khi nào không**: startup cần đi nhanh, team nhỏ, không có K8s experience

### Combo 4: Modular Monolith → Microservices (phương pháp tiến hóa)
- **Approach**: bắt đầu monolith nhưng tổ chức code theo domain module rõ ràng, tách service khi cần
- **Framework**: NestJS (Module system) / Django (App system) / Spring Boot (multi-module Maven/Gradle)
- **Communication internal**: function call (cùng process), event bus in-memory
- **Database**: shared PostgreSQL nhưng schema per module (strict boundary)
- **Khi nào tách**: module A scale khác module B, team ownership rõ ràng, deploy cycle khác nhau
- **Khi nào**: chưa biết domain boundary rõ, team <10, product đang explore
- **Khi nào không**: domain boundary đã rõ ràng, team >15, cần deploy independent ngay

### Combo 5: Serverless Microservices (AWS Lambda / GCP Cloud Run)
- **Compute**: AWS Lambda (Node/Python/Go) hoặc GCP Cloud Run (container)
- **API Gateway**: AWS API Gateway / GCP API Gateway
- **Communication async**: AWS SQS/SNS / GCP Pub/Sub / EventBridge
- **Database**: DynamoDB (serverless) / Aurora Serverless (SQL) / PlanetScale
- **Auth**: AWS Cognito / Firebase Auth
- **Tracing**: AWS X-Ray / GCP Cloud Trace
- **IaC**: SST (Serverless Stack) / Terraform / Pulumi / SAM
- **Deploy**: serverless deploy (sst deploy / serverless deploy)
- **Khi nào**: traffic spike-heavy (pay-per-use), team nhỏ không muốn quản lý server, AWS/GCP đã là platform chính
- **Khi nào không**: cần persistent connection (WebSocket khó trên Lambda), latency-sensitive (cold start), predictable high traffic (container rẻ hơn)

## Layer bổ sung thường gặp

| Nhu cầu | Khuyến nghị |
|---|---|
| API Gateway | Kong / Traefik / Envoy / AWS API Gateway / NGINX |
| Service mesh | Istio / Linkerd (nếu >10 services, cần mTLS, traffic control) |
| Service discovery | Kubernetes DNS (nếu K8s) / Consul / Eureka |
| Config management | Consul KV / Spring Cloud Config / AWS Parameter Store / Infisical |
| Secret management | Vault / AWS Secrets Manager / Infisical / Doppler |
| Circuit breaker | Resilience4j (Java) / opossum (Node) / custom (Go) |
| Distributed transaction | Saga pattern (choreography hoặc orchestration) — KHÔNG dùng 2PC |
| IaC | Terraform / Pulumi / CDK / SST |
| Container orchestration | Kubernetes (EKS/GKE/AKS) / Docker Swarm (nhỏ) / Nomad |
| CI/CD | GitHub Actions / GitLab CI / ArgoCD (GitOps) |
| Feature flag | Unleash / LaunchDarkly / PostHog |
| Contract testing | Pact (consumer-driven contract test) |
| Schema management | Buf (Protobuf) / AsyncAPI (event schema) |

## Anti-patterns

- Microservices từ ngày 1 cho startup → monolith first, tách khi cần
- Distributed monolith (services phụ thuộc nhau chặt, deploy phải cùng lúc) → cần bounded context rõ
- Shared database giữa services → mỗi service sở hữu data riêng
- Synchronous chain (A → B → C → D) → 1 service chậm = cả chain chậm, dùng async khi có thể
- 2-Phase commit (distributed transaction) → dùng Saga pattern
- Không có distributed tracing → debug production incident gần như impossible
- Không có circuit breaker → cascade failure
- Quá nhiều service cho team nhỏ → mỗi dev maintain 5 service = không ai hiểu rõ service nào
- Dùng Kubernetes cho 2-3 container → Docker Compose hoặc PaaS (Railway, Render) đủ
- Không có contract test → breaking change phát hiện ở production
- Event schema không có versioning → consumer break khi producer đổi schema
