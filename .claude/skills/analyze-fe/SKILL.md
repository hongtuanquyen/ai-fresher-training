---
name: analyze-fe
description: >
  Phase Analysis — role Fe. Đọc context manifest
  .claude/0-project/tasks/<task-id>/00-intake/context-fe.md để load đúng file
  context, đọc thêm 01-analysis/ba.md (acceptance criteria), rồi sinh phân
  tích kỹ thuật FE vào 01-analysis/fe.md theo template chuẩn 14 mục, map mọi
  mục về AC ID. Trigger khi user yêu cầu "phân tích FE", "analyze fe",
  "/analyze <task-id> --role=fe", hoặc cần phân tích kỹ thuật FE cho một
  task. KHÔNG trigger cho phân tích nghiệp vụ (dùng analyze-ba) hay viết
  code thật (đó là phase Implement).
---

# Analyze — Front-end (Fe)

Quy trình Fe của phase Analysis
(Intake → Analysis (BA → **Fe**) → Plan → Implement → Review).

## Nguyên tắc cốt lõi

- **Context manifest là luật**: `00-intake/context-fe.md` liệt kê file Fe
  được load. KHÔNG đọc file ngoài manifest (non-negotiable Context manifest).
- `ba.md` là **input bắt buộc**: mọi phần phân tích phải map về AC ID
  trong `ba.md`. Không có AC tương ứng → Open questions, không bịa.
- BẮT BUỘC đọc `.claude/0-project/issues-log.md` trước khi phân tích
  (học từ lỗi cũ — non-negotiable trong AGENTS.md).
- Chỉ ghi vào `01-analysis/fe.md` — không sửa file khác.
- Không tự cài/duyệt package; package mới → `dependencies-proposal.md`
  (governance §8), chỉ tham chiếu DEP-ID.

## Quy trình

1. Xác định `<task-id>`. Thiếu → hỏi user.
2. **Đọc context manifest**:
   `.claude/0-project/tasks/<task-id>/00-intake/context-fe.md`
   - Rỗng/chưa tồn tại → DỪNG, báo cần điền `context-fe.md` trước.
3. Đọc `.claude/0-project/tasks/<task-id>/01-analysis/ba.md` để lấy AC,
   business rule, screen DD delta.
   - Nếu `ba.md` còn template/rỗng → DỪNG, yêu cầu chạy analyze-ba trước.
4. Đọc đúng các file trong manifest (thường: design tokens/components,
   conventions, issues-log). Chỉ đọc nếu manifest liệt kê.
5. Phân tích kỹ thuật FE: component tree, state, routing, interaction
   states, a11y, i18n, API, dependencies.
6. Lập §13 Phạm vi ảnh hưởng & §14 Lưu ý sửa file common: grep toàn repo
   tìm consumer, liệt kê đầy đủ, đánh giá nguy cơ regression.
7. Map mọi component/screen/logic về AC ID ở §12.
8. Ghi ra `.claude/0-project/tasks/<task-id>/01-analysis/fe.md` đúng theo
   template dưới đây (ghi đè nếu đã có).

## Template output `01-analysis/fe.md`

````markdown
# Mobile Analysis

> Output của role Mobile. Input: `ba.md` + design tokens + Figma + API docs.
> Phải map tới AC ID trong `ba.md`.

## 1. Tổng quan kỹ thuật

- Screen mới: ...
- Screen sửa: ...
- Module bị ảnh hưởng: ...

## 2. Component tree (mỗi screen)

### Screen: `<ScreenName>`

```
<ScreenName>
├── <HeaderComponent>     [reuse từ design-system]
├── <FormSection>         [tạo mới]
│   ├── <InputField>      [reuse]
│   └── <SubmitButton>    [reuse]
└── <FooterNote>          [tạo mới]
```

- Reuse: liệt kê component từ `.claude/design-system/components.md`
- Tạo mới: liệt kê + lý do (chưa có trong design-system)

## 3. State management

| State | Scope (local/global) | Lưu ở               | Cache strategy |
| ----- | -------------------- | ------------------- | -------------- |
| ...   | ...                  | Redux/Context/local | ...            |

Tham chiếu: `.claude/mobile/conventions/state-management.md`

## 4. Navigation / routing

- Entry point: ...
- Route name + params: ...
- Back behavior: ...
- Deep link (nếu có): ...

## 5. Interaction states (đầy đủ cho mỗi screen/component)

| Component | Loading | Empty | Error | Disabled | Success |
| --------- | ------- | ----- | ----- | -------- | ------- |
| `<Name>`  | ...     | ...   | ...   | ...      | ...     |

## 6. Platform-specific behavior

| Behavior | iOS | Android |
| -------- | --- | ------- |
| ...      | ... | ...     |

## 7. Accessibility (a11y)

