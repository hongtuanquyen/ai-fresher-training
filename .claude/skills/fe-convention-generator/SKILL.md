---
name: fe-convention-generator
description: >
  Sinh nội dung conventions BẮT BUỘC tuân theo cho 9 file trong .claude/conventions/
  của dự án FE, dựa vào tech stack đã chốt trong .claude/FE-architecture.md.
  Mỗi convention kèm cặp ví dụ "❌ Sai / ✅ Đúng" + sample code thực thi được.
  Trigger khi user yêu cầu "tạo conventions FE", "gen convention frontend", "viết
  convention từ FE-architecture", hoặc bất cứ khi nào cần khởi tạo nội dung cho
  thư mục .claude/conventions/ dựa trên stack FE đã chốt.
  KHÔNG trigger cho backend conventions, code review, hoặc viết convention chung
  chung không dựa vào architecture file.
---

# Skill: fe-convention-generator

Bạn là agent sinh nội dung convention cho dự án FE. Bạn ĐỌC `.claude/FE-architecture.md`
để biết tech stack thực tế của dự án, rồi GHI nội dung convention vào 9 file trong
`.claude/conventions/`. Convention phải mang tính BẮT BUỘC (dùng từ "PHẢI", "KHÔNG
ĐƯỢC", "BẮT BUỘC"), KHÔNG phải gợi ý.

## Inputs

- File nguồn: `.claude/FE-architecture.md` (BẮT BUỘC tồn tại — nếu không, dừng và
  báo user chạy `web-tech-advisor` trước).
- Thư mục đích: `.claude/conventions/`. (Nếu không tồn tại thì tạo mới, nhưng KHÔNG
  được tạo file convention nào ngoài 9 file đích đã liệt kê bên dưới).
- 9 file đích (tên cố định, KHÔNG đổi):
  1. `naming-conventions.md`
  2. `component-structure.md`
  3. `styling-rules.md`
  4. `state-management.md`
  5. `api-integration.md`
  6. `error-handling.md`
  7. `commenting-rules.md`
  8. `i18n-policy.md`
  9. `monitoring.md`

## Quy tắc ghi file (BẮT BUỘC)

Với mỗi file đích:
1. Nếu file **không tồn tại** → tạo mới.
2. Nếu file **tồn tại nhưng rỗng** (0 bytes hoặc chỉ whitespace) → ghi đè.
3. Nếu file **đã có nội dung** (non-empty) → **SKIP**, KHÔNG ghi đè. Báo lại cho
   user file nào skip.

Kiểm tra trạng thái bằng `Read` (file rỗng sẽ trả system reminder báo empty) hoặc
`Bash: wc -c <file>` trước khi quyết định.

## Workflow

### Bước 1 — Đọc architecture
Dùng `Read` để đọc `.claude/FE-architecture.md`. Trích xuất:
- Framework + version (Next.js App Router, React version).
- Language (TypeScript).
- Styling (Tailwind, shadcn/ui).
- State (TanStack Query, Zustand).
- Form (react-hook-form, zod).
- HTTP client pattern (`fetch` wrapper, `credentials: 'include'`).
- Auth model (JWT httpOnly cookie).
- Cấu trúc thư mục đã chốt (đặc biệt `src/app/`, `src/components/`, `src/lib/`,
  `src/hooks/`, `src/stores/`, `src/types/`).

Convention sinh ra PHẢI khớp 100% với những gì có trong architecture. KHÔNG
được đề xuất stack khác (ví dụ không nói "dùng Redux" nếu architecture chốt
Zustand).

### Bước 2 — Quét trạng thái 9 file đích
Chạy `Bash: wc -c .claude/conventions/*.md` để biết file nào rỗng/non-empty.
Lập danh sách `to_write` và `to_skip`.

### Bước 3 — Sinh nội dung từng file

Quy trình 2 lớp:
1. **Lấy sườn chung** từ section "Hướng dẫn nội dung từng file" bên dưới —
   đây là chủ đề bắt buộc, KHÔNG được bỏ.
2. **Phát triển chi tiết theo stack thực tế** đã đọc ở Bước 1: thay placeholder
   chung bằng tên lib/API thật (vd "lib styling đã chốt" → "Tailwind + shadcn"
   nếu architecture chốt như vậy). Code mẫu PHẢI dùng đúng package, đúng import
   path, đúng API của stack thực tế. Nếu architecture KHÔNG nhắc tới mục nào
   (vd chưa chốt error tracker), giữ rule ở mức nguyên tắc chung và ghi chú
   "TBD theo lib chốt sau" — KHÔNG bịa lib.

Mỗi file PHẢI có:
- Header với tiêu đề + role (FE) + nguồn tham chiếu.
- Block "Phạm vi áp dụng" + "Mức độ bắt buộc" (luôn ghi: "MUST — vi phạm = block PR").
- Tối thiểu **5 rule**, mỗi rule có:
  - Tiêu đề rule (đánh số `## Rule N — ...`).
  - 1-2 câu giải thích **tại sao**.
  - Block `### ❌ Sai` với code snippet (TS/TSX) cho thấy lỗi.
  - Block `### ✅ Đúng` với code snippet sửa lại.
  - (Optional) `### Ghi chú` cho edge case.
- Footer "Checklist tự kiểm" — bullet list ngắn để dev tick trước khi commit.

Code mẫu PHẢI:
- Dùng đúng import path khớp cấu trúc thư mục trong architecture (`@/lib/api-client`,
  `@/components/ui/button`, v.v.).
- Là TypeScript hợp lệ (không pseudo-code).
- Tham chiếu đúng package name (vd `@tanstack/react-query`, không phải `react-query`).

### Bước 4 — Ghi file
Dùng `Write` lần lượt cho từng file trong `to_write`. KHÔNG dùng `Edit` (file rỗng).

### Bước 5 — Báo cáo
Trả về user:
- Danh sách file đã ghi (`to_write`).
- Danh sách file skip vì đã có nội dung (`to_skip`).
- Nhắc user review 9 file và điều chỉnh nếu cần.

## Hướng dẫn nội dung từng file (sườn chung — framework-agnostic)

Mỗi file dưới đây là **bộ khung chủ đề chung của frontend hiện đại**. Khi sinh
nội dung, bạn lấy bộ khung này làm sườn, **rồi phát triển chi tiết bằng tech
stack thực tế** đọc được từ `FE-architecture.md` (framework, styling lib,
state lib, form lib, HTTP client, i18n lib, tracker...). Nếu architecture
không nhắc tới 1 mục nào, giữ rule ở mức nguyên tắc chung, KHÔNG bịa lib.

### `naming-conventions.md` — Sườn
- Quy ước đặt tên file & folder (case style, số ít/nhiều).
- Quy ước đặt tên component, hook, util, store, type/interface, constant, enum.
- Quy ước biến boolean (prefix `is/has/can`), hàm event handler (`handle*` /
  `on*`), hàm async, hàm pure.
- Quy ước đặt tên key (query key, cache key, storage key, event name).
- Quy ước viết tắt: cấm/được phép, danh sách viết tắt thống nhất nếu có.
- Phát triển chi tiết theo stack (vd: nếu có file-based router thì áp quy ước
  của router đó; nếu có state lib thì quy ước tên store/slice).

### `component-structure.md` — Sườn
- Đơn vị 1 file 1 component, named vs default export (gắn với quy ước của
  framework).
- Phân tách Server / Client component (nếu framework có khái niệm này).
- Thứ tự khai báo trong file: imports → types → constants → component → sub.
- Giới hạn độ dài file/component, khi nào tách sub-component / extract hook.
- Props typing bắt buộc, cấm `any`, quy tắc destructure & default value.
- Co-location: test, story, style cạnh component.
- Phân lớp: presentational vs container, vị trí logic (hook riêng vs trong
  component).

### `styling-rules.md` — Sườn
- Phương pháp styling chính (utility-first / CSS-in-JS / CSS module) — tuân
  theo lib đã chốt; cấm trộn nhiều phương pháp.
- Cấm hard-code giá trị thiết kế (màu, spacing, font-size); PHẢI dùng token /
  theme variable từ design system.
- Conditional className PHẢI qua helper thống nhất (vd `cn`, `clsx`), KHÔNG
  concat string thủ công.
- Responsive mobile-first, breakpoint dùng đúng API của lib.
- Quy tắc về dark mode / theme switching nếu có.
- Tránh inline `style={{}}` trừ giá trị runtime không thể static — phải comment
  lý do.
- Cách extend / override component của UI lib (nếu có shadcn / MUI / Chakra...):
  qua variant API, KHÔNG override class lặp lại ở call-site.

### `state-management.md` — Sườn
- Phân loại state rõ ràng:
  - **Server state** (data từ API) — dùng lib server-state đã chốt.
  - **URL state** (filter, pagination) — dùng router/search params.
  - **Client state cục bộ** — `useState` / `useReducer` (hoặc tương đương).
  - **Client state global** — store lib đã chốt.
- Cấm duplicate server state vào client store (gây stale).
- Quy ước query/mutation: cấu trúc key, stale time, retry, invalidation sau
  mutation.
- Form state PHẢI dùng form lib đã chốt + schema validation; cấm quản lý field
  rời rạc bằng `useState`.
- Persist state (localStorage / cookie): chỉ cho state thật sự cần, có
  versioning, KHÔNG persist token nhạy cảm vào localStorage.

### `api-integration.md` — Sườn
- Mọi HTTP request PHẢI đi qua **một** HTTP client tập trung (wrapper / instance);
  cấm gọi `fetch`/`axios` trực tiếp rải rác.
- Base URL, header auth, credential mode chỉ cấu hình 1 chỗ.
- Endpoint path tập trung (constants/registry) hoặc co-locate trong hook —
  thống nhất 1 cách, KHÔNG trộn.
- Request/response type BẮT BUỘC khai báo, không dùng `any`.
- Validate input phía client trước khi gửi (qua schema lib đã chốt).
- Xử lý lỗi: client throw → tầng data layer bắt; cấm `try/catch` nuốt lỗi rồi
  return null/undefined.
- Quy ước đặt tên hook data (`useXxxQuery`, `useXxxMutation`...) — chốt 1 pattern.
- Cancellation, timeout, retry policy: nêu rõ default + khi nào override.

### `error-handling.md` — Sườn
- Phân tầng xử lý lỗi: **boundary UI** (cho lỗi render), **data layer** (cho
  lỗi fetch/mutation), **form** (cho lỗi validate), **global** (cho lỗi
  uncaught).
- Mỗi route/feature quan trọng PHẢI có error boundary fallback (theo API của
  framework).
- Cấm `catch {}` rỗng; cấm `alert()` để báo lỗi UI.
- Quy ước message hiển thị cho user: human-friendly, KHÔNG phơi raw message từ
  server / stack trace.
- Phân loại HTTP status: 401/403 → flow auth (logout, redirect); 4xx khác →
  hiển thị inline; 5xx → toast/banner generic.
- Mọi lỗi không-mong-đợi PHẢI được log qua logger tập trung (xem `monitoring.md`).

### `commenting-rules.md` — Sườn
- Comment giải thích **tại sao**, KHÔNG giải thích **cái gì** (tên đã làm việc đó).
- Comment bắt buộc cho: workaround, business rule không hiển nhiên, browser/lib
  quirk, performance hack.
- TODO/FIXME PHẢI có owner + ngày + ticket (nếu có).
- JSDoc/TSDoc cho hàm export public của module dùng chung; KHÔNG ép cho hàm
  nội bộ component.
- Cấm code commented-out (dùng git history).
- Cấm comment kiểu changelog trong file (dùng commit message / CHANGELOG).
- Magic number/string cần comment HOẶC đặt tên hằng số rõ ràng.

### `i18n-policy.md` — Sườn
- Cấm hard-code chuỗi UI rải rác trong JSX/template, kể cả khi app chỉ 1 ngôn
  ngữ — luôn đi qua 1 lớp tra cứu (lib i18n đã chốt, hoặc messages object
  trung tâm nếu chưa cần lib).
- Quy ước key i18n: namespace theo feature/route, snake_case hoặc dot.notation —
  thống nhất 1.
- Cấm trộn logic + text trong template string (`Bạn có ${n} từ`) — dùng API
  formatting/plural của lib.
- Date/number/currency PHẢI qua `Intl.*` hoặc API của lib, locale theo cấu
  hình app, KHÔNG hard-code format.
- Plural & gender PHẢI dùng API tương ứng (ICU MessageFormat / Intl.PluralRules).
- Khi thêm chuỗi mới: thêm cho TẤT CẢ locale đang support; cấm để key thiếu
  bản dịch lên production.

### `monitoring.md` — Sườn
- Mọi log đi qua **logger wrapper tập trung** (vd `lib/logger`); cấm
  `console.log` rải rác trong code merge vào main.
- Phân cấp log: `debug` / `info` / `warn` / `error` — quy ước rõ khi nào dùng.
- Error tracking (Sentry/Datadog/...) cấu hình 1 chỗ; component KHÔNG gọi SDK
  trực tiếp.
- Analytics/event tracking đi qua wrapper, đặt tên event theo convention
  thống nhất (vd `feature.action.result`).
- Cấm log dữ liệu nhạy cảm: password, token, PII của user khác, payload
  thanh toán.
- Performance: theo dõi Core Web Vitals (LCP, INP, CLS) qua API của framework
  hoặc lib chuyên dụng; nêu ngưỡng cảnh báo.
- Component render nặng PHẢI tối ưu (memo / virtualize / code-split) — quy ước
  ngưỡng và cách đo.

## Nguyên tắc khi viết

- **Tiếng Việt** mặc định (giải thích + checklist), code snippet giữ nguyên
  tiếng Anh chuẩn.
- **Bắt buộc tone**: "PHẢI", "KHÔNG ĐƯỢC", "BẮT BUỘC". Tránh "nên", "có thể",
  "khuyến nghị" — đây là convention.
- **Mỗi rule = 1 cặp Sai/Đúng**. Code phải runnable, import path khớp project.
- **KHÔNG sao chép** rule từ project khác mà không phù hợp stack đã chốt
  (ví dụ không viết rule về Redux nếu dùng Zustand).
- **Không tạo file ngoài 9 file** đã liệt kê. Nếu có chủ đề thừa, gộp vào file
  gần nghĩa nhất.

## Sample format 1 rule (template tham chiếu)

```markdown
## Rule 3 — Mọi request HTTP PHẢI đi qua `api()` wrapper

Lý do: cookie auth, error handling, base URL phải tập trung 1 chỗ. Gọi `fetch`
trực tiếp sẽ rò rỉ logic và quên `credentials: 'include'`.

### ❌ Sai

\`\`\`tsx
// src/hooks/use-folders.ts
export function useFolders() {
  return useQuery({
    queryKey: ['folders'],
    queryFn: async () => {
      const res = await fetch('http://localhost:3001/api/v1/folders');
      return res.json();
    },
  });
}
\`\`\`

### ✅ Đúng

\`\`\`tsx
// src/hooks/use-folders.ts
import { api } from '@/lib/api-client';
import type { Folder } from '@/types/api';

export function useFolders() {
  return useQuery({
    queryKey: ['folders'],
    queryFn: () => api<Folder[]>('/folders'),
  });
}
\`\`\`
```
