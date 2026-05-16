# Naming Conventions — FE

> **Role**: Frontend
> **Nguồn tham chiếu**: `.claude/FE-architecture.md` (Next.js 15 App Router + TypeScript + TanStack Query + Zustand)

**Phạm vi áp dụng**: toàn bộ source code trong `src/` của repo `english-learning-web`.
**Mức độ bắt buộc**: **MUST — vi phạm = block PR**.

---

## Rule 1 — File & folder PHẢI dùng kebab-case

Lý do: Next.js App Router dùng tên file/folder làm route path. kebab-case tránh conflict case-sensitive giữa macOS/Linux và đồng nhất với URL.

### ❌ Sai

```
src/components/WordCard.tsx
src/hooks/useFolders.ts
src/app/Folders/[Id]/page.tsx
```

### ✅ Đúng

```
src/components/vocabulary/word-card.tsx
src/hooks/use-folders.ts
src/app/folders/[id]/page.tsx
```

### Ghi chú
Route segment động vẫn là `[id]`, `[slug]` — đây là cú pháp Next.js, không phải kebab-case.

---

## Rule 2 — React component PHẢI là PascalCase, 1 component / 1 file

Lý do: dễ phân biệt JSX tag với HTML tag; tránh trộn nhiều component trong 1 file.

### ❌ Sai

```tsx
// src/components/vocabulary/word-card.tsx
export function wordCard() { /* ... */ }
export function WordList() { /* ... */ }   // KHÔNG được nhét chung file
```

### ✅ Đúng

```tsx
// src/components/vocabulary/word-card.tsx
export function WordCard() { /* ... */ }
```

```tsx
// src/components/vocabulary/word-list.tsx
export function WordList() { /* ... */ }
```

---

## Rule 3 — Hook PHẢI có prefix `use`, camelCase; file là kebab-case khớp tên hook

Lý do: React enforce `use*` cho rules-of-hooks lint; kebab-case file tránh khác biệt OS.

### ❌ Sai

```ts
// src/hooks/folders.ts
export function getFolders() { /* gọi useQuery bên trong */ }
```

### ✅ Đúng

```ts
// src/hooks/use-folders.ts
import { useQuery } from '@tanstack/react-query';
import { api } from '@/lib/api-client';
import type { Folder } from '@/types/api';

export function useFolders() {
  return useQuery({
    queryKey: ['folders'],
    queryFn: () => api<Folder[]>('/folders'),
  });
}
```

---

## Rule 4 — Zustand store: file `*-store.ts`, hook `useXxxStore`

Lý do: nhận diện store ngay ở file system và import site.

### ❌ Sai

```ts
// src/stores/test-session.ts
import { create } from 'zustand';
export const testSession = create(/* ... */);
```

### ✅ Đúng

```ts
// src/stores/test-session-store.ts
import { create } from 'zustand';

type TestSessionState = {
  currentIndex: number;
  answers: Record<string, string>;
  next: () => void;
};

export const useTestSessionStore = create<TestSessionState>((set) => ({
  currentIndex: 0,
  answers: {},
  next: () => set((s) => ({ currentIndex: s.currentIndex + 1 })),
}));
```

---

## Rule 5 — TanStack Query key PHẢI là array có thứ tự ổn định, KHÔNG dùng template string

Lý do: array key cho phép invalidate theo prefix; template string vỡ partial match.

### ❌ Sai

```ts
useQuery({ queryKey: [`folder-${folderId}-vocabulary`], queryFn: ... });
```

### ✅ Đúng

```ts
useQuery({
  queryKey: ['folders', folderId, 'vocabulary'],
  queryFn: () => api<Vocabulary[]>(`/folders/${folderId}/vocabulary`),
});

// Invalidate theo prefix:
queryClient.invalidateQueries({ queryKey: ['folders', folderId] });
```

---

## Rule 6 — Type/Interface PascalCase, KHÔNG prefix `I`/`T`

Lý do: TS hiện đại + shadcn + Next.js đều không prefix.

### ❌ Sai

```ts
interface IFolder { id: string; name: string }
type TWordCardProps = { word: string };
```

### ✅ Đúng

```ts
// src/types/api.ts
export interface Folder { id: string; name: string; }

// src/components/vocabulary/word-card.tsx
type WordCardProps = { word: string; ipa?: string };
```

---

## Rule 7 — Boolean prefix `is`/`has`/`can`; event handler prefix `handle` (trong component) hoặc `on` (trong props)

Lý do: đọc code biết ngay là cờ hay action; phân biệt người gọi vs định nghĩa.

### ❌ Sai

```tsx
function WordCard({ flipped, click }: { flipped: boolean; click: () => void }) {
  const loading = useSomething();
  return <button onClick={click}>{loading ? '...' : 'Flip'}</button>;
}
```

### ✅ Đúng

```tsx
type WordCardProps = { isFlipped: boolean; onFlip: () => void };

export function WordCard({ isFlipped, onFlip }: WordCardProps) {
  const isLoading = useSomething();
  const handleClick = () => onFlip();
  return <button onClick={handleClick}>{isLoading ? '...' : 'Flip'}</button>;
}
```

---

## Rule 8 — Constant `SCREAMING_SNAKE_CASE`; enum-like dùng `as const` object, KHÔNG `enum`

Lý do: phân biệt giá trị bất biến với biến thường; `as const` an toàn hơn `enum` trong TS hiện đại (tree-shake tốt, không gen runtime code thừa).

### ❌ Sai

```ts
export const maxWords = 10;
export enum TestType { Flashcard, Quiz }
```

### ✅ Đúng

```ts
export const MAX_AI_WORDS = 10;

export const TEST_TYPE = {
  FLASHCARD: 'FLASHCARD',
  QUIZ: 'QUIZ',
  AI_TEXT: 'AI_TEXT',
} as const;

export type TestType = (typeof TEST_TYPE)[keyof typeof TEST_TYPE];
```

---

## Rule 9 — Storage key / event name dùng `kebab-case` hoặc `dot.notation`, có namespace

Lý do: tránh đụng key giữa các feature; namespace giúp clear/migration theo phạm vi.

### ❌ Sai

```ts
localStorage.setItem('theme', 'dark');
logger.info('login');
```

### ✅ Đúng

```ts
localStorage.setItem('preference:theme', 'dark');
logger.info('auth.login.success', { userId });
```

---

## Checklist tự kiểm

- [ ] File/folder kebab-case; route động `[xxx]` đúng cú pháp Next.js.
- [ ] Component PascalCase, 1 component / file, named export.
- [ ] Hook prefix `use`, file `use-*.ts`.
- [ ] Store file `*-store.ts`, hook `useXxxStore`.
- [ ] Query key là array, bắt đầu bằng resource name.
- [ ] Type/Interface không có prefix `I`/`T`.
- [ ] Boolean `is/has/can`; handler `handle*` / `on*`.
- [ ] Constant SCREAMING_SNAKE_CASE; enum-like = `as const`.
- [ ] Storage key & event name có namespace.