- Touch target size: ≥ 44x44 pt
- VoiceOver/TalkBack labels: liệt kê element + label
- Contrast: WCAG AA
- Focus order: ...

## 8. i18n keys mới

| Key              | en  | ja  |
| ---------------- | --- | --- |
| `screen.x.title` | ... | ... |

Tham chiếu: `.claude/mobile/conventions/i18n-policy.md`

## 9. API integration

| Screen/Action | Endpoint | Method | Loading UI | Error UI | Retry          |
| ------------- | -------- | ------ | ---------- | -------- | -------------- |
| ...           | `/...`   | GET    | skeleton   | toast    | 3x exp backoff |

Tham chiếu: `.claude/mobile/architecture/api-integration.md`

## 10. Offline behavior (nếu có)

- ...

## 11. Dependencies mới (nếu có)

> Mọi package mới phải có entry trong `dependencies-proposal.md`.

- DEP-01: `<package>@<version>` — xem `dependencies-proposal.md`

## 12. Mapping AC → Component/Screen

| AC ID   | Screen | Component | File dự kiến             |
| ------- | ------ | --------- | ------------------------ |
| AC-01.1 | ...    | ...       | `App/Containers/.../...` |

## 13. Phạm vi ảnh hưởng (impact scope)

> Liệt kê MỌI file/module bị task này chạm vào, kèm loại thay đổi.
> Đây là cơ sở để khoá scope (allowlist) ở phase Plan.
> Δ: 🆕 tạo mới · ✏️ sửa · 🗑️ xoá.

| Δ   | File / Module | Loại (screen/component/hook/service/store/util/config) | Lý do thay đổi | AC liên quan | Dùng chung? |
| --- | ------------- | ------------------------------------------------------ | -------------- | ------------ | ----------- |
| ✏️  | `App/.../...`  | ...                                                    | ...            | AC-01.1      | Không / Có  |

- **Ảnh hưởng trực tiếp**: file chứa logic chính của feature.
- **Ảnh hưởng gián tiếp**: file import/phụ thuộc vào file đã sửa (caller,
  type chung, theme/config). Phải truy ngược và liệt kê.
- **Không thuộc phạm vi**: ghi rõ ranh giới — file gần giống nhưng KHÔNG sửa.

## 14. Lưu ý đặc biệt — sửa file common (nhiều nơi dùng)

> File "common" = component/hook/util/type/service/config được ≥2 nơi
> import. Sửa file common = rủi ro regression diện rộng → bắt buộc xử lý
> theo checklist dưới đây.

### 14.1. Nhận diện file common

- Trước khi sửa, tìm mọi nơi import file đó (grep/search toàn repo).
- Liệt kê đầy đủ consumer ở bảng dưới — KHÔNG bỏ sót.

| File common | Số nơi dùng | Consumer chính (đường dẫn) | Nguy cơ regression |
| ----------- | ----------- | -------------------------- | ------------------ |
| `App/Components/<X>` | N | `...`, `...`             | thấp / vừa / cao   |

### 14.2. Nguyên tắc khi sửa

- **Backward-compatible trước**: ưu tiên thêm prop/param optional có
  default, KHÔNG đổi signature/behavior mặc định của consumer cũ.
- Nếu **buộc phải breaking**: liệt kê toàn bộ consumer cần sửa theo, đưa
  chúng vào "Phạm vi ảnh hưởng" (§13) và allowlist của plan — không sửa
  lén ngoài scope (non-negotiable Scope lock).
- **Không** nhồi logic riêng của feature vào file common; nếu cần hành vi
  riêng → tách biến thể/wrapper, giữ file common thuần dùng chung.
- Đổi style/token dùng chung → kiểm tra ảnh hưởng thị giác ở mọi consumer,
  không chỉ screen của task.

### 14.3. Bắt buộc trước khi đóng task

- [ ] Đã liệt kê đủ consumer của mỗi file common bị sửa.
- [ ] Thay đổi backward-compatible, hoặc mọi consumer breaking đã được
      cập nhật và nằm trong scope.
- [ ] Regression check thủ công các consumer rủi ro vừa/cao.
- [ ] Nếu phát sinh bug > 15 phút khi sửa file common → ghi
      `.claude/0-project/issues-log.md` (non-negotiable Issues-log).
````

## Ràng buộc (non-negotiables)

- Không load file ngoài `context-fe.md`.
- Mọi mục phải map về AC ID trong `ba.md`; không có AC → Open questions.
- Không tự cài/duyệt package — chỉ tham chiếu DEP-ID trong
  `dependencies-proposal.md`.
- Không viết code thật (đó là phase Implement) — chỉ phân tích.
- §13 và §14 phải đầy đủ: thiếu consumer của file common = chưa đạt.
