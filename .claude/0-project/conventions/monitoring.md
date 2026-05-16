# Monitoring — FE

> **Role**: Frontend
> **Nguồn tham chiếu**: `.claude/FE-architecture.md` (Next.js 15 + React 19. Error tracker / analytics: **chưa chốt** — TBD theo lib chốt sau)

**Phạm vi áp dụng**: mọi log, event, performance metric phát ra từ FE.
**Mức độ bắt buộc**: **MUST — vi phạm = block PR**.

> **Ghi chú stack**: Architecture CHƯA chốt Sentry/Datadog/PostHog/...
> Convention dưới định nghĩa **wrapper `@/lib/logger`** với API tối thiểu để
> swap tracker thật sau mà không phải đổi call-site.

---

## Rule 1 — CẤM `console.log` trong code merge vào main; PHẢI dùng `logger` wrapper

Lý do: `console.log` rò rỉ thông tin lên DevTools production và không thể bật/tắt theo môi trường.

### ❌ Sai

```ts
console.log('user logged in', user);
console.log('folders:', folders);
```

### ✅ Đúng

```ts
// src/lib/logger.ts
type Level = 'debug' | 'info' | 'warn' | 'error';
type LogContext = Record<string, unknown>;

const isDev = process.env.NODE_ENV !== 'production';

function emit(level: Level, event: string, ctx?: LogContext) {
  if (level === 'debug' && !isDev) return;
  // TBD: thay bằng SDK tracker thật khi chốt (Sentry.captureMessage / posthog.capture).
  if (isDev) {
    // eslint-disable-next-line no-console
    console[level === 'debug' ? 'log' : level](`[${level}] ${event}`, ctx ?? {});
  }
}

export const logger = {
  debug: (event: string, ctx?: LogContext) => emit('debug', event, ctx),
  info:  (event: string, ctx?: LogContext) => emit('info',  event, ctx),
  warn:  (event: string, ctx?: LogContext) => emit('warn',  event, ctx),
  error: (event: string, ctx?: LogContext) => emit('error', event, ctx),
};
```

```ts
import { logger } from '@/lib/logger';

logger.info('auth.login.success', { userId: user.id });
logger.error('vocabulary.bulk_import.failed', { folderId, count, error });
```

---

## Rule 2 — Phân cấp log đúng nghĩa: `debug` cục bộ, `info` cột mốc, `warn` bất thường có thể bỏ qua, `error` lỗi cần chú ý

Lý do: dùng sai cấp = noise hoặc bỏ sót sự cố.

### ❌ Sai

```ts
logger.error('user clicked button');         // ❌ click != error
logger.debug('payment failed', { ... });     // ❌ payment fail là error
```

### ✅ Đúng

```ts
logger.info('flashcard.session.start', { folderId, wordCount });
logger.warn('speech.api.unsupported', { userAgent: navigator.userAgent });
logger.error('test_session.submit.failed', { error });
```

---

## Rule 3 — Tên event PHẢI theo pattern `feature.action.result` (snake_case, dot)

Lý do: tên ổn định cho phép aggregate/filter trên tracker; tự do gây schema explosion.

### ❌ Sai

```ts
logger.info('User Created Folder');
logger.info('createFolderSuccess');
logger.info('FE-folder-create-ok');
```

### ✅ Đúng

```ts
logger.info('folder.create.success', { folderId });
logger.info('folder.create.failed',  { reason: 'duplicate_name' });
logger.info('quiz.session.complete', { folderId, score, total });
```

---

## Rule 4 — CẤM log dữ liệu nhạy cảm: password, token, raw cookie, PII của user khác, payload thanh toán

Lý do: log = audit trail; lộ secret = sự cố security.

### ❌ Sai

```ts
logger.info('auth.login.attempt', { email, password });          // ❌
logger.error('api.failed', { headers: request.headers });        // ❌ chứa cookie
```

### ✅ Đúng

```ts
logger.info('auth.login.attempt', { email });                    // ✅ không log password
logger.error('api.failed', { url, status });                     // ✅ không log header raw
```

### Ghi chú
Khi cần log object có thể chứa secret, dùng helper `redact()`:

```ts
// src/lib/logger.ts
const SENSITIVE_KEYS = new Set(['password', 'token', 'authorization', 'cookie']);

export function redact<T>(obj: T): T {
  if (!obj || typeof obj !== 'object') return obj;
  return Object.fromEntries(
    Object.entries(obj as Record<string, unknown>).map(([k, v]) =>
      [k, SENSITIVE_KEYS.has(k.toLowerCase()) ? '[REDACTED]' : v],
    ),
  ) as T;
}
```

