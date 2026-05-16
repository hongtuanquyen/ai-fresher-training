# Web App — Tech Stack Reference

## Đặc điểm archetype
- Nhiều tương tác: form, table, filter, modal, drag-drop
- State management phức tạp (multi-level, async, optimistic update)
- Auth/authorization bắt buộc
- API integration nặng (REST/GraphQL, real-time)
- Performance: INP, responsiveness quan trọng hơn LCP
- SEO thường không quan trọng (app sau login)
- Cần component library mạnh (table, date picker, select, toast)

## Stack combinations đã curate

### Combo 1: Next.js App Router + Tailwind + Zustand + TanStack Query (khuyến nghị mặc định)
- **Framework**: Next.js 15 App Router
- **Language**: TypeScript (strict)
- **Styling**: Tailwind CSS
- **UI**: shadcn/ui (accessible, customizable)
- **State**: Zustand (simple) hoặc Jotai (atomic)
- **Server state**: TanStack Query (cache, retry, background refetch)
- **Form**: React Hook Form + Zod
- **Table**: TanStack Table
- **Auth**: NextAuth.js / Clerk / Supabase Auth
- **Testing**: Vitest + React Testing Library + Playwright
- **Deploy**: Vercel
- **Khi nào**: team biết React, cần full-stack capability, ecosystem lớn nhất
- **Khi nào không**: app đơn giản không cần SSR, hoặc team muốn tránh React complexity

### Combo 2: Vite + React + Tailwind + Zustand (SPA thuần)
- **Framework**: Vite (không cần SSR/SSG)
- **Routing**: TanStack Router hoặc React Router
- **Language**: TypeScript
- **Styling**: Tailwind CSS
- **UI**: shadcn/ui hoặc Radix UI
- **State**: Zustand
- **Server state**: TanStack Query
- **Form**: React Hook Form + Zod
- **Auth**: Tự implement hoặc Auth0 SDK
- **Testing**: Vitest + RTL + Playwright
- **Deploy**: Vercel / Netlify / S3+CloudFront / bất kỳ static host
- **Khi nào**: SPA thuần (behind login, SEO không cần), deploy tĩnh, CI/CD đơn giản, tách biệt FE-BE rõ ràng
- **Khi nào không**: cần SSR/SEO, cần API routes cùng project

### Combo 3: Nuxt 3 + Pinia + Vuetify/PrimeVue
- **Framework**: Nuxt 3
- **Language**: TypeScript
- **Styling**: UnoCSS hoặc Tailwind
- **UI**: Vuetify 3 (Material) hoặc PrimeVue (enterprise)
- **State**: Pinia
- **Server state**: Nuxt `useFetch` / `useAsyncData`
- **Form**: VeeValidate + Zod
- **Auth**: Nuxt Auth Utils / Sidebase Auth
- **Testing**: Vitest + Vue Test Utils + Playwright
- **Deploy**: Vercel / Netlify / Node server
- **Khi nào**: team dùng Vue, cần component library enterprise-grade có sẵn (table, form, menu đầy đủ)
- **Khi nào không**: team React, hoặc cần ecosystem library đa dạng hơn

### Combo 4: SvelteKit + Tailwind + Skeleton UI
- **Framework**: SvelteKit
- **Language**: TypeScript
- **Styling**: Tailwind CSS
- **UI**: Skeleton UI / Melt UI
- **State**: Svelte stores (built-in)
- **Form**: Superforms + Zod
- **Auth**: Lucia Auth / custom
- **Testing**: Vitest + Playwright
- **Deploy**: Vercel / Cloudflare / Node
- **Khi nào**: team nhỏ, muốn DX tốt nhất, bundle nhỏ, performance cao, sẵn sàng với ecosystem nhỏ hơn
- **Khi nào không**: team lớn cần hiring dễ, cần nhiều 3rd party library

### Combo 5: Angular + Angular Material / PrimeNG
- **Framework**: Angular 18+
- **Language**: TypeScript (built-in)
- **Styling**: Angular Material hoặc PrimeNG
- **State**: NgRx (complex) hoặc Angular Signals (simple)
- **Form**: Angular Reactive Forms (built-in, rất mạnh)
- **Auth**: Angular Auth OIDC Client
- **Testing**: Jest / Vitest + Cypress
- **Deploy**: bất kỳ
- **Khi nào**: enterprise, team Java/.NET background, cần opinionated framework (tránh "chọn library" fatigue), project lớn cần structure rõ ràng
- **Khi nào không**: startup nhỏ cần iterate nhanh, team không quen Angular

## Layer bổ sung thường gặp

| Nhu cầu | Khuyến nghị |
|---|---|
| Realtime | Socket.io / Ably / Pusher / Supabase Realtime |
| Rich text editor | TipTap / Plate / Lexical |
| Drag & drop | dnd-kit (React) / SortableJS (framework-agnostic) |
| Chart/visualization | Recharts (React) / Chart.js / D3 / ECharts |
| Date picker | react-day-picker / date-fns |
| Toast/notification | Sonner (React) / Vue Toastification |
| File upload | UploadThing / Filepond / custom presigned URL |
| PDF generation | React-PDF / jsPDF |
| Error tracking | Sentry |
| Feature flag | LaunchDarkly / Vercel Flags / PostHog / Unleash |
| Email | React Email + Resend |
| Background job | Inngest / Trigger.dev (nếu full-stack) |

## Anti-patterns

- Dùng Redux cho app nhỏ → boilerplate quá nhiều, dùng Zustand/Jotai thay
- Fetch data trong useEffect thay vì dùng TanStack Query → không có cache, retry, loading state management
- Không dùng TypeScript strict → bug runtime nhiều
- Import toàn bộ component library → bundle bloat (dùng tree-shakeable hoặc copy-paste như shadcn)
- Tự build auth từ đầu → security risk, dùng library/service
- Không setup path alias → import hell (`../../../../components`)
