# Static / Landing Page — Tech Stack Reference

## Đặc điểm archetype
- Ít tương tác, chủ yếu hiển thị nội dung
- SEO rất quan trọng (marketing, lead gen)
- Performance critical (Core Web Vitals ảnh hưởng ranking)
- Ít state management, ít form phức tạp
- Deploy tĩnh hoặc edge
- CMS integration phổ biến (blog, content update không cần deploy)

## Stack combinations đã curate

### Combo 1: Astro + Tailwind (khuyến nghị mặc định)
- **Framework**: Astro — zero JS by default, island architecture, multi-framework support
- **Styling**: Tailwind CSS
- **CMS**: Astro Content Collections (local) hoặc Sanity/Contentful (headless CMS)
- **Deploy**: Vercel / Netlify / Cloudflare Pages
- **Khi nào**: content-heavy, cần tốc độ tối đa, SEO là ưu tiên #1
- **Khi nào không**: cần nhiều tương tác phức tạp (form wizard, real-time)

### Combo 2: Next.js (Static Export) + Tailwind
- **Framework**: Next.js với `output: 'export'` (SSG)
- **Styling**: Tailwind CSS
- **UI**: shadcn/ui
- **Deploy**: Vercel / bất kỳ static host
- **Khi nào**: team đã biết React, landing page nhưng có thể scale lên web app sau, cần component library mạnh
- **Khi nào không**: overkill nếu chỉ cần trang tĩnh đơn giản

### Combo 3: Nuxt (Static) + UnoCSS
- **Framework**: Nuxt 3 với `nuxt generate` (SSG)
- **Styling**: UnoCSS hoặc Tailwind
- **UI**: Nuxt UI
- **Deploy**: Vercel / Netlify
- **Khi nào**: team dùng Vue, cần SEO tốt + DX của Vue
- **Khi nào không**: team không biết Vue, không lý do chuyển

### Combo 4: SvelteKit (Static) + Tailwind
- **Framework**: SvelteKit với `@sveltejs/adapter-static`
- **Styling**: Tailwind CSS
- **Deploy**: Vercel / Cloudflare Pages
- **Khi nào**: muốn bundle nhỏ nhất, team sẵn sàng học Svelte, performance extreme
- **Khi nào không**: team lớn cần ecosystem mature + hiring dễ

## Layer bổ sung thường gặp

| Nhu cầu | Khuyến nghị |
|---|---|
| Animation | Framer Motion (React) / Motion One (framework-agnostic) / GSAP |
| Form liên hệ | Formspree / Netlify Forms / custom API route |
| Analytics | Plausible / Umami (privacy-friendly) / Google Analytics |
| CMS | Sanity / Contentful / Strapi (self-host) / Keystatic (git-based) |
| Image optimization | Framework built-in (next/image, astro:assets) / Cloudinary |
| Font | `next/font` / Fontsource / self-host |
| Email | Resend / SendGrid / Amazon SES |
| A/B testing | Vercel Flags / PostHog / custom |

## Anti-patterns

- Dùng SPA framework (CRA, Vite React thuần) cho landing page → SEO kém, FCP chậm
- Dùng full-stack framework (Remix, Rails) cho trang 5 page tĩnh → overkill
- Không optimize image → LCP fail
- Import toàn bộ icon library → bundle bloat
