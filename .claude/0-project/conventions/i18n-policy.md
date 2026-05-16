# i18n Policy — FE

> **Role**: Frontend
> **Nguồn tham chiếu**: `.claude/FE-architecture.md` (UI tiếng Việt mặc định, lib i18n: **chưa chốt** — TBD theo lib chốt sau)

**Phạm vi áp dụng**: mọi chuỗi UI hiển thị cho user trong `src/`.
**Mức độ bắt buộc**: **MUST — vi phạm = block PR**.

> **Ghi chú stack**: Architecture chốt UI tiếng Việt và CHƯA chỉ định lib i18n
> (next-intl / react-i18next). Convention dưới dùng pattern **messages object
> trung tâm** trong `src/lib/messages.ts` để dễ migrate sang lib khi cần.

---

## Rule 1 — CẤM hard-code chuỗi UI rải rác trong JSX; mọi chuỗi PHẢI đi qua `messages` registry

Lý do: kể cả app 1 ngôn ngữ, tập trung chuỗi giúp đổi từ ngữ nhanh, search dễ, và migrate i18n sau này không phải sửa cả codebase.

### ❌ Sai

```tsx
export function FolderEmpty() {
  return (
    <div>
      <h2>Bạn chưa có folder nào</h2>
      <p>Tạo folder đầu tiên để bắt đầu học từ vựng.</p>
      <Button>Tạo folder</Button>
    </div>
  );
}
```

### ✅ Đúng

```ts
// src/lib/messages.ts
export const messages = {
  folder: {
    empty_title: 'Bạn chưa có folder nào',
    empty_description: 'Tạo folder đầu tiên để bắt đầu học từ vựng.',
    create_cta: 'Tạo folder',
  },
} as const;

export const t = (key: string): string => {
  const parts = key.split('.');
  let cur: unknown = messages;
  for (const p of parts) cur = (cur as Record<string, unknown>)?.[p];
  if (typeof cur !== 'string') throw new Error(`Missing message: ${key}`);
  return cur;
};
```

```tsx
import { t } from '@/lib/messages';

export function FolderEmpty() {
  return (
    <div>
      <h2>{t('folder.empty_title')}</h2>
      <p>{t('folder.empty_description')}</p>
      <Button>{t('folder.create_cta')}</Button>
    </div>
  );
}
```

---

## Rule 2 — Quy ước key: `feature.action_or_field` (dot notation, snake_case)

Lý do: namespace theo feature giúp split chunk khi migrate sang lib; snake_case dễ đọc trong key dài.

### ❌ Sai

```ts
{
  CreateFolderButton: 'Tạo folder',
  emptyMsg1: 'Chưa có folder',
  'random-string-123': 'Lưu',
}
```

### ✅ Đúng

```ts
{
  folder: {
    create_cta: 'Tạo folder',
    empty_title: 'Chưa có folder',
  },
  common: {
    save: 'Lưu',
    cancel: 'Hủy',
  },
}
```

---

## Rule 3 — CẤM trộn logic + text trong template string; PHẢI dùng hàm format

Lý do: template string `\`Bạn có ${n} từ\`` không thể migrate sang ICU MessageFormat; plural sai.

### ❌ Sai

```tsx
<p>{`Bạn có ${count} từ trong folder ${folderName}`}</p>
```

### ✅ Đúng

```ts
// src/lib/messages.ts
export const formatters = {
  folder: {
    word_count: (count: number, folderName: string) =>
      `Bạn có ${count} từ trong folder "${folderName}"`,
  },
};
```

```tsx
import { formatters } from '@/lib/messages';
<p>{formatters.folder.word_count(count, folderName)}</p>
```

---

## Rule 4 — Date / number / currency PHẢI dùng `Intl.*` với locale `vi-VN`; CẤM hard-code format

Lý do: hard-code format vỡ khi đổi locale và sai chuẩn quốc tế hoá.

### ❌ Sai

