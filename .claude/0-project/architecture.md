# Frontend Architecture — English Learning App

> **Stack đã chốt**: Next.js (App Router) + TypeScript
> **Repo**: độc lập, không mono-repo. Repo gợi ý: `english-learning-web`
> **Deadline**: 3 ngày

---

## 1. Kiến trúc tổng thể

```
┌──────────────────────────┐         ┌──────────────────────────┐
│  english-learning-web    │ ──────▶ │  english-learning-api    │
│  Next.js 15 App Router   │  REST   │  NestJS (repo riêng)     │
│  Port 3000               │  +JWT   │  Port 3001               │
│                          │  cookie │                          │
└──────────────────────────┘         └──────────────────────────┘
        │
        ├──▶ Web Speech API (browser native — đọc từ vựng, KHÔNG gọi BE)
        └──▶ papaparse (parse CSV phía client → gửi JSON lên BE)
```

**Điểm quan trọng khi tách 2 repo:**
- FE **KHÔNG** import code từ BE. Type response BE → định nghĩa lại trong `src/types/api.ts` (copy tay, hoặc generate từ OpenAPI nếu có thời gian).
- CORS: BE phải allow origin `http://localhost:3000` và domain prod, `credentials: true`.
- Auth: nhận JWT trong **httpOnly cookie** do BE set (cross-site cookie cần `SameSite=None; Secure` ở prod). Local dev cùng `localhost` khác port → set `SameSite=Lax` là đủ.
- Env: FE chỉ cần biết 1 biến `NEXT_PUBLIC_API_BASE_URL` trỏ tới BE.

---

## 2. Tech stack

| Layer | Chọn | Lý do |
|---|---|---|
| Framework | **Next.js 15 App Router** | Đã chốt |
| Language | TypeScript | Type-safe |
| Styling | **Tailwind CSS v4** | Setup nhanh, không cần design tinh |
| UI components | **shadcn/ui** | Copy-paste, kiểm soát hoàn toàn, đẹp sẵn — không phải lib |
| State server | **TanStack Query (React Query)** | Cache + refetch + optimistic update cho flashcard/test |
| State client | **Zustand** | State phiên test (câu hiện tại, đáp án) — chỉ khi cần |
| Form | **react-hook-form + zod** | Form import từ, tạo folder |
| CSV parse | **papaparse** | Parse CSV phía client → gửi JSON lên BE (đỡ phải xử lý multipart) |
| Audio | **Web Speech API** (`window.speechSynthesis`) | Free, không cần BE, không cần API key |
| Charts | **Recharts** | Dashboard tiến độ |
| HTTP | `fetch` + custom wrapper | Không cần axios |
| Auth | JWT trong httpOnly cookie do BE set | FE chỉ cần `credentials: 'include'` |

---

## 3. Cấu trúc thư mục

```
english-learning-web/                 # Repo FE độc lập
├── src/
│   ├── app/                          # App Router
│   │   ├── (auth)/
│   │   │   ├── login/page.tsx
│   │   │   └── register/page.tsx
│   │   ├── (dashboard)/
│   │   │   ├── layout.tsx                   # Sidebar + auth guard
│   │   │   ├── page.tsx                     # Dashboard tiến độ
│   │   │   ├── folders/
│   │   │   │   ├── page.tsx                 # List folders
│   │   │   │   ├── new/page.tsx             # Tạo folder
│   │   │   │   └── [id]/
│   │   │   │       ├── page.tsx             # Chi tiết folder + list từ
│   │   │   │       ├── import/page.tsx      # Import CSV / nhập tay
│   │   │   │       └── test/
│   │   │   │           ├── flashcard/page.tsx
│   │   │   │           ├── quiz/page.tsx
│   │   │   │           └── ai-text/page.tsx # Optional
│   │   ├── layout.tsx
│   │   └── globals.css
│   ├── components/
│   │   ├── ui/                       # shadcn/ui copy ra đây
│   │   ├── flashcard/
│   │   │   ├── flashcard-deck.tsx
│   │   │   └── flashcard-item.tsx
│   │   ├── quiz/
│   │   │   ├── quiz-runner.tsx
│   │   │   └── quiz-result.tsx
│   │   ├── vocabulary/
│   │   │   ├── word-form.tsx                # Form thêm từ thủ công
│   │   │   ├── csv-import.tsx               # Parse CSV bằng papaparse
│   │   │   └── word-card.tsx                # Card hiện từ + IPA + nút phát âm
│   │   ├── dashboard/
│   │   │   ├── progress-chart.tsx
│   │   │   └── stats-cards.tsx
│   │   └── layout/
│   │       ├── sidebar.tsx
│   │       └── header.tsx
│   ├── lib/
│   │   ├── api-client.ts             # fetch wrapper, credentials: 'include'
│   │   ├── speech.ts                 # Web Speech API helper
│   │   ├── csv.ts                    # papaparse wrapper
│   │   └── utils.ts                  # cn(), formatDate, etc.
│   ├── hooks/
│   │   ├── use-folders.ts            # TanStack Query
│   │   ├── use-vocabulary.ts
│   │   ├── use-test-session.ts
│   │   └── use-auth.ts
│   ├── stores/
│   │   └── test-session-store.ts     # Zustand: state phiên test
│   ├── types/
│   │   └── api.ts                    # Type response BE (copy tay từ BE DTO)
│   └── middleware.ts                 # Redirect nếu chưa có cookie auth
├── public/
├── .env.local                        # NEXT_PUBLIC_API_BASE_URL=http://localhost:3001/api/v1
├── .env.example
├── next.config.ts
├── tailwind.config.ts
├── tsconfig.json
├── package.json
├── .gitignore
└── README.md
```

