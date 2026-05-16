# API Integration — FE

> **Role**: Frontend
> **Nguồn tham chiếu**: `.claude/FE-architecture.md` (`fetch` wrapper `api()` + JWT httpOnly cookie + TanStack Query)

**Phạm vi áp dụng**: mọi request HTTP đi từ FE đến BE NestJS.
**Mức độ bắt buộc**: **MUST — vi phạm = block PR**.

---

## Rule 1 — Mọi request HTTP PHẢI đi qua `api()` trong `@/lib/api-client`; CẤM `fetch` trực tiếp trong component/hook

Lý do: base URL, `credentials: 'include'`, header, error handling phải tập trung. Gọi `fetch` rời sẽ quên cookie → 401 ngẫu nhiên.

### ❌ Sai

```ts
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
```

### ✅ Đúng

```ts
// src/lib/api-client.ts
const BASE = process.env.NEXT_PUBLIC_API_BASE_URL!;

export class ApiError extends Error {
  constructor(public status: number, public body: unknown, message?: string) {
    super(message ?? `API ${status}`);
  }
}

export async function api<T>(path: string, init?: RequestInit): Promise<T> {
  const res = await fetch(`${BASE}${path}`, {
    ...init,
    credentials: 'include',
    headers: { 'Content-Type': 'application/json', ...init?.headers },
  });
  if (!res.ok) {
    const body = await res.json().catch(() => null);
    throw new ApiError(res.status, body);
  }
  return res.json() as Promise<T>;
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

## Rule 2 — `credentials: 'include'` BẮT BUỘC; CẤM tự gắn `Authorization: Bearer` ở FE

Lý do: token trong httpOnly cookie do BE set; FE không đọc được, không cần gắn header.

### ❌ Sai

```ts
const token = localStorage.getItem('token');
fetch(url, { headers: { Authorization: `Bearer ${token}` } });   // ❌ không có token ở FE
```

### ✅ Đúng

```ts
// Đã xử lý trong api(). Component không làm gì.
const folders = await api<Folder[]>('/folders');
```

---

## Rule 3 — Response/Request type BẮT BUỘC khai báo trong `src/types/api.ts`; CẤM `any`

Lý do: type là contract; thiếu type rò rỉ `any` xuống component và phá refactor.

### ❌ Sai

```ts
const data = await api('/folders');                  // any
const stats: any = await api('/stats/dashboard');
```

### ✅ Đúng

```ts
// src/types/api.ts (copy tay từ DTO của NestJS BE)
export interface Folder {
  id: string;
  name: string;
  description: string | null;
  createdAt: string;
}

export interface DashboardStats {
  totalWords: number;
  totalSessions: number;
  accuracy: number;
  weeklyChart: { date: string; correct: number; total: number }[];
}
```

```ts
const folders = await api<Folder[]>('/folders');
const stats = await api<DashboardStats>('/stats/dashboard');
```

---

## Rule 4 — Request body PHẢI validate bằng zod trước khi gửi

Lý do: BE đã validate, nhưng client validate sớm tiết kiệm round-trip và bắt lỗi sát người dùng.

### ❌ Sai

```ts
const onSubmit = (raw: unknown) => {
  api('/folders', { method: 'POST', body: JSON.stringify(raw) });   // ❌ gửi bừa
};
```

### ✅ Đúng

```ts
import { z } from 'zod';

const createFolderSchema = z.object({
  name: z.string().min(1).max(100),
  description: z.string().max(500).optional(),
});
type CreateFolderInput = z.infer<typeof createFolderSchema>;