---

## Rule 5 — Component KHÔNG gọi tracker SDK trực tiếp; PHẢI đi qua `logger`

Lý do: 1 chỗ swap SDK; component không phụ thuộc vendor.

### ❌ Sai

```tsx
'use client';
import * as Sentry from '@sentry/nextjs';

const onSubmit = async () => {
  try { await submit(); }
  catch (e) { Sentry.captureException(e); }
};
```

### ✅ Đúng

```tsx
'use client';
import { logger } from '@/lib/logger';

const onSubmit = async () => {
  try { await submit(); }
  catch (e) { logger.error('test_session.submit.failed', { error: e }); throw e; }
};
```

---

## Rule 6 — Web Vitals (LCP, INP, CLS) PHẢI report qua `logger`; ngưỡng cảnh báo theo Google "good"

Lý do: regress perf âm thầm là rủi ro lớn nhất với app FE.

### ❌ Sai

```ts
// Không report Web Vitals → không phát hiện regression
```

### ✅ Đúng

```tsx
// src/app/web-vitals.tsx
'use client';
import { useReportWebVitals } from 'next/web-vitals';
import { logger } from '@/lib/logger';

const THRESHOLD = { LCP: 2500, INP: 200, CLS: 0.1 } as const;

export function WebVitals() {
  useReportWebVitals((metric) => {
    const limit = THRESHOLD[metric.name as keyof typeof THRESHOLD];
    const breached = limit !== undefined && metric.value > limit;
    logger[breached ? 'warn' : 'info']('web_vitals.report', {
      name: metric.name,
      value: metric.value,
      rating: metric.rating,
      id: metric.id,
    });
  });
  return null;
}
```

```tsx
// src/app/layout.tsx
import { WebVitals } from './web-vitals';
// ...
<body>
  <WebVitals />
  {children}
</body>
```

---

## Rule 7 — List > 100 item PHẢI virtualize; component render > 16ms PHẢI memo / code-split

Lý do: 1 frame = 16ms ở 60fps. List dài render đầy đủ blocks UI thread và phá INP.

### ❌ Sai

```tsx
'use client';
export function VocabularyList({ words }: { words: Vocabulary[] }) {
  return (
    <ul>
      {words.map((w) => <WordCard key={w.id} word={w} />)}   {/* 1000 item → lag */}
    </ul>
  );
}
```

### ✅ Đúng

```tsx
'use client';
import { useVirtualizer } from '@tanstack/react-virtual';
import { useRef } from 'react';

export function VocabularyList({ words }: { words: Vocabulary[] }) {
  const parentRef = useRef<HTMLDivElement>(null);
  const v = useVirtualizer({
    count: words.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 64,
    overscan: 8,
  });

  return (
    <div ref={parentRef} className="h-[600px] overflow-auto">
      <div style={{ height: v.getTotalSize(), position: 'relative' }}>
        {v.getVirtualItems().map((row) => (
          <div key={row.key} style={{ position: 'absolute', top: row.start, height: row.size, width: '100%' }}>
            <WordCard word={words[row.index]} />
          </div>
        ))}
      </div>
    </div>
  );
}
```

### Ghi chú
`@tanstack/react-virtual` là package mới — phải qua governance §8 trước khi cài.

---

## Rule 8 — Track event nghiệp vụ quan trọng (auth, test session, AI generate) qua `logger.info`; CẤM bỏ trống

Lý do: không track = không biết feature có ai dùng → không quyết định được scope sau.

### ❌ Sai

```ts
const submit = useMutation({ mutationFn: submitTestSession });
// không track → không biết bao nhiêu user hoàn thành test
```

### ✅ Đúng

```ts
const submit = useMutation({
  mutationFn: submitTestSession,
  onSuccess: (res) => {
    logger.info('test_session.submit.success', {
      type: res.type,
      total: res.totalCount,
      correct: res.correctCount,
      durationSec: res.durationSec,
    });
  },
  onError: (error) => logger.error('test_session.submit.failed', { error }),
});
```

---

## Checklist tự kiểm

- [ ] Không còn `console.log` trong code merge.
- [ ] Mọi log đi qua `@/lib/logger`.
- [ ] Cấp log đúng nghĩa.
- [ ] Tên event đúng pattern `feature.action.result`.
- [ ] Không log password/token/cookie/PII; có `redact()` khi cần.
- [ ] Không gọi SDK tracker trực tiếp trong component.
- [ ] `<WebVitals />` đã gắn ở root layout.
- [ ] List dài đã virtualize.
- [ ] Auth + test session + AI generate có event track.