---

## 4. API client pattern (FE gọi BE)

```ts
// src/lib/api-client.ts
const BASE = process.env.NEXT_PUBLIC_API_BASE_URL!;

export async function api<T>(path: string, init?: RequestInit): Promise<T> {
  const res = await fetch(`${BASE}${path}`, {
    ...init,
    credentials: 'include',           // gửi cookie JWT
    headers: { 'Content-Type': 'application/json', ...init?.headers },
  });
  if (!res.ok) throw new Error(await res.text());
  return res.json();
}
```

Mọi hook TanStack Query gọi qua `api()`. Auth state suy ra từ kết quả `GET /auth/me`.

---

## 5. Routes & màn hình

**Tổng số page dự tính: 11** (10 bắt buộc + 1 optional)
— gồm 2 page Auth, 1 Dashboard, 4 Folder/Vocabulary, 3 Test (1 optional), 1 NotFound.

| # | Route | Tên page | Loại | Chức năng phục vụ |
|---|---|---|---|---|
| 1 | `/login` | LoginPage | Auth | Form đăng nhập (email + password) → BE set httpOnly cookie → redirect `/`. |
| 2 | `/register` | RegisterPage | Auth | Form đăng ký tài khoản mới → auto login → redirect `/`. |
| 3 | `/` | DashboardPage | Dashboard | Tổng quan tiến độ học: số từ tổng, số session, accuracy chung, biểu đồ tuần (Recharts), list session gần nhất. |
| 4 | `/folders` | FoldersListPage | Folder | Danh sách folder của user kèm số từ mỗi folder + nút "Tạo folder" + vào chi tiết. |
| 5 | `/folders/new` | NewFolderPage | Folder | Form tạo folder mới (name + description) → redirect về `/folders/[id]`. |
| 6 | `/folders/[id]` | FolderDetailPage | Folder | Chi tiết folder: list từ vựng (word + meaning + IPA + nút phát âm), nút "Import từ", nút chọn loại test (Flashcard / Quiz / AI Text). Cho phép xóa từ. |
| 7 | `/folders/[id]/import` | ImportVocabularyPage | Vocabulary | 2 tab: "Nhập tay" (form thêm 1 từ: word, meaning, IPA, example) + "Import CSV" (upload + preview + bulk submit qua papaparse). |
| 8 | `/folders/[id]/test/flashcard` | FlashcardTestPage | Test | Phiên test flashcard: lật thẻ 2 mặt (word ↔ meaning), hiện IPA, nút phát âm Web Speech, next/prev, đánh dấu đúng/sai → submit kết quả về BE. |
| 9 | `/folders/[id]/test/quiz` | QuizTestPage | Test | Phiên trắc nghiệm: mỗi câu 1 từ + 4 đáp án (1 đúng, 3 distractor random từ folder), tracking đúng/sai → màn hình kết quả → submit BE. |
| 10 | `/folders/[id]/test/ai-text` | AiTextTestPage | Test (Optional) | Chọn ≤ 10 từ trong folder → BE gọi Gemini sinh đoạn văn tiếng Anh dùng các từ đó → hiển thị đoạn text + cho phép phát âm. **Cắt nếu hết deadline.** |
| 11 | `*` (not-found.tsx) | NotFoundPage | System | Trang 404 cho route không tồn tại. |

### Layout & guard

- **`(auth)` group** (`/login`, `/register`): layout đơn giản căn giữa, không sidebar. Nếu đã đăng nhập → redirect `/`.
- **`(dashboard)` group** (page #3 → #10): layout có sidebar + header. Middleware Next.js check cookie auth, chưa đăng nhập → redirect `/login`.
- **Error boundary** (`error.tsx`) đặt ở các segment `[id]` và `test/*` để bắt lỗi load folder / submit session.

---

## 6. Cắt scope 3 ngày (FE)

- **Day 1**: Setup project, Tailwind + shadcn, Auth pages, layout dashboard, list/create folder.
- **Day 2**: Chi tiết folder, form nhập từ, CSV import, Flashcard component, Quiz component.
- **Day 3**: Submit kết quả test, Dashboard stats + Recharts, Web Speech audio, polish. **AI text làm cuối — cắt nếu hết giờ.**

---

## 7. Setup commands

```bash
pnpm create next-app@latest english-learning-web --typescript --tailwind --app --eslint
cd english-learning-web
pnpm add @tanstack/react-query zustand react-hook-form zod papaparse recharts
pnpm add -D @types/papaparse
pnpx shadcn@latest init
pnpx shadcn@latest add button input card dialog form tabs
```

`.env.local`:
```
NEXT_PUBLIC_API_BASE_URL=http://localhost:3001/api/v1
```
