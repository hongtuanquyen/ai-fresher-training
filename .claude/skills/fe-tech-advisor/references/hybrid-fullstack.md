# Hybrid / Full-stack — Tech Stack Reference

## Đặc điểm archetype
- Vừa có phần public (SEO, SSR/SSG) vừa có phần app (behind login, SPA-like)
- Hoặc: full-stack framework xử lý cả FE + BE trong 1 codebase
- Database + API + frontend cùng deploy
- Auth phức tạp (role-based, multi-tenant)
- Background job, cron, webhook
- File upload, email, notification
- Monorepo hoặc single-repo full-stack phổ biến

## Stack combinations đã curate

### Combo 1: Next.js Full-stack + Prisma + Postgres (khuyến nghị mặc định)
- **Framework**: Next.js 15 App Router (Server Actions + API Routes)
- **Language**: TypeScript
- **Database**: PostgreSQL (Neon / Supabase / PlanetScale MySQL)
- **ORM**: Prisma hoặc Drizzle
- **Auth**: NextAuth.js v5 / Clerk
- **Styling**: Tailwind CSS
- **UI**: shadcn/ui
- **State**: Zustand (client) + React Server Components (server)
- **Server state**: TanStack Query (cho client components)
- **Email**: React Email + Resend
- **File upload**: UploadThing / Vercel Blob
- **Background job**: Inngest / Trigger.dev
- **Testing**: Vitest + Playwright
- **Deploy**: Vercel
- **Khi nào**: team React, cần full-stack trong 1 framework, Vercel deploy, startup/side project, prototype nhanh
- **Khi nào không**: team muốn tách rõ FE-BE, backend phức tạp cần framework riêng

### Combo 2: Nuxt 3 Full-stack + Drizzle + SQLite/Postgres
- **Framework**: Nuxt 3 (Nitro server)
- **Language**: TypeScript
- **Database**: SQLite (D1) hoặc PostgreSQL
- **ORM**: Drizzle
- **Auth**: Nuxt Auth Utils / Lucia
- **Styling**: UnoCSS / Tailwind
- **UI**: Nuxt UI
- **File upload**: custom (S3 presigned URL)
- **Deploy**: Cloudflare (D1 + Workers) hoặc Vercel
- **Khi nào**: team Vue, muốn edge deployment (Cloudflare), DX tốt với auto-imports
- **Khi nào không**: team React, cần ecosystem library lớn

### Combo 3: SvelteKit Full-stack + Drizzle + Postgres
- **Framework**: SvelteKit
- **Language**: TypeScript
- **Database**: PostgreSQL
- **ORM**: Drizzle
- **Auth**: Lucia Auth
- **Styling**: Tailwind CSS
- **UI**: Skeleton UI / Melt UI
- **Form**: Superforms + Zod
- **Deploy**: Vercel / Cloudflare / Node
- **Khi nào**: team nhỏ, muốn DX tốt nhất, Svelte reactivity, less boilerplate
- **Khi nào không**: team lớn cần hiring dễ, cần nhiều 3rd party React library

### Combo 4: Remix + Prisma + Postgres
- **Framework**: Remix (React Router v7 framework mode)
- **Language**: TypeScript
- **Database**: PostgreSQL
- **ORM**: Prisma
- **Auth**: Remix Auth / custom session
- **Styling**: Tailwind CSS
- **UI**: shadcn/ui (dùng được với Remix)
- **Form**: Remix built-in (progressive enhancement)
- **Deploy**: Fly.io / Vercel / Railway
- **Khi nào**: cần progressive enhancement (form hoạt động không cần JS), nested routing mạnh, data loading pattern rõ ràng
- **Khi nào không**: cần SSG (Remix chủ yếu SSR), team muốn React Server Components

### Combo 5: T3 Stack (Next.js + tRPC + Prisma + Tailwind)
- **Framework**: Next.js
- **API layer**: tRPC (end-to-end type-safe, không cần viết API schema)
- **Database**: PostgreSQL
- **ORM**: Prisma
- **Auth**: NextAuth.js
- **Styling**: Tailwind CSS
- **UI**: shadcn/ui
- **Validation**: Zod (shared FE-BE)
- **Deploy**: Vercel
- **Khi nào**: TypeScript purist, muốn end-to-end type safety, API layer auto-generated, solo dev hoặc team nhỏ
- **Khi nào không**: API cần expose cho mobile/3rd party (tRPC là RPC, không phải REST), team lớn muốn API contract rõ ràng

### Combo 6: Monorepo (Turborepo) + Next.js + Hono/Express
- **Monorepo**: Turborepo
- **Frontend**: Next.js / Nuxt / SvelteKit
- **Backend**: Hono / Express / Fastify (tách riêng)
- **Shared**: TypeScript types, Zod schemas, utils
- **Database**: Prisma / Drizzle (trong BE package)
- **Deploy**: Vercel (FE) + Railway/Fly.io (BE)
- **Khi nào**: cần tách rõ FE-BE nhưng share types, team lớn mỗi phần có owner riêng, cần API cho cả web + mobile
- **Khi nào không**: solo dev, prototype, không cần API riêng

## Layer bổ sung thường gặp

| Nhu cầu | Khuyến nghị |
|---|---|
| Realtime | Pusher / Ably / Supabase Realtime / Socket.io |
| Queue/background job | Inngest / Trigger.dev / BullMQ |
| Cron | Vercel Cron / Inngest scheduled |
| File storage | Vercel Blob / S3 / Cloudflare R2 |
| Search (full-text) | PostgreSQL full-text / Typesense / MeiliSearch |
| Rate limiting | Upstash Redis / in-memory |
| Multi-tenant | Row-level security (Postgres) / schema-per-tenant |
| Webhook | Svix / custom with queue |
| Monitoring | Sentry + Vercel Analytics |
| CI/CD | GitHub Actions |
| Logging | Axiom / Betterstack |

## Anti-patterns

- Dùng full-stack framework nhưng vẫn tạo REST API riêng bên ngoài → mất lợi ích co-location
- Server Actions cho mọi thứ (kể cả data fetching) → Server Actions là cho mutations, dùng RSC/loader cho reads
- Không type-safe giữa FE-BE → dùng tRPC hoặc shared Zod schema
- SQLite cho production multi-instance → chỉ dùng khi single-instance hoặc edge (D1)
- Không setup seed data → mỗi dev mất thời gian tạo test data
- Prisma Client generate ở runtime → generate ở build time, commit `prisma/migrations`
