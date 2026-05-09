---
description: Tạo file .claude/project.md tổng quan dự án Next.js theo template chuẩn 6 nhóm (Overview, Architecture, Build & Release, Integration, Data & Reliability, Quality)
---

# Create Next.js Project Overview

Tạo file `.claude/project.md` cho dự án **Next.js** hiện tại, đóng vai trò tài liệu tổng quan dự án.

## Mục tiêu

Sinh ra file `.claude/project.md` có cấu trúc 6 nhóm chuẩn, được điền sẵn dựa trên việc **khảo sát thực tế** codebase. Những thông tin không tự suy ra được thì để placeholder `<TODO: ...>` để người dùng bổ sung sau.

## Quy trình

### Bước 1 — Khảo sát codebase

Đọc và phân tích các file sau (nếu tồn tại) để trích thông tin:

- `package.json` → tên dự án, Next.js version, React version, dependencies (state mgmt, data fetching, styling, auth, analytics, testing...), scripts (`dev`, `build`, `start`, `lint`, `test`).
- `next.config.js` / `next.config.mjs` / `next.config.ts` → output mode (standalone / export), images, i18n, rewrites/redirects, headers, experimental flags, transpilePackages.
- `tsconfig.json` → ngôn ngữ, path alias (`paths`), strict mode.
- `app/` vs `pages/` → App Router (RSC) hay Pages Router; layout, route group, middleware.
- `middleware.ts` / `middleware.js` → auth guard, locale, A/B routing.
- `tailwind.config.*`, `postcss.config.*`, `*.module.css`, `styled-components`/`emotion` setup → styling stack & design tokens.
- `.env`, `.env.local`, `.env.development`, `.env.production` → env handling, prefix `NEXT_PUBLIC_`.
- `public/` → static assets, `manifest.json` (PWA?), `robots.txt`, `sitemap.xml`.
- `eslint.config.*` / `.eslintrc*`, `.prettierrc*`, `vitest.config.*` / `jest.config.*`, `playwright.config.*` / `cypress.config.*` → testing & quality.
- `package-lock.json` / `pnpm-lock.yaml` / `yarn.lock` / `bun.lockb` → package manager.
- `Dockerfile`, `vercel.json`, `netlify.toml`, `.github/workflows/*` → deployment & CI/CD.
- `README.md` → bối cảnh nghiệp vụ.

Khi quét, hãy chạy song song các lệnh đọc/grep cần thiết.

### Bước 2 — Suy luận

Từ dependencies và config, xác định:

