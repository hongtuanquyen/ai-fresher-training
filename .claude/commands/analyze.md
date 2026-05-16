---
description: Phase Analysis — chạy subagent BA analyze trước, rồi dùng output đó cho subagent Fe analyze. Output ba.md + fe.md trong 01-analysis.
argument-hint: <mã task> (vd T-001)
---

# Analyze — Phase 2 (BA → Fe)

Điều phối phase Analysis cho task `$1`: BA phân tích trước, Fe phân tích sau
dựa trên output của BA. Tuần tự, KHÔNG song song (Fe cần `ba.md`).

## Input

- Mã task: `$1` (bắt buộc, vd `T-001`). Rỗng → DỪNG, hỏi user mã task.

## Quy trình

### Bước 1 — BA analyze

- Gọi subagent **business-analysist** với yêu cầu: "analyze task `$1`".
- BA (router) sẽ delegate sang skill `analyze-ba`.
- Kỳ vọng output: `.claude/0-project/tasks/$1/01-analysis/ba.md`.
- Nếu BA DỪNG (vd `context-ba.md` rỗng) → báo user lý do, DỪNG cả command,
  KHÔNG chạy bước 2.

### Bước 2 — Fe analyze (dùng output BA)

- Chỉ chạy khi `01-analysis/ba.md` đã tồn tại và không còn là template.
- Gọi subagent **front-end** với yêu cầu: "analyze task `$1`, dùng
  `01-analysis/ba.md` làm input".
- Fe (router) sẽ delegate sang skill `analyze-fe`.
- Kỳ vọng output: `.claude/0-project/tasks/$1/01-analysis/fe.md`.
- Nếu Fe DỪNG (vd `context-fe.md` rỗng, hoặc `ba.md` chưa đạt) → báo user
  lý do, DỪNG.

### Bước 3 — Tổng kết

- In bảng: bước | subagent | skill | file output | trạng thái.
- Xác nhận đã có cả `01-analysis/ba.md` và `01-analysis/fe.md`.

## Ràng buộc

- Tuần tự BA → Fe; Fe không chạy nếu BA chưa ra `ba.md` hợp lệ.
- Mỗi subagent là router, chỉ thực thi qua skill đã define
  (`analyze-ba` / `analyze-fe`) — command không tự phân tích thay.
- Không sửa file ngoài `01-analysis/` (skill tương ứng tự ghi).
