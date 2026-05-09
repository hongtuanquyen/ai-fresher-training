# Implementation Plan

> Chia task thành sub-tasks nhỏ (target <300 dòng/sub-task). Test-first order.
> Mỗi sub-task có DoD + scope allowlist (hard limit cho AI khi gen code).

## Tổng quan

- Task: <task-id> — <tên>
- Số sub-tasks: <N>
- Estimate tổng: <thời gian>
- Hot path? (startup, navigation transition): có / không → perf check ở G4

## Thứ tự thực hiện

`Shared types/models → Test (test-first) → Component/screen → API integration → Navigation → E2E test`

Xem `dependency-graph.mermaid`.

---

## T-01: <tên ngắn>

- **Layer**: Mobile | Test | Integration
- **Depends on**: [—]
- **Files to create**:
  - `App/.../NewFile.tsx`
- **Files to touch** (allowlist — AI KHÔNG được sửa file ngoài danh sách này):
  - `App/.../ExistingFile.tsx`
- **Traces to**: AC-01.1, AC-01.2
- **Definition of Done**:
  - [ ] Code compile (iOS + Android)
  - [ ] Lint + typecheck pass
  - [ ] Unit test cho AC-01.1, AC-01.2 pass
  - [ ] Không sửa file ngoài allowlist
  - [ ] Commit `[T-01] <mô tả>`
- **Estimated size**: ~<N> dòng (target <300)
- **Rollback**: revert commit

---

## T-02: <tên ngắn>

- **Layer**: ...
- **Depends on**: [T-01]
- **Files to create**: ...
- **Files to touch**: ...
- **Traces to**: AC-...
- **Definition of Done**:
  - [ ] ...
- **Estimated size**: ~<N> dòng
- **Rollback**: ...

---

## T-03: ...

---

## Gate G3 Checklist

- [ ] Mọi sub-task có DoD và allowlist rõ
- [ ] Test-first order đúng (test sub-task trước code sub-task tương ứng)
- [ ] Mọi AC trong `ba.md` đã được trace tới ít nhất 1 sub-task
- [ ] Perf check schedule cho G4 (nếu hot path)
- [ ] Human approved
