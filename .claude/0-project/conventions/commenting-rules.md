# Commenting Rules — FE

> **Role**: Frontend
> **Nguồn tham chiếu**: `.claude/FE-architecture.md`

**Phạm vi áp dụng**: mọi file `.ts`, `.tsx` trong `src/`.
**Mức độ bắt buộc**: **MUST — vi phạm = block PR**.

---

## Rule 1 — Comment giải thích **TẠI SAO**, KHÔNG giải thích **CÁI GÌ**

Lý do: tên biến/hàm tốt đã nói "cái gì". Comment lặp lại là noise và sẽ rot khi code thay đổi.

### ❌ Sai

```ts
// Lấy danh sách folders
const folders = await api<Folder[]>('/folders');

// Tăng index lên 1
setIndex((i) => i + 1);
```

### ✅ Đúng

```ts
const folders = await api<Folder[]>('/folders');

// Web Speech API trên Safari iOS yêu cầu user gesture trước lần phát đầu;
// gọi 1 lần với utterance rỗng để "unlock" engine.
window.speechSynthesis.speak(new SpeechSynthesisUtterance(''));
```

---

## Rule 2 — Comment BẮT BUỘC cho: workaround, business rule không hiển nhiên, browser/lib quirk, perf hack

Lý do: 6 tháng sau, không ai (kể cả tác giả) nhớ tại sao có dòng đó.

### ❌ Sai

```ts
const debounced = useMemo(() => debounce(search, 300), [search]);
```

### ✅ Đúng

```ts
// BE rate-limit /search 5 req/s; debounce 300ms tránh 429 khi user gõ nhanh.
const debounced = useMemo(() => debounce(search, 300), [search]);
```

---

## Rule 3 — TODO/FIXME PHẢI có owner + ngày (+ ticket nếu có); CẤM TODO trống

Lý do: TODO không owner = TODO vĩnh viễn.

### ❌ Sai

```ts
// TODO: handle empty list
// FIXME: edge case
```

### ✅ Đúng

```ts
// TODO(quyen, 2026-05-15): hỗ trợ paginate khi folder > 200 từ. Ticket: ENG-42
// FIXME(quyen, 2026-05-12): Recharts tooltip lệch trên Safari < 17 — chờ upstream fix.
```

---

## Rule 4 — JSDoc/TSDoc CHỈ cho hàm export public của module dùng chung (`src/lib/`, `src/hooks/`); CẤM ép cho component nội bộ

Lý do: TS đã làm 90% công việc của JSDoc. JSDoc cho component nội bộ chỉ tạo noise.

### ❌ Sai

```tsx
/**
 * WordCard component
 * @param props - props của component
 * @returns JSX element
 */
export function WordCard(props: WordCardProps) { /* ... */ }
```

### ✅ Đúng

```ts
// src/lib/speech.ts
/**
 * Phát âm từ qua Web Speech API.
 * @param text — chuỗi cần đọc
 * @param lang — BCP-47, mặc định 'en-US'
 * @throws nếu trình duyệt không hỗ trợ `speechSynthesis`
 */
export function speak(text: string, lang = 'en-US'): void {
  if (!('speechSynthesis' in window)) throw new Error('Speech synthesis not supported');
  const u = new SpeechSynthesisUtterance(text);
  u.lang = lang;
  window.speechSynthesis.speak(u);
}
```

```tsx
// Component nội bộ — không JSDoc
export function WordCard({ word, ipa }: WordCardProps) { /* ... */ }
```

---

## Rule 5 — CẤM code commented-out trong file merge vào main

Lý do: git history giữ code cũ. Code comment-out là rác và gây confusion về intent.

### ❌ Sai

```ts
const result = await api<Folder[]>('/folders');
// const oldResult = await fetch('/folders').then(r => r.json());
// console.log('debug', oldResult);
```

### ✅ Đúng

```ts
const result = await api<Folder[]>('/folders');
```

---

## Rule 6 — CẤM comment kiểu changelog trong file

Lý do: changelog thuộc commit message / CHANGELOG.md, không thuộc source.

### ❌ Sai

```ts
// 2026-05-09: thêm field ipa
// 2026-05-10: đổi type Folder
// 2026-05-12: fix bug import CSV
export interface Vocabulary { /* ... */ }
```

### ✅ Đúng

```ts
export interface Vocabulary { /* ... */ }
// Lịch sử thay đổi xem `git log src/types/api.ts`.
```

---

## Rule 7 — Magic number/string PHẢI có comment hoặc đặt tên hằng số rõ ràng

Lý do: `10` đứng 1 mình không nói gì; `MAX_AI_WORDS` nói tất cả.

### ❌ Sai

```ts
if (selected.length > 10) throw new Error('Quá nhiều');
setTimeout(retry, 3000);
```

### ✅ Đúng

```ts
const MAX_AI_WORDS = 10;       // Giới hạn input cho /ai/generate-text (theo spec BE).
const RETRY_DELAY_MS = 3000;

if (selected.length > MAX_AI_WORDS) throw new Error('Quá nhiều từ');
setTimeout(retry, RETRY_DELAY_MS);
```

---

## Rule 8 — File header KHÔNG bắt buộc; nếu có chỉ ghi mục đích, CẤM `@author`/`@created`/`@modified`

Lý do: tác giả/ngày đã có git blame; metadata trùng lặp là rot.

### ❌ Sai

```ts
/**
 * @file word-card.tsx
 * @author Quyen
 * @created 2026-05-09
 * @modified 2026-05-12
 */
```

### ✅ Đúng

```ts
// Card hiển thị 1 từ vựng + IPA + nút phát âm. Dùng trong flashcard và list.
```

---

## Checklist tự kiểm

- [ ] Comment trả lời "tại sao", không lặp lại "cái gì".
- [ ] Workaround / business rule / perf hack đều có comment.
- [ ] TODO/FIXME có owner + ngày.
- [ ] JSDoc chỉ ở `src/lib/`, `src/hooks/` export public.
- [ ] Không còn code commented-out.
- [ ] Không có comment changelog trong file.
- [ ] Magic number đặt tên hoặc có comment.
- [ ] Không có file header chứa @author/@date.
