# Manual Test Checklist

> Human tick từng item trên real device (cả iOS + Android nếu cross-platform).
> AI không tự tick. Mọi item phải pass trước Gate G5.

## Devices used

- [ ] iOS: <model> / iOS <version>
- [ ] Android: <model> / Android <version>

## 1. UI visual (so với Figma)

- [ ] Screen `<X>` — layout, spacing, color, typography khớp Figma
- [ ] Screen `<Y>` — ...
- [ ] Icon, image hiển thị đúng (cả @1x/@2x/@3x)
- [ ] Dark mode (nếu support)

## 2. Animation & transition

- [ ] Navigation transition mượt
- [ ] Loading skeleton/spinner xuất hiện đúng lúc
- [ ] Animation không giật (≥55 fps cảm nhận)

## 3. Interaction states

- [ ] Loading state hiển thị
- [ ] Empty state hiển thị + CTA đúng
- [ ] Error state hiển thị + retry hoạt động
- [ ] Disabled state visual rõ ràng
- [ ] Success state / toast hiển thị

## 4. Platform-specific behavior

- [ ] iOS: swipe back, safe area, haptics, keyboard avoidance
- [ ] Android: hardware back button, status bar, keyboard avoidance
- [ ] Notch / Dynamic Island / cutout không che nội dung

## 5. Network conditions

- [ ] Online (wifi)
- [ ] Online (4G)
- [ ] Slow 3G — loading state đủ lâu, không freeze
- [ ] Offline — error/empty state đúng convention
- [ ] Reconnect — recover đúng

## 6. Auth & permission (nếu có)

- [ ] Token hết hạn → refresh / re-login đúng flow
- [ ] Permission denied (camera/photo/notification) — fallback UI

## 7. Deep link (nếu có)

- [ ] Cold start từ deep link
- [ ] Warm start từ deep link
- [ ] Invalid deep link → fallback

## 8. Edge cases (từ `ba.md` §7)

- [ ] EC-01: ...
- [ ] EC-02: ...

## 9. a11y manual

- [ ] VoiceOver đọc đúng label, thứ tự logic
- [ ] TalkBack đọc đúng
- [ ] Touch target dễ tap (không miss)
- [ ] Dynamic type / font scaling không vỡ layout

## 10. Cross-build check

- [ ] Dev build hoạt động
- [ ] Staging build hoạt động
- [ ] Release build hoạt động (no debug code, no console.log thừa)

## Sign-off

- Tested by: <name>
- Date: YYYY-MM-DD
- Result: ✅ pass / ❌ có issue (link issues)
