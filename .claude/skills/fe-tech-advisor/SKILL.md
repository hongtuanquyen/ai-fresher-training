---
name: fe-tech-advisor
description: >
  Đọc yêu cầu dự án frontend web (từ chat hoặc file spec), phân tích nhu cầu, rồi đề xuất 2-3 option tech stack hoàn chỉnh để so sánh. Mỗi option gồm: danh sách công nghệ + lý do chọn + bảng pros/cons + folder structure + config files mẫu. Trigger skill này khi user hỏi về lựa chọn công nghệ cho dự án frontend mới, cần tư vấn tech stack web, muốn so sánh framework/library, hỏi "nên dùng gì cho dự án X", hoặc đưa spec/requirement và muốn biết nên dùng stack nào. Cũng trigger khi user đưa file PDF/doc/markdown chứa yêu cầu dự án và muốn đề xuất công nghệ. KHÔNG trigger cho câu hỏi cụ thể về cách dùng 1 framework đã chọn (ví dụ "cách dùng useEffect trong React"), chỉ trigger khi cần TƯ VẤN CHỌN stack.
---

# FE Tech Stack Advisor

Skill phân tích yêu cầu dự án frontend web và đề xuất 2-3 option tech stack hoàn chỉnh để so sánh.

## Khi nào dùng skill này

- User mô tả dự án frontend mới và hỏi nên dùng công nghệ gì
- User upload file spec (PDF/doc/markdown/Excel) và muốn tư vấn stack
- User muốn so sánh nhiều framework/library cho dự án cụ thể
- User hỏi dạng: "nên dùng React hay Vue cho dự án X?", "tech stack nào phù hợp?", "setup dự án web mới cần gì?"

## Workflow

### Bước 1 — Thu thập yêu cầu

Đọc input từ user (chat hoặc file). Trích xuất các tín hiệu sau:

```
PROJECT SIGNALS:
├── Loại dự án: landing page | marketing site | web app (SaaS/dashboard/admin) | e-commerce | blog/CMS | portfolio | hybrid
├── Quy mô: nhỏ (<10 page) | vừa (10-50 page) | lớn (>50 page/module)
├── Team size: solo | small (2-5) | medium (5-15) | large (>15)
├── Timeline: gấp (<1 tháng) | bình thường (1-3 tháng) | dài hạn (>3 tháng)
├── SEO: quan trọng | nice-to-have | không cần
├── Realtime: cần (chat, notification, live data) | không
├── Auth: cần | không | OAuth/social login
├── i18n: cần | không | sau này mới cần
├── a11y: bắt buộc (WCAG AA/AAA) | nice-to-have | không yêu cầu
├── Offline: cần (PWA) | không
├── Performance: critical (e-commerce, media) | normal
├── Design system: có sẵn (Figma/Sketch) | tự build | dùng UI library
├── API backend: có sẵn (REST/GraphQL) | tự build cùng | BaaS (Firebase/Supabase) | chưa có
├── Deploy target: Vercel | AWS | self-hosted | chưa biết
├── Budget: có tiền cho paid service | chỉ open-source | flexible
├── Team experience: React | Vue | Angular | Svelte | không có preference
└── Ràng buộc đặc biệt: legacy integration | specific browser support | compliance
```

**Nếu thiếu thông tin quan trọng**: hỏi user TỐI ĐA 3 câu gộp. Không phỏng vấn dài. Nếu user không biết → dùng giá trị mặc định hợp lý và ghi rõ assumption.

**Nếu có file spec**: đọc file trước, trích xuất signal, rồi chỉ hỏi những gì file không cover.

### Bước 2 — Phân loại dự án

Dựa trên tín hiệu, xác định **project archetype**. Đọc file reference tương ứng:

| Archetype | File reference | Khi nào |
|---|---|---|
| Static / Landing | `references/static-landing.md` | Landing page, marketing, blog, portfolio, ít tương tác |
| Web App | `references/web-app.md` | Dashboard, SaaS, admin panel, CRUD-heavy, nhiều tương tác |
| E-commerce | `references/e-commerce.md` | Product listing, cart, checkout, payment, SEO quan trọng |
| Hybrid / Full-stack | `references/hybrid-fullstack.md` | Cần cả SSR/SSG + app-like interaction, hoặc full-stack framework |

→ **Đọc file reference tương ứng** trước khi đề xuất. File chứa các stack combination đã được curate cho archetype đó.

### Bước 3 — Đề xuất 2-3 option

Mỗi option là **một bộ tech stack hoàn chỉnh**, không phải chỉ framework. Output theo template dưới đây.

**Nguyên tắc chọn option**:
- Option phải **khác nhau có ý nghĩa** (không phải chỉ đổi CSS library)
- Mỗi option phải **giải quyết được yêu cầu** — không đề xuất stack không phù hợp chỉ để "cho đa dạng"
- Ưu tiên stack có **ecosystem mạnh, maintained tốt, community lớn** tại thời điểm hiện tại
- Nếu team đã có kinh nghiệm với framework cụ thể → option đầu tiên nên dùng framework đó (giảm learning curve)
- Ghi rõ **khi nào option này tốt hơn option kia** — không để user tự đoán

