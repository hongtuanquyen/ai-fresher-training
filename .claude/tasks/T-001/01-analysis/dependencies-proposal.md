# Dependencies Proposal

> Chỉ tạo file này khi cần package mới chưa có trong `.claude/mobile/architecture/tech-stack.md`.
> Mỗi package = 1 entry. Phải pass governance gate (§8) trước khi dùng trong code.

## DEP-01: `<package-name>` @ `<version>`

- **Platform**: iOS | Android | Both
- **Mục đích**: <tại sao cần>
- **Alternatives đã cân nhắc**:
  - `<alt-1>` — lý do không chọn: ...
  - `<alt-2>` — lý do không chọn: ...
- **License**: MIT | Apache-2.0 | BSD-3-Clause | ...
- **Maintenance**:
  - Last commit: YYYY-MM-DD
  - Contributors: <số>
  - Weekly downloads: <số>
  - GitHub stars: <số>
- **Security**:
  - Known CVE: có / không (link)
  - `npm audit` result: ...
- **Bundle size impact**: +<X> KB (gzip)
- **Native dependency**: có / không
  - iOS: cần `pod install`? autolink?
  - Android: cần `gradle sync`? manual link?
- **Transitive deps mới**: liệt kê dep phụ đáng chú ý
- **Risk assessment**: low | medium | high

### Approval

- [ ] License scanner pass
- [ ] `npm audit` pass
- [ ] Bundle analyzer pass (hoặc justify được)
- [ ] Human approve
- [ ] Updated `.claude/mobile/architecture/tech-stack.md`

---

## DEP-02: ...
