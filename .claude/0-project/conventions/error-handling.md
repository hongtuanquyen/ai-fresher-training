# Error Handling — FE

> **Role**: Frontend
> **Nguồn tham chiếu**: `.claude/FE-architecture.md` (Next.js App Router + TanStack Query + RHF)

**Phạm vi áp dụng**: mọi đường lỗi từ render, fetch, mutation, form, uncaught.
**Mức độ bắt buộc**: **MUST — vi phạm = block PR**.

> Toast lib: architecture chưa chốt cụ thể (gợi ý `sonner` hoặc shadcn `toast`) — TBD theo lib chốt sau. Trong ví dụ dưới dùng `sonner`.

---

## Rule 1 — Mỗi route segment quan trọng PHẢI có `error.tsx` của Next.js App Router

Lý do: lỗi render trong segment không có `error.tsx` sẽ rơi lên root → cả app trắng trang.

### ❌ Sai

```
src/app/folders/[id]/page.tsx     ← throw inside page
(không có error.tsx ở segment)
```

### ✅ Đúng

```tsx
// src/app/folders/[id]/error.tsx
'use client';

import { getUserMessage } from '@/lib/error-message';

export default function FolderError({ error, reset }: { error: Error; reset: () => void }) {
  return (
    <div className="space-y-3 p-6">
      <h2 className="text-lg font-semibold">Không tải được folder</h2>
      <p className="text-muted-foreground">{getUserMessage(error)}</p>
      <button onClick={reset} className="underline">Thử lại</button>
    </div>
  );
}
```

---

## Rule 2 — Lỗi từ TanStack Query PHẢI render UI fallback; CẤM silent fail

Lý do: render `data` rỗng khi thực sự lỗi → user tưởng "chưa có dữ liệu".

### ❌ Sai

```tsx
const { data = [] } = useFolders();        // ❌ error vô hình
return <ul>{data.map(...)}</ul>;
```

### ✅ Đúng

```tsx
const { data, isLoading, isError, error, refetch } = useFolders();

if (isLoading) return <Skeleton />;
if (isError) {
  return (
    <div role="alert" className="space-y-2">
      <p>{getUserMessage(error)}</p>
      <button onClick={() => refetch()}>Thử lại</button>
    </div>
  );
}
return <ul>{data!.map((f) => <li key={f.id}>{f.name}</li>)}</ul>;
```

---

## Rule 3 — Form error PHẢI hiển thị từ `formState.errors`; CẤM `alert()`

Lý do: `alert()` block UI, không accessible, không khớp design system.

### ❌ Sai

```tsx
const onSubmit = (v: FormValues) => {
  if (!v.word) return alert('Bắt buộc');
};
```

### ✅ Đúng

```tsx
const { register, handleSubmit, formState: { errors } } = useForm<FormValues>({
  resolver: zodResolver(schema),
});

return (
  <form onSubmit={handleSubmit(onSubmit)}>
    <input {...register('word')} aria-invalid={Boolean(errors.word)} />
    {errors.word && <p className="text-sm text-destructive">{errors.word.message}</p>}
  </form>
);
```

---

## Rule 4 — Lỗi mutation PHẢI hiện toast; CẤM nuốt im lặng

Lý do: user cần biết "save" thất bại; nuốt lỗi → user nghĩ thành công và mất data.

### ❌ Sai

```ts
const m = useMutation({ mutationFn: saveWord });
// component không xử lý m.error
```

### ✅ Đúng

```tsx
'use client';
import { toast } from 'sonner';
import { getUserMessage } from '@/lib/error-message';

const createFolder = useMutation({
  mutationFn: (input: CreateFolderInput) =>
    api<Folder>('/folders', { method: 'POST', body: JSON.stringify(input) }),
  onSuccess: () => {
    toast.success('Đã tạo folder');
    qc.invalidateQueries({ queryKey: ['folders'] });
  },
  onError: (err) => toast.error(getUserMessage(err)),
});
```

---

## Rule 5 — CẤM `catch {}` rỗng và CẤM phơi raw message của BE cho user

