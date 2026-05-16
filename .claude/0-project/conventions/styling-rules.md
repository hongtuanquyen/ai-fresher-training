# Styling Rules — FE

> **Role**: Frontend
> **Nguồn tham chiếu**: `.claude/FE-architecture.md` (Tailwind CSS v4 + shadcn/ui)

**Phạm vi áp dụng**: mọi file `.tsx` / `.ts` có style; `globals.css`; `tailwind.config.ts`.
**Mức độ bắt buộc**: **MUST — vi phạm = block PR**.

---

## Rule 1 — CHỈ dùng Tailwind utility; CẤM CSS module / styled-components / styled-jsx

Lý do: dự án đã chốt utility-first. Trộn nhiều phương pháp gây CSS specificity war và tăng bundle.

### ❌ Sai

```css
/* src/components/vocabulary/word-card.module.css */
.card { padding: 16px; border-radius: 8px; }
```

```tsx
import styles from './word-card.module.css';
export function WordCard() { return <div className={styles.card}>...</div>; }
```

### ✅ Đúng

```tsx
export function WordCard() {
  return <div className="rounded-lg p-4">...</div>;
}
```

### Ghi chú
`globals.css` CHỈ chứa: Tailwind directive, CSS variable cho shadcn theme, font-face. KHÔNG viết class custom rải rác.

---

## Rule 2 — Conditional className PHẢI dùng `cn()` từ `@/lib/utils`; CẤM concat string thủ công

Lý do: `cn()` (clsx + tailwind-merge) tự dedupe conflict (`p-2 p-4` → `p-4`); concat string không.

### ❌ Sai

```tsx
<button className={`rounded px-4 py-2 ${isActive ? 'bg-blue-500 text-white' : 'bg-gray-200'} ${className || ''}`}>
```

### ✅ Đúng

```tsx
import { cn } from '@/lib/utils';

<button
  className={cn(
    'rounded px-4 py-2',
    isActive ? 'bg-primary text-primary-foreground' : 'bg-muted',
    className,
  )}
/>
```

---

## Rule 3 — CẤM hard-code màu/spacing/font; PHẢI dùng token Tailwind + biến theme shadcn

Lý do: hard-code phá dark mode, theming, design system consistency.

### ❌ Sai

```tsx
<div className="bg-[#3b82f6] text-[#ffffff] p-[13px]" style={{ fontSize: '15px' }}>
```

### ✅ Đúng

```tsx
<div className="bg-primary text-primary-foreground p-3 text-sm">
```

### Ghi chú
Token semantic shadcn: `background`, `foreground`, `primary`, `secondary`, `muted`, `accent`, `destructive`, `border`, `ring`. Spacing: `0`, `1`, `2`, `3`, `4`, `6`, `8`, `12`, `16`, `24` — không tạo arbitrary value `p-[13px]`.

---

## Rule 4 — Mobile-first: viết base class trước, breakpoint prefix (`sm:`, `md:`, `lg:`) cho màn lớn hơn

Lý do: Tailwind breakpoint là `min-width`. Viết desktop trước rồi override xuống mobile sai về mặt logic và dễ sót.

### ❌ Sai

```tsx
<div className="grid-cols-3 md:grid-cols-2 sm:grid-cols-1 grid">
```

### ✅ Đúng

```tsx
<div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3">
```

---

## Rule 5 — Inline `style={{}}` CHỈ cho giá trị runtime không thể static; PHẢI có comment lý do

Lý do: inline style bypass theme, không tree-shake, ưu tiên hơn class.

### ❌ Sai

```tsx
<div style={{ padding: 16, backgroundColor: 'white', borderRadius: 8 }}>
```

### ✅ Đúng

```tsx
// Lý do: progress là số runtime, Tailwind không có util cho width động.
<div
  className="h-2 rounded bg-primary"
  style={{ width: `${progressPercent}%` }}
/>
```

---

## Rule 6 — Override shadcn component qua `cva` variant; CẤM lặp class > 2 lần ở call-site

Lý do: lặp class là magic; thêm variant đưa quyết định thiết kế vào 1 chỗ.

### ❌ Sai

```tsx
// 5 nơi khác nhau đều viết:
<Button className="bg-emerald-600 hover:bg-emerald-700 text-white">Save</Button>
```

### ✅ Đúng

```tsx
// src/components/ui/button.tsx — thêm variant 'success' vào buttonVariants (cva)
const buttonVariants = cva(/* base */, {
  variants: {
    variant: {
      default: '...',
      success: 'bg-emerald-600 text-white hover:bg-emerald-700',
    },
  },
});
```

```tsx
<Button variant="success">Save</Button>
```

---

## Rule 7 — Dark mode: dùng prefix `dark:`; CẤM check theme bằng JS để switch class

Lý do: prefix `dark:` chạy ở CSS layer, không re-render React; check JS gây flash + waste render.

### ❌ Sai

```tsx
'use client';
const theme = useTheme();
<div className={theme === 'dark' ? 'bg-zinc-900 text-white' : 'bg-white text-black'}>
```

### ✅ Đúng

```tsx
<div className="bg-background text-foreground">
{/* token tự đổi theo .dark class trên <html> */}
```

---

## Rule 8 — Thứ tự className: layout → spacing → sizing → typography → color → state → responsive

Lý do: thứ tự thống nhất giảm cognitive load khi review diff. Bật `prettier-plugin-tailwindcss` để tự sort.

### ❌ Sai

```tsx
<button className="hover:bg-primary/90 text-white p-4 flex bg-primary rounded items-center md:p-6">
```

### ✅ Đúng

```tsx
<button className="flex items-center rounded bg-primary p-4 text-white hover:bg-primary/90 md:p-6">
```

---

## Checklist tự kiểm

- [ ] Không có CSS module / styled-components / styled-jsx mới.
- [ ] Conditional class qua `cn()`.
- [ ] Không hex/px hard-code; dùng token Tailwind + theme shadcn.
- [ ] Mobile-first (base → `sm:` → `md:` → `lg:`).
- [ ] `style={{}}` chỉ cho runtime value + có comment.
- [ ] Class lặp > 2 lần đã chuyển thành `cva` variant.
- [ ] Dark mode bằng `dark:` prefix, không qua JS.
- [ ] `prettier-plugin-tailwindcss` đã sort class.
