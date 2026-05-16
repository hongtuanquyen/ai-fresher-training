# Realtime & Event-driven — Tech Stack Reference

## Đặc điểm archetype
- WebSocket / SSE / long-polling là transport chính
- Event sourcing, CQRS có thể áp dụng
- Pub/sub pattern phổ biến
- State synchronization giữa nhiều client
- Connection management (heartbeat, reconnect, backpressure)
- Horizontal scaling cần sticky session hoặc shared state
- Use case: chat, collaboration (Figma-like), live dashboard, notification, IoT telemetry, gaming

## Stack combinations đã curate

### Combo 1: Node.js + Socket.io + Redis Pub/Sub + PostgreSQL
- **Runtime**: Node.js 22 LTS
- **Framework**: Fastify hoặc NestJS + @nestjs/websockets
- **Realtime**: Socket.io (auto fallback, room, namespace, acknowledgement)
- **Pub/Sub**: Redis Pub/Sub (cross-instance broadcast)
- **Database**: PostgreSQL (persistent data) + Redis (session, presence)
- **Queue**: BullMQ (async task) hoặc Redis Streams (event log)
- **Auth**: JWT cho HTTP, token-based handshake cho WebSocket
- **Deploy**: Docker → multiple instance behind load balancer (sticky session hoặc Redis adapter)
- **Khi nào**: chat, notification, live update, team Node.js, cần auto-reconnect + fallback
- **Khi nào không**: >100k concurrent connections (cần Go/Elixir), cần event sourcing phức tạp

### Combo 2: Elixir + Phoenix LiveView + PostgreSQL
- **Runtime**: BEAM VM (Erlang VM)
- **Framework**: Phoenix 1.7+
- **Realtime**: Phoenix Channels (built-in WebSocket) + Phoenix PubSub + Presence
- **Database**: PostgreSQL (Ecto ORM)
- **Queue**: Oban (PostgreSQL-based, không cần Redis)
- **Auth**: phx_gen_auth (built-in)
- **Deploy**: Fly.io (clustering dễ) / Docker
- **Khi nào**: cần handle triệu connection đồng thời (BEAM VM tối ưu cho concurrency), realtime collaboration, fault-tolerant, team sẵn sàng học Elixir
- **Khi nào không**: team không muốn học ngôn ngữ mới, cần ecosystem library lớn

### Combo 3: Go + Gorilla/nhooyr WebSocket + NATS + PostgreSQL
- **Runtime**: Go 1.23+
- **WebSocket**: nhooyr.io/websocket hoặc gorilla/websocket
- **Message broker**: NATS (ultra-low latency pub/sub) hoặc NATS JetStream (persistent)
- **Database**: PostgreSQL
- **Queue**: NATS JetStream hoặc River (PG-based)
- **Auth**: golang-jwt
- **Deploy**: Single binary → K8s / Docker / VPS
- **Khi nào**: cần throughput cực cao, low memory, IoT (nhiều device kết nối), team Go
- **Khi nào không**: rapid prototyping, team không biết Go

### Combo 4: NestJS + GraphQL Subscriptions + Redis + PostgreSQL
- **Framework**: NestJS + @nestjs/graphql (Apollo)
- **Realtime**: GraphQL Subscriptions (WebSocket transport)
- **Pub/Sub**: Redis Pub/Sub (graphql-redis-subscriptions)
- **Database**: PostgreSQL (Prisma)
- **Khi nào**: API đã dùng GraphQL, cần realtime query result, team TypeScript
- **Khi nào không**: chỉ cần push notification đơn giản (overkill), REST API

### Combo 5: Supabase Realtime (managed)
- **Platform**: Supabase (PostgreSQL + Realtime + Auth + Storage)
- **Realtime**: Supabase Realtime (PostgreSQL logical replication → WebSocket)
- **Database**: PostgreSQL (managed)
- **Auth**: Supabase Auth (built-in)
- **Queue**: Supabase Edge Functions + pg_cron
- **Khi nào**: prototype nhanh, team nhỏ, không muốn quản lý infra, realtime theo row-level change
- **Khi nào không**: cần custom WebSocket logic, high-frequency event (game tick), self-host bắt buộc

## Layer bổ sung thường gặp

| Nhu cầu | Khuyến nghị |
|---|---|
| Message broker | Redis Pub/Sub (simple) / NATS (fast) / RabbitMQ (routing) / Kafka (event log) |
| Event store | EventStoreDB / PostgreSQL (append-only table) / Kafka |
| Presence tracking | Redis SET + TTL / Phoenix Presence / custom heartbeat |
| Connection balancing | Sticky session (Nginx ip_hash) / Redis adapter (Socket.io) / BEAM clustering |
| Rate limiting WS | Token bucket per connection / Redis sliding window |
| Offline sync | CRDTs (Yjs, Automerge) / Operational Transform / last-write-wins |
| Push notification | Firebase Cloud Messaging / Apple APNs / OneSignal / Novu |
| Webhook outgoing | Svix / custom with retry queue |
| SSE (Server-Sent Events) | Native HTTP/2 stream / framework built-in — khi chỉ cần server→client |

## Anti-patterns

- WebSocket cho notification đơn giản → SSE hoặc polling đủ rồi, đỡ phức tạp
- Không có reconnect strategy → client mất kết nối im lặng
- Broadcast tất cả event cho tất cả client → dùng room/channel/topic filter
- WebSocket state ở memory server → mất khi restart/deploy, dùng Redis
- Không có backpressure → client chậm làm server buffer overflow
- Dùng Kafka cho <10k msg/s → overkill, Redis Pub/Sub hoặc NATS đủ
- Event sourcing cho CRUD app đơn giản → complexity không justify
