# E-commerce — Tech Stack Reference

## Đặc điểm archetype
- SEO cực kỳ quan trọng (product page, category page phải crawlable)
- Performance critical: LCP ảnh hưởng conversion rate trực tiếp
- Image-heavy: cần optimization pipeline
- Checkout flow: form phức tạp, payment integration, cart state
- Search & filter: faceted search, sort, pagination
- Auth: customer account, order history, wishlist
- i18n / multi-currency phổ biến
- Analytics & tracking: conversion funnel, A/B test
- Compliance: GDPR cookie consent, PCI DSS (nếu xử lý card)

## Stack combinations đã curate

### Combo 1: Next.js + Tailwind + Shopify Storefront API (khuyến nghị nếu dùng Shopify)
- **Framework**: Next.js 15 App Router
- **Commerce**: Shopify Storefront API (headless) + Shopify Checkout
- **Styling**: Tailwind CSS
- **UI**: shadcn/ui
- **State**: Zustand (cart) + TanStack Query (product data)
- **Search**: Shopify built-in hoặc Algolia
- **Payment**: Shopify Checkout (hosted, PCI compliant)
- **Image**: next/image + Shopify CDN
- **Analytics**: Vercel Analytics + GA4 + Meta Pixel
- **Deploy**: Vercel
- **Khi nào**: đã dùng Shopify cho inventory/order, muốn custom storefront, team React
- **Khi nào không**: không dùng Shopify, cần full control payment flow

### Combo 2: Next.js + Medusa.js (open-source commerce)
- **Framework**: Next.js 15
- **Commerce engine**: Medusa.js (headless, self-host, extensible)
- **Styling**: Tailwind CSS
- **UI**: shadcn/ui
- **State**: Zustand + TanStack Query
- **Search**: MeiliSearch / Algolia
- **Payment**: Stripe / PayPal via Medusa plugin
- **Image**: next/image + Cloudinary / imgix
- **Deploy**: Vercel (frontend) + Railway/Render (Medusa backend)
- **Khi nào**: cần control toàn bộ, không muốn vendor lock-in, team có khả năng quản lý backend
- **Khi nào không**: team nhỏ không muốn quản lý infra, cần đi nhanh

### Combo 3: Nuxt 3 + Saleor / Vendure
- **Framework**: Nuxt 3
- **Commerce engine**: Saleor (Python/GraphQL) hoặc Vendure (Node/TypeScript)
- **Styling**: Tailwind / UnoCSS
- **UI**: Nuxt UI / PrimeVue
- **State**: Pinia
- **Search**: Algolia / Typesense
- **Payment**: Stripe / Adyen
- **Image**: Nuxt Image + Cloudinary
- **Deploy**: Vercel / Netlify / Node server
- **Khi nào**: team Vue, cần GraphQL commerce backend, SEO tốt với Nuxt SSR
- **Khi nào không**: team React

### Combo 4: Remix + Stripe + custom (full control)
- **Framework**: Remix
- **Commerce**: custom (tự build product/cart/order logic)
- **Styling**: Tailwind CSS
- **State**: Remix loaders/actions (server state), Zustand (client state nhẹ)
- **Payment**: Stripe Elements + Stripe API
- **Search**: Typesense / MeiliSearch
- **Image**: Cloudinary / Imgix
- **Deploy**: Vercel / Fly.io
- **Khi nào**: sản phẩm niche (subscription box, digital goods, marketplace) không fit Shopify/Medusa model, cần full control
- **Khi nào không**: standard e-commerce, không muốn build cart/checkout từ đầu

## Layer bổ sung thường gặp

| Nhu cầu | Khuyến nghị |
|---|---|
| Product search | Algolia / Typesense / MeiliSearch |
| Image CDN | Cloudinary / imgix / Vercel Image Optimization |
| Reviews/ratings | Judge.me API / Yotpo / custom |
| Email transactional | Resend / SendGrid / Amazon SES |
| Inventory sync | Webhook + queue (nếu multi-channel) |
| Cookie consent | CookieYes / Osano / custom banner |
| A/B testing | Vercel Flags / PostHog / VWO |
| Monitoring | Vercel Analytics + Sentry + Web Vitals |
| Sitemap | next-sitemap / nuxt-simple-sitemap |
| Structured data | JSON-LD cho Product, BreadcrumbList, Organization |
| Multi-currency | Dinero.js / currency.js |
| Cart persistence | localStorage + server sync |

## Anti-patterns

- Client-side only rendering cho product page → SEO chết
- Không optimize image → LCP > 4s, bounce rate tăng
- Xử lý payment ở client-side → security risk, PCI violation
- Không có structured data (JSON-LD) → mất rich snippets
- Cart state chỉ ở client → mất khi đổi device
- Không pre-render category page → crawl budget lãng phí
- Bundle toàn bộ product catalog vào client → initial load quá nặng