- **Router**: App Router (`app/`) hay Pages Router (`pages/`); RSC, Server Actions, Route Handlers.
- **Rendering strategy**: SSG / SSR / ISR / CSR / RSC — dựa vào việc dùng `generateStaticParams`, `revalidate`, `dynamic`, `'use client'`.
- **State management**: redux/@reduxjs/toolkit, zustand, jotai, recoil, mobx, valtio.
- **Data fetching / cache**: @tanstack/react-query, swr, apollo, urql, trpc, graphql-request, ky, axios, native `fetch` với `cache`/`next.revalidate`.
- **Styling**: tailwindcss, styled-components, @emotion/*, sass, css modules, vanilla-extract, unocss; UI kit: shadcn/ui, mui, chakra, antd, radix-ui.
- **Forms & validation**: react-hook-form, formik, zod, yup, valibot.
- **Auth**: next-auth / @auth/*, clerk, supabase-auth, firebase-auth, lucia, custom JWT/cookie.
- **Analytics & monitoring**: @vercel/analytics, @sentry/nextjs, posthog, mixpanel, amplitude, datadog-rum, logrocket.
- **CMS / headless**: contentful, sanity, strapi, prismic, payload.
- **Testing**: vitest, jest, @testing-library/react, playwright, cypress, storybook, chromatic.
- **Build tool**: Turbopack (nếu `next dev --turbo`) hay Webpack mặc định.
- **Package manager**: từ lockfile.
- **i18n**: next-intl, next-i18next, built-in `i18n` config.

### Bước 3 — Sinh file

Ghi file `.claude/project.md` (tạo thư mục `.claude/` nếu chưa có). KHÔNG ghi đè nếu file đã tồn tại — hỏi người dùng trước.

Dùng template dưới đây, thay placeholder bằng giá trị tìm được. Giữ nguyên những mục không xác định được dưới dạng `<TODO: ...>`.

### Bước 4 — Báo cáo

In ra một bản tóm tắt ngắn:

- Đường dẫn file vừa tạo.
- Các trường đã tự điền được.
- Các trường còn `<TODO: ...>` cần người dùng bổ sung.

## Template `.claude/project.md`

````markdown
# Tổng quan dự án

> Dự án Next.js — tài liệu tổng quan theo 6 nhóm.

## 1. Overview (Bối cảnh & nghiệp vụ)

- **Tên dự án**: <TODO>
- **Mô tả nghiệp vụ**: <TODO: mục tiêu sản phẩm, người dùng mục tiêu, phạm vi>
- **Trình duyệt hỗ trợ**: <TODO: ví dụ Chrome/Edge/Firefox/Safari last 2 versions, mobile web>
- **Glossary**: xem [glossary.md](./glossary.md)

## 2. Architecture (Kiến trúc & Tech stack)

- **Framework**: Next.js <version> (App Router / Pages Router)
- **Ngôn ngữ**: TypeScript <version> / JavaScript
- **React**: <version>
- **Rendering strategy**: <SSG / SSR / ISR / CSR / RSC — mô tả ngắn>
- **Architecture pattern**: <TODO: Feature-based / Atomic Design / Clean Architecture ...>
- **State management**: <từ dependencies>
- **Data fetching / cache**: <react-query / swr / fetch + revalidate ...>
- **Styling**: <Tailwind / CSS Modules / styled-components ...>
- **UI kit**: <shadcn/ui / MUI / Chakra ... — nếu có>
- **Forms & validation**: <react-hook-form + zod ...>
- **Folder Structure**:
  ```
  <cây thư mục app/ hoặc src/ rút gọn>
  ```
- **Path alias**: <liệt kê từ tsconfig `paths`>
- **Build tool**: <Turbopack / Webpack>
- **Package manager**: <npm / pnpm / yarn / bun>
- **Monorepo**: <có/không — Turborepo / Nx / pnpm workspace>

## 3. Build & Release (Build & môi trường)

- **Scripts**: <các script trong package.json: dev / build / start / lint / test>
- **Output mode**: <default / standalone / export>
- **Environments**: <dev / staging / production — URL>
- **Env config**:
  - File: `.env`, `.env.local`, `.env.production` ...
  - Prefix biến public: `NEXT_PUBLIC_*`
  - Cơ chế runtime vs build-time: <TODO>
- **Feature flags**: <TODO: cơ chế bật/tắt feature, nếu có>
- **Deployment**: <Vercel / Netlify / Cloudflare Pages / Docker self-host ...>
- **CI/CD**: <GitHub Actions / GitLab CI / ... — workflow chính>
- **Cache & assets**: <CDN, hashed assets, revalidate strategy>

## 4. Integration (Tích hợp ngoài)

- **API integration**:
  - Loại: REST / GraphQL / WebSocket / SSE / tRPC
  - Base URL theo env: <dev / staging / prod>
  - BFF / Route Handlers / Server Actions: <có dùng không>
  - Auth scheme: <Bearer / OAuth2 / Cookie session ...>
  - Error handling convention: <TODO>
- **Authentication & Authorization**:
  - Provider: <NextAuth / Clerk / Supabase / custom>
  - Token storage: <httpOnly cookie / localStorage>
  - Refresh token: <cơ chế>
  - AuthZ: <middleware / route guard / RBAC>
- **Third-party services / SDKs**: <analytics, maps, payment (Stripe), social login, CMS, search ...>
- **SEO & Social**:
  - Metadata API / `next-seo`
  - OpenGraph, Twitter card
  - `sitemap.xml`, `robots.txt`, canonical, structured data
- **i18n**: <next-intl / next-i18next / built-in — locale list>
- **Web Push / Notification**: <Service Worker push / FCM Web — nếu có>

## 5. Data & Reliability (Dữ liệu & độ tin cậy)

- **Client storage**:
  - `localStorage` / `sessionStorage`: <dữ liệu nào>
  - `IndexedDB`: <nếu có>
  - Cookie: <httpOnly, Secure, SameSite settings>
- **Caching**:
  - HTTP cache headers
  - `fetch` cache / `revalidate` / `revalidateTag` / `revalidatePath`
  - Query cache (stale time, gc time)
- **Offline / PWA**: <có/không — manifest, service worker strategy>
- **Performance**:
  - Core Web Vitals target (LCP / INP / CLS)
  - Code-splitting, dynamic import, `next/image`, `next/font`
  - Bundle budget
- **Logging & Monitoring**:
  - Log levels: <TODO>
  - Remote logging: <Sentry / Datadog RUM / LogRocket>
  - Error boundary / `error.tsx`, `not-found.tsx`
- **Security**:
  - CSP / security headers (`next.config` headers)
  - CORS, XSS / CSRF prevention
  - Sanitize HTML (`dangerouslySetInnerHTML` policy)
  - Dependency audit

## 6. Quality (Chất lượng)

- **Linting / formatting**: <ESLint config (next/core-web-vitals), Prettier, Stylelint>
- **Type checking**: `tsc --noEmit` trong CI
- **Testing strategy**:
  - Unit: <Vitest / Jest — coverage target>
  - Component: <@testing-library/react / Storybook interaction>
  - E2E: <Playwright / Cypress>
  - Visual regression: <Chromatic / Percy — nếu có>
- **Accessibility (a11y)**: <WCAG level target, axe / Lighthouse>
- **Git hooks**: <husky, lint-staged, commitlint>
````

## Lưu ý

- Không cài thêm package, không sửa code khác.
- Nếu dự án **không phải Next.js** (ví dụ CRA, Vite + React thuần, Vue, SvelteKit), dừng lại và thông báo cho người dùng.
- Nếu thư mục `.claude/` chưa tồn tại, tạo nó.
- Nếu `.claude/project.md` đã tồn tại, hỏi người dùng có muốn ghi đè không trước khi tiếp tục.