### Bước 4 — Output theo template

Với **MỖI option**, output đầy đủ các phần sau:

---

#### TEMPLATE OUTPUT (lặp cho mỗi option)

```markdown
## Option [A/B/C]: <Tên ngắn gọn> (ví dụ: "Next.js + Tailwind + Zustand")

### Tổng quan
<1-2 câu mô tả approach của option này và khi nào nên chọn>

### Tech Stack

| Layer | Công nghệ | Version | Lý do chọn |
|---|---|---|---|
| Framework | Next.js | 15.x | SSR/SSG, file-based routing, React ecosystem |
| Language | TypeScript | 5.x | Type safety, DX tốt, catch bug sớm |
| Styling | Tailwind CSS | 4.x | Utility-first, design token dễ, purge nhỏ bundle |
| State | Zustand | 5.x | Nhẹ, đơn giản, không boilerplate |
| Data fetching | TanStack Query | 5.x | Cache, retry, background refetch |
| Form | React Hook Form + Zod | | Validation type-safe, performance tốt |
| UI components | shadcn/ui | | Customizable, accessible, copy-paste |
| Testing | Vitest + Playwright | | Unit nhanh, E2E reliable |
| Linting | ESLint + Prettier | | Code quality + format nhất quán |
| i18n | next-intl | | Nếu cần i18n (bỏ nếu không) |
| Auth | NextAuth.js | | Nếu cần auth (bỏ nếu không) |
| Deploy | Vercel | | Zero-config cho Next.js |

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
│   ├── app/                    # App Router (Next.js 15)
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   ├── (auth)/             # Route group
│   │   │   ├── login/
│   │   │   └── register/
│   │   └── dashboard/
│   │       ├── layout.tsx
│   │       └── page.tsx
│   ├── components/
│   │   ├── ui/                 # shadcn/ui components
│   │   ├── forms/
│   │   └── layouts/
│   ├── hooks/
│   ├── lib/                    # Utilities, API client
│   ├── stores/                 # Zustand stores
│   ├── types/
│   └── styles/
│       └── globals.css
├── public/
├── tests/
│   ├── unit/
│   └── e2e/
├── .env.example
├── next.config.ts
├── tailwind.config.ts
├── tsconfig.json
├── vitest.config.ts
├── playwright.config.ts
└── package.json
\```

### Config Files mẫu

<Tạo file thật cho các config quan trọng nhất — xem quy tắc bên dưới>
```

---

#### QUY TẮC CONFIG FILES

Tạo **file thật** (dùng create_file) cho các config sau, đặt trong `/mnt/user-data/outputs/option-[a|b|c]/`:

**Bắt buộc tạo** (mọi option):
1. `package.json` — dependencies đầy đủ, scripts chuẩn (dev, build, test, lint)
2. `tsconfig.json` — strict mode, path alias
3. Config framework chính (vd `next.config.ts`, `vite.config.ts`, `nuxt.config.ts`)
4. `.env.example` — env vars cần thiết
5. `.eslintrc.json` hoặc `eslint.config.js`

**Tạo nếu có trong stack**:
6. `tailwind.config.ts` (nếu dùng Tailwind)
7. `vitest.config.ts` hoặc `jest.config.ts` (nếu có testing)
8. `playwright.config.ts` (nếu có E2E)

**Không tạo** (quá chi tiết, user tự setup):
- Dockerfile, CI/CD, Terraform, k8s
- Component code mẫu (trừ khi user hỏi riêng)

### Bước 5 — Bảng so sánh tổng hợp

Sau khi output tất cả option, tạo **1 bảng so sánh duy nhất**:

```markdown
## So sánh tổng hợp

| Tiêu chí | Option A | Option B | Option C |
|---|---|---|---|
| Learning curve | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ |
| Performance | ... | ... | ... |
| SEO | ... | ... | ... |
| Ecosystem/community | ... | ... | ... |
| Bundle size | ... | ... | ... |
| DX (Developer Experience) | ... | ... | ... |
| Scalability | ... | ... | ... |
| Time to MVP | ... | ... | ... |
| Chi phí vận hành | ... | ... | ... |

### Đề xuất của tôi
<Dựa trên yêu cầu cụ thể của user, chọn 1 option và giải thích ngắn tại sao>
```

## Lưu ý quan trọng

1. **Luôn search web** cho version mới nhất của framework/library trước khi đề xuất. Không dùng version cũ từ training data.
2. **Config file phải chạy được** — không placeholder, không `// TODO`. User copy về phải `npm install && npm run dev` được.
3. **Không đề xuất stack đã deprecated** hoặc maintenance mode (ví dụ: Create React App, Gatsby trừ khi user chỉ định).
4. **Nếu user đã chỉ định framework**: vẫn đề xuất 2 option nhưng option đầu tiên dùng framework user chọn, option 2 là alternative với giải thích tại sao có thể tốt hơn.
5. **Ghi rõ assumption**: nếu thiếu thông tin và dùng default, ghi rõ "Giả định: ..." để user biết và có thể điều chỉnh.
6. **Tiếng Việt**: output bằng tiếng Việt trừ khi user viết tiếng Anh.
