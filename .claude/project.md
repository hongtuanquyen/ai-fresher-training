# Tổng quan dự án

Tài liệu tóm tắt tổng quan dự án frontend web, được nhóm theo 6 phần chính.

## 1. Overview (Bối cảnh & nghiệp vụ)

- **Mô tả nghiệp vụ**: mục tiêu sản phẩm, đối tượng người dùng, phạm vi.
- **Trình duyệt hỗ trợ**: danh sách browser + version tối thiểu (Chrome, Safari, Firefox, Edge), responsive / mobile web / desktop only.
- **Glossary**: xem [.claude/glossary.md](./glossary.md) để dùng thuật ngữ nhất quán.

## 2. Architecture (Kiến trúc & Tech stack)

- **Tech stack**: framework (Next.js / React / Vue / Nuxt / SvelteKit / Remix), ngôn ngữ (TS / JS), version.
- **Rendering strategy**: CSR / SSR / SSG / ISR / RSC — chọn ở đâu, lý do.
- **Architecture pattern**: Feature-based / Atomic Design / Clean Architecture / Module Federation...
- **State management**: Redux Toolkit / Zustand / Jotai / Pinia / TanStack Query / Context...
- **Routing**: file-based (Next/Nuxt) / React Router / TanStack Router — quy ước route, layout, guard.
- **Styling**: Tailwind / CSS Modules / styled-components / SCSS / UnoCSS — design tokens, theme.
- **Folder Structure**: cấu trúc thư mục chính và quy ước đặt tên.
- **Path alias**: cấu hình alias (tsconfig paths / vite resolve / webpack alias).
- **Build tool**: Vite / Webpack / Turbopack / Rspack — config chính, plugin quan trọng.
- **Package manager**: npm / pnpm / yarn / bun — version, workspace (monorepo?).

## 3. Build & Release (Build & môi trường)

- **Environments**: dev / staging / production — URL, biến cấu hình.
- **Build commands**: lệnh build / dev / preview cho từng env.
- **Env config**: `.env`, `.env.local`, `.env.production` — quy ước prefix (`NEXT_PUBLIC_`, `VITE_`), cách inject biến runtime vs build-time.
- **Feature flags**: cơ chế bật/tắt feature (LaunchDarkly / Unleash / config file).
- **Deployment**: hosting (Vercel / Netlify / Cloudflare Pages / S3+CDN / self-host), CI/CD pipeline.
- **Versioning & Cache**: hashed assets, cache-control, strategy invalidate CDN.

## 4. Integration (Tích hợp ngoài)

- **API integration**:
  - Loại: REST / GraphQL / WebSocket / SSE / tRPC
  - Base URL theo env, BFF / proxy layer
  - Auth scheme (Bearer, OAuth2, Cookie/Session, ...)
  - Data fetching: TanStack Query / SWR / RTK Query / Apollo
  - Error handling convention (boundary, toast, retry)
- **Authentication & Authorization (AuthN / AuthZ)**: luồng login, lưu token (httpOnly cookie / localStorage), refresh, CSRF, phân quyền theo route/component.
- **Third-party services / SDKs**: analytics (GA4, Mixpanel, Amplitude), maps, payment (Stripe), social login, CMS (Contentful / Sanity), search (Algolia)...
- **Web Push / Notification**: Service Worker push, FCM Web — config, permission flow.
- **SEO & Social**: meta tags, OpenGraph, sitemap, robots.txt, canonical, structured data.

## 5. Data & Reliability (Dữ liệu & độ tin cậy)

- **Client storage**:
  - `localStorage` / `sessionStorage` / `IndexedDB` / Cookie — dữ liệu nào, có nhạy cảm không.
  - Secure cookie flags (HttpOnly, Secure, SameSite).
- **Caching**: HTTP cache, service worker cache, query cache (stale-time, GC time).
- **Offline / PWA**: có hỗ trợ không, manifest, service worker strategy (network-first / cache-first).
- **Performance**: Core Web Vitals (LCP / INP / CLS) target, code-splitting, lazy load, image optimization, bundle budget.
- **Logging & Monitoring**: console policy theo env, remote logging (Sentry / Datadog RUM / LogRocket), error boundary.
- **Security**: CSP, CORS, XSS / CSRF prevention, sanitize HTML, dependency audit.

## 6. Quality (Chất lượng)

- **Testing strategy**:
  - Unit (Vitest / Jest)
  - Component (Testing Library / Storybook interaction)
  - E2E (Playwright / Cypress)
  - Visual regression (Chromatic / Percy)
  - Coverage target.
- **Linting & Formatting**: ESLint, Prettier, Stylelint, commit hooks (husky, lint-staged).
- **Type checking**: `tsc --noEmit` trong CI.
- **Accessibility (a11y)**: WCAG level target, axe / Lighthouse a11y audit.
- **i18n**: thư viện (next-intl / react-i18next / vue-i18n), nguồn dữ liệu locale.
