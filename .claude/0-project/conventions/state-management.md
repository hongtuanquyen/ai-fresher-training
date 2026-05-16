# State Management — FE

> **Role**: Frontend
> **Nguồn tham chiếu**: `.claude/FE-architecture.md` (TanStack Query + Zustand + react-hook-form + zod)

**Phạm vi áp dụng**: mọi state trong `src/`.
**Mức độ bắt buộc**: **MUST — vi phạm = block PR**.

---

## Rule 1 — Phân loại state theo bảng dưới; CẤM dùng sai loại

| Loại state | Ví dụ | Công cụ BẮT BUỘC |
|---|---|---|
| Server state | folder list, vocabulary, stats | **TanStack Query** (`@tanstack/react-query`) |
| URL state | filter folder, tab active, page index | `useSearchParams` / `useRouter` (Next.js) |
| Client state cục bộ | dialog open, current flashcard index | `useState` / `useReducer` |
| Client state global | test session đang chạy, user preference | **Zustand** trong `src/stores/` |
| Form state | form thêm từ, login | **react-hook-form + zod resolver** |

Lý do: chọn sai công cụ → state stale, prop drilling, hoặc bundle thừa.

---

## Rule 2 — Server state CHỈ dùng TanStack Query; CẤM `useEffect` + `useState` để fetch

Lý do: `useEffect` fetch không cache, không retry, không dedupe — phải tự viết lại tất cả.

### ❌ Sai

```tsx
'use client';
export function FolderList() {
  const [folders, setFolders] = useState<Folder[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch('/api/v1/folders').then((r) => r.json()).then((d) => {
      setFolders(d);
      setLoading(false);
    });
  }, []);

  if (loading) return <p>...</p>;
  return <ul>{folders.map((f) => <li key={f.id}>{f.name}</li>)}</ul>;
}
```

### ✅ Đúng

```tsx
'use client';
import { useFolders } from '@/hooks/use-folders';

export function FolderList() {
  const { data, isLoading } = useFolders();
  if (isLoading) return <p>...</p>;
  return <ul>{data?.map((f) => <li key={f.id}>{f.name}</li>)}</ul>;
}
```

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

## Rule 3 — CẤM duplicate server data sang Zustand

Lý do: Query là source-of-truth; copy sang store gây stale + sync hell.

### ❌ Sai

```ts
// src/stores/folders-store.ts
export const useFoldersStore = create<{ folders: Folder[]; setFolders: (f: Folder[]) => void }>(
  (set) => ({ folders: [], setFolders: (folders) => set({ folders }) }),
);
```

```tsx
const { data } = useFolders();
useEffect(() => { if (data) setFolders(data); }, [data]);   // ❌ duplicate
```

### ✅ Đúng

```tsx
const { data: folders = [] } = useFolders();    // dùng trực tiếp từ Query cache
```

Zustand chỉ giữ **client state** thuần:

```ts
// src/stores/test-session-store.ts
import { create } from 'zustand';

type State = {
  currentIndex: number;
  answers: Record<string, string>;
  next: () => void;
  setAnswer: (vocabularyId: string, answer: string) => void;
  reset: () => void;
};

export const useTestSessionStore = create<State>((set) => ({
  currentIndex: 0,
  answers: {},
  next: () => set((s) => ({ currentIndex: s.currentIndex + 1 })),
  setAnswer: (id, answer) => set((s) => ({ answers: { ...s.answers, [id]: answer } })),
  reset: () => set({ currentIndex: 0, answers: {} }),
}));
```

---

## Rule 4 — Mutation PHẢI invalidate query liên quan trong `onSuccess`

Lý do: không invalidate → UI hiện data cũ sau khi tạo/sửa/xóa.

### ❌ Sai

```ts
const createFolder = useMutation({
  mutationFn: (input: CreateFolderInput) =>
    api<Folder>('/folders', { method: 'POST', body: JSON.stringify(input) }),
});
// không invalidate → list folder không refresh
```

### ✅ Đúng

```ts
import { useMutation, useQueryClient } from '@tanstack/react-query';

export function useCreateFolder() {
  const qc = useQueryClient();
  return useMutation({
    mutationFn: (input: CreateFolderInput) =>
      api<Folder>('/folders', { method: 'POST', body: JSON.stringify(input) }),
    onSuccess: () => {
      qc.invalidateQueries({ queryKey: ['folders'] });
    },
  });
}
```

---

## Rule 5 — Form state PHẢI dùng react-hook-form + zod resolver; CẤM `useState` rời rạc cho field

Lý do: nhiều `useState` re-render mỗi keystroke và phải tự viết validation.