Lý do: catch rỗng giấu bug; raw BE message lộ stack trace + thuật ngữ kỹ thuật.

### ❌ Sai

```ts
try { await api(...); } catch {}                       // ❌ giấu bug
toast.error(error.message);                            // ❌ "Cannot read properties of undefined..."
```

### ✅ Đúng

```ts
// src/lib/error-message.ts
import { ApiError } from '@/lib/api-client';

export function getUserMessage(error: unknown): string {
  if (error instanceof ApiError) {
    if (error.status === 401) return 'Phiên đăng nhập đã hết hạn.';
    if (error.status === 403) return 'Bạn không có quyền thực hiện thao tác này.';
    if (error.status >= 500) return 'Hệ thống đang gặp sự cố, vui lòng thử lại.';
    if (error.status >= 400) {
      const body = error.body as { message?: string } | null;
      return body?.message ?? 'Yêu cầu không hợp lệ.';
    }
  }
  return 'Có lỗi xảy ra, vui lòng thử lại.';
}
```

```ts
toast.error(getUserMessage(err));
```

---

## Rule 6 — Lỗi 401 PHẢI redirect `/login` và clear query cache

Lý do: cookie hết hạn; tiếp tục refetch sẽ spam 401.

### ❌ Sai

```ts
// queryClient mặc định + không xử lý 401 → infinite retry
```

### ✅ Đúng

```tsx
// src/app/providers.tsx
'use client';
import { useState } from 'react';
import { QueryClient, QueryClientProvider, MutationCache, QueryCache } from '@tanstack/react-query';
import { ApiError } from '@/lib/api-client';
import { useRouter } from 'next/navigation';

export function Providers({ children }: { children: React.ReactNode }) {
  const router = useRouter();
  const [client] = useState(() => {
    const onAuthError = (err: unknown) => {
      if (err instanceof ApiError && err.status === 401) {
        client.clear();
        router.replace('/login');
      }
    };
    const client: QueryClient = new QueryClient({
      defaultOptions: {
        queries: { retry: (count, err) => !(err instanceof ApiError && err.status === 401) && count < 2 },
      },
      queryCache: new QueryCache({ onError: onAuthError }),
      mutationCache: new MutationCache({ onError: onAuthError }),
    });
    return client;
  });
  return <QueryClientProvider client={client}>{children}</QueryClientProvider>;
}
```

---

## Rule 7 — Async event handler PHẢI bọc try/catch hoặc dựa vào hook mutation; CẤM để promise lỡ

Lý do: unhandled promise rejection không hiện UI và không log.

### ❌ Sai

```tsx
<button onClick={() => api('/folders/x', { method: 'DELETE' })}>Xóa</button>
```

### ✅ Đúng

```tsx
const remove = useDeleteFolder();
<button onClick={() => remove.mutate(id)} disabled={remove.isPending}>Xóa</button>
```

---

## Rule 8 — Mọi lỗi không-mong-đợi PHẢI log qua `@/lib/logger`; CẤM `console.error` rải rác

Lý do: logger là chỗ duy nhất biết phải gửi đi đâu (Sentry sau, tracker, hay no-op).

### ❌ Sai

```ts
catch (e) { console.error('save failed', e); }
```

### ✅ Đúng

```ts
import { logger } from '@/lib/logger';

catch (e) {
  logger.error('vocabulary.save.failed', { error: e });
  toast.error(getUserMessage(e));
}
```

---

## Checklist tự kiểm

- [ ] Route segment quan trọng có `error.tsx`.
- [ ] Hook query/mutation đều render UI fallback khi `isError`.
- [ ] Form lỗi qua `formState.errors`, không `alert()`.
- [ ] Mutation có `onError` show toast.
- [ ] Không có `catch {}` rỗng.
- [ ] User không thấy raw error BE (đi qua `getUserMessage`).
- [ ] 401 → clear cache + redirect `/login`.
- [ ] Không có async event handler để promise lỡ.
- [ ] Lỗi unexpected log qua `logger`, không `console.error`.
