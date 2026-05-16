---
description: Phase Plan — sinh implementation plan cho task qua skill implementation-planning. Output 02-plan/plan.md.
argument-hint: <mã task> (vd T-001)
---

# Plan — Phase 3 (Implementation Planning)

Sinh implementation plan cho task `$1` dựa trên kết quả phân tích phase
Analysis (BA → Fe). Plan là input bắt buộc cho phase Implement.

## Input

- Mã task: `$1` (bắt buộc, vd `T-001`). Rỗng → DỪNG, hỏi user mã task.

## Quy trình

### Bước 1 — Kiểm tra tiền điều kiện

- Yêu cầu đã tồn tại và không còn là template:
  - `.claude/0-project/tasks/$1/01-analysis/fe.md`
  - `.claude/0-project/tasks/$1/01-analysis/ba.md`
- Nếu `fe.md` chưa điền (còn template) → DỪNG, báo user chạy
  `/analyze $1 --role=fe` trước. KHÔNG tự suy diễn nội dung.

### Bước 2 — Sinh plan

- Gọi skill **implementation-planning** với yêu cầu: "tạo plan cho task `$1`".
- Skill đọc `01-analysis/fe.md`, `dependencies-proposal.md`, `ba.md` và sinh
  output theo template chuẩn.
- Kỳ vọng output: `.claude/0-project/tasks/$1/02-plan/plan.md`.

### Bước 3 — Tổng kết

- In tóm tắt: số sub-task, dependencies mới, trạng thái AC coverage.
- Xác nhận đã có `02-plan/plan.md` và pre-implement checklist được điền.

## Ràng buộc

- Chỉ thực thi qua skill `implementation-planning` — command không tự lập
  kế hoạch thay.
- Chỉ ghi vào `02-plan/plan.md` — không sửa file analysis.
- Không bỏ sót AC: thiếu coverage thì plan KHÔNG hợp lệ.
- Không tự cài/duyệt package mới — chỉ tham chiếu DEP-ID.