### ❌ Sai

```tsx
'use client';
export function WordForm() {
  const [word, setWord] = useState('');
  const [meaning, setMeaning] = useState('');
  const [error, setError] = useState('');

  const submit = () => {
    if (!word) return setError('Bắt buộc');
    // ... call API
  };
  return (
    <form onSubmit={submit}>
      <input value={word} onChange={(e) => setWord(e.target.value)} />
      <input value={meaning} onChange={(e) => setMeaning(e.target.value)} />
      {error && <p>{error}</p>}
    </form>
  );
}
```

### ✅ Đúng

```tsx
'use client';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({
  word: z.string().min(1, 'Bắt buộc'),
  meaning: z.string().min(1, 'Bắt buộc'),
  ipa: z.string().optional(),
});
type FormValues = z.infer<typeof schema>;

export function WordForm({ onSubmit }: { onSubmit: (v: FormValues) => void }) {
  const { register, handleSubmit, formState: { errors } } = useForm<FormValues>({
    resolver: zodResolver(schema),
    defaultValues: { word: '', meaning: '', ipa: '' },
  });

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('word')} />
      {errors.word && <p>{errors.word.message}</p>}
      <input {...register('meaning')} />
      {errors.meaning && <p>{errors.meaning.message}</p>}
    </form>
  );
}
```

---

## Rule 6 — Query key PHẢI cấu trúc `[resource, ...params]`; tách registry nếu dùng nhiều nơi

Lý do: cấu trúc ổn định giúp `invalidateQueries` theo prefix; tách hằng số tránh typo.

### ❌ Sai

```ts
useQuery({ queryKey: ['get-folder-' + id], ... });
useQuery({ queryKey: [id, 'folder'], ... });
```

### ✅ Đúng

```ts
// src/lib/query-keys.ts
export const queryKeys = {
  folders: ['folders'] as const,
  folder: (id: string) => ['folders', id] as const,
  folderVocabulary: (id: string) => ['folders', id, 'vocabulary'] as const,
  stats: ['stats', 'dashboard'] as const,
};
```

```ts
useQuery({ queryKey: queryKeys.folder(id), queryFn: () => api<Folder>(`/folders/${id}`) });
```

---

## Rule 7 — URL state cho filter/pagination/tab; CẤM `useState` cho thứ cần shareable

Lý do: filter trong `useState` mất khi reload, không share URL, không deep-link.

### ❌ Sai

```tsx
'use client';
const [tab, setTab] = useState<'manual' | 'csv'>('manual');
```

### ✅ Đúng

```tsx
'use client';
import { useRouter, useSearchParams, usePathname } from 'next/navigation';

export function ImportTabs() {
  const router = useRouter();
  const pathname = usePathname();
  const params = useSearchParams();
  const tab = (params.get('tab') ?? 'manual') as 'manual' | 'csv';

  const setTab = (next: 'manual' | 'csv') => {
    const sp = new URLSearchParams(params);
    sp.set('tab', next);
    router.replace(`${pathname}?${sp.toString()}`);
  };

  return <Tabs value={tab} onValueChange={(v) => setTab(v as 'manual' | 'csv')} />;
}
```

---

## Rule 8 — Persist state: CHỈ persist cái thực sự cần; CẤM persist token vào localStorage

Lý do: token trong localStorage là target XSS. Auth dùng httpOnly cookie do BE set — KHÔNG động vào.

### ❌ Sai

```ts
localStorage.setItem('access_token', token);   // ❌ XSS leak
```

### ✅ Đúng

```ts
// Token nằm trong httpOnly cookie do BE set khi /auth/login. FE không touch.
// Chỉ persist UI preference:
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

export const usePreferenceStore = create(
  persist<{ theme: 'light' | 'dark'; setTheme: (t: 'light' | 'dark') => void }>(
    (set) => ({ theme: 'light', setTheme: (theme) => set({ theme }) }),
    { name: 'preference', version: 1 },
  ),
);
```

---

## Checklist tự kiểm

- [ ] Đúng loại state → đúng công cụ (bảng Rule 1).
- [ ] Không `useEffect` fetch + `useState` cho server data.
- [ ] Không duplicate server data sang Zustand.
- [ ] Mọi mutation đều invalidate query liên quan.
- [ ] Form dùng RHF + zod, không quản lý field bằng `useState`.
- [ ] Query key qua `queryKeys` registry hoặc `[resource, ...params]`.
- [ ] Filter/tab/pagination đi qua URL state.
- [ ] Không persist token / dữ liệu nhạy cảm vào localStorage.