```tsx
<span>{new Date(createdAt).toLocaleDateString()}</span>     // locale runtime, không xác định
<span>{`${(accuracy * 100).toFixed(2)}%`}</span>
<span>{`${price.toFixed(0)}đ`}</span>
```

### ✅ Đúng

```ts
// src/lib/format.ts
const LOCALE = 'vi-VN';

export const formatDate = (iso: string) =>
  new Intl.DateTimeFormat(LOCALE, { dateStyle: 'medium' }).format(new Date(iso));

export const formatPercent = (ratio: number) =>
  new Intl.NumberFormat(LOCALE, { style: 'percent', maximumFractionDigits: 1 }).format(ratio);

export const formatCurrency = (amount: number) =>
  new Intl.NumberFormat(LOCALE, { style: 'currency', currency: 'VND' }).format(amount);
```

```tsx
import { formatDate, formatPercent } from '@/lib/format';
<span>{formatDate(createdAt)}</span>
<span>{formatPercent(accuracy)}</span>
```

---

## Rule 5 — Plural / điều kiện hiển thị PHẢI qua helper trong `messages.ts`; CẤM ternary lồng để chọn từ

Lý do: ternary trong JSX khó migrate sang `Intl.PluralRules` hoặc ICU.

### ❌ Sai

```tsx
<p>{count === 0 ? 'Không có từ' : count === 1 ? '1 từ' : `${count} từ`}</p>
```

### ✅ Đúng

```ts
// src/lib/messages.ts
const pr = new Intl.PluralRules('vi-VN');

export const formatters = {
  vocabulary: {
    count_label: (count: number) => {
      switch (pr.select(count)) {
        case 'zero': return 'Không có từ';
        default: return `${count} từ`;
      }
    },
  },
};
```

```tsx
<p>{formatters.vocabulary.count_label(count)}</p>
```

---

## Rule 6 — Chuỗi mới PHẢI thêm vào `messages.ts` ngay cùng PR; CẤM tạm hard-code "để fix sau"

Lý do: "tạm" trở thành vĩnh viễn. Mỗi lần thêm chuỗi mất 30 giây thêm key — không có cớ skip.

### ❌ Sai

```tsx
<Button onClick={onSubmit}>Lưu nhanh</Button>   // ❌ thêm tạm
{/* TODO: i18n sau */}
```

### ✅ Đúng

```ts
// src/lib/messages.ts
export const messages = {
  common: { save: 'Lưu', save_quick: 'Lưu nhanh' },
};
```

```tsx
<Button onClick={onSubmit}>{t('common.save_quick')}</Button>
```

---

## Rule 7 — `lang="..."` trên thẻ HTML PHẢI khớp locale UI; chuỗi ngoại ngữ trong app dùng `<span lang="...">`

Lý do: screen reader phát âm sai khi `lang` sai; SEO bị nhầm ngôn ngữ.

### ❌ Sai

```tsx
// src/app/layout.tsx
<html>
  <body>
    <p>{englishWord}</p>
  </body>
</html>
```

### ✅ Đúng

```tsx
// src/app/layout.tsx
<html lang="vi">
  <body>{children}</body>
</html>
```

```tsx
// src/components/vocabulary/word-card.tsx
<span lang="en" className="text-2xl">{word}</span>
{ipa && <span lang="en-fonipa" className="text-sm text-muted-foreground">/{ipa}/</span>}
```

---

## Checklist tự kiểm

- [ ] Không có chuỗi UI hard-code trong JSX (dùng `t()` / `messages`).
- [ ] Key đúng pattern `feature.snake_case`.
- [ ] Không trộn logic + text trong template string.
- [ ] Date/number/currency qua `Intl.*` với `vi-VN`.
- [ ] Plural qua helper `Intl.PluralRules`.
- [ ] Chuỗi mới đã add vào `messages.ts` ngay cùng PR.
- [ ] `<html lang="vi">`; chuỗi tiếng Anh có `<span lang="en">`.