export function useCreateFolder() {
  const qc = useQueryClient();
  return useMutation({
    mutationFn: (input: CreateFolderInput) => {
      const parsed = createFolderSchema.parse(input);
      return api<Folder>('/folders', { method: 'POST', body: JSON.stringify(parsed) });
    },
    onSuccess: () => qc.invalidateQueries({ queryKey: ['folders'] }),
  });
}
```

---

## Rule 5 — CẤM `try/catch` nuốt lỗi rồi return `null`/`undefined` trong queryFn/mutationFn

Lý do: TanStack Query phụ thuộc throw để xác định state error. Nuốt lỗi = mất loading skeleton, mất retry, mất toast.

### ❌ Sai

```ts
queryFn: async () => {
  try {
    return await api<Folder[]>('/folders');
  } catch (e) {
    console.error(e);
    return [];                            // ❌ Query thấy "thành công"
  }
},
```

### ✅ Đúng

```ts
queryFn: () => api<Folder[]>('/folders'),  // throw → Query bắt → component dùng `error`
```

---

## Rule 6 — Hook data đặt tên theo pattern `useXxx` (query) / `useCreateXxx`, `useUpdateXxx`, `useDeleteXxx` (mutation)

Lý do: đọc tên là biết loại; tránh hook mơ hồ.

### ❌ Sai

```ts
export function useFolderApi() { /* trộn query và mutation */ }
export function useNewFolder() { /* unclear: hook khởi tạo? mutation? */ }
```

### ✅ Đúng

```ts
export function useFolders() { /* query: list */ }
export function useFolder(id: string) { /* query: detail */ }
export function useCreateFolder() { /* mutation */ }
export function useUpdateFolder() { /* mutation */ }
export function useDeleteFolder() { /* mutation */ }
```

---

## Rule 7 — Endpoint path co-locate trong hook tương ứng; CẤM lặp cùng path ở 2 nơi

Lý do: endpoint chỉ xuất hiện 1 nơi đủ tập trung khi project nhỏ; KHÔNG được lặp.

### ❌ Sai

```ts
// hook A
fetch(`${BASE}/folders/${id}/vocabulary`);
// component B trực tiếp
fetch(`${BASE}/folders/${id}/vocabulary`);
```

### ✅ Đúng

```ts
// src/hooks/use-vocabulary.ts — chỉ ở đây path tồn tại
export function useVocabulary(folderId: string) {
  return useQuery({
    queryKey: ['folders', folderId, 'vocabulary'],
    queryFn: () => api<Vocabulary[]>(`/folders/${folderId}/vocabulary`),
    enabled: Boolean(folderId),
  });
}
```

Component chỉ gọi `useVocabulary(folderId)`.

---

## Rule 8 — CẤM `console.log` payload/response trong code merge vào main

Lý do: rò rỉ thông tin nhạy cảm + ô nhiễm console production.

### ❌ Sai

```ts
const data = await api<Folder[]>('/folders');
console.log('folders:', data);
```

### ✅ Đúng

```ts
const data = await api<Folder[]>('/folders');
// Debug: dùng React Query Devtools, không log raw.
```

---

## Rule 9 — Query có dependency PHẢI dùng `enabled` để tránh fire khi tham số chưa sẵn sàng

Lý do: gọi `/folders/undefined/vocabulary` lãng phí và spam log lỗi 4xx.

### ❌ Sai

```ts
const { data } = useQuery({
  queryKey: ['folders', folderId, 'vocabulary'],
  queryFn: () => api<Vocabulary[]>(`/folders/${folderId}/vocabulary`),
});
```

### ✅ Đúng

```ts
const { data } = useQuery({
  queryKey: ['folders', folderId, 'vocabulary'],
  queryFn: () => api<Vocabulary[]>(`/folders/${folderId}/vocabulary`),
  enabled: Boolean(folderId),
});
```

---

## Checklist tự kiểm

- [ ] Mọi request đi qua `api()`, không `fetch`/`axios` trực tiếp.
- [ ] `credentials: 'include'`, không tự gắn `Authorization`.
- [ ] Type request/response khai báo trong `src/types/api.ts`.
- [ ] Form/CSV input đã validate zod trước khi gửi.
- [ ] queryFn/mutationFn throw error, không nuốt.
- [ ] Hook đặt tên đúng pattern.
- [ ] Không lặp endpoint path ở 2 nơi.
- [ ] Không `console.log` payload.
- [ ] Query phụ thuộc id có `enabled: Boolean(id)`.
