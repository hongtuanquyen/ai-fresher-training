---
description: Parse mọi raw file của một task (raw-spec) thành markdown trong parsed-spec. PDF dùng skill pdf-reading, file khác dùng skill markitdown.
argument-hint: <mã task> (vd T-001)
---

# Intake — parse raw spec

Parse toàn bộ file trong thư mục raw-spec của task `$1` thành markdown.

## Input

- Mã task: `$1` (bắt buộc, vd `T-001`).
- Thư mục nguồn: `.claude/0-project/tasks/$1/00-intake/raw-spec/`
- Thư mục đích: `.claude/0-project/tasks/$1/00-intake/parsed-spec/`

> Nếu `$1` rỗng → DỪNG, hỏi user mã task.
> Nếu thư mục raw-spec không tồn tại hoặc rỗng (bỏ qua file `README`) →
> báo user và DỪNG.

## Quy trình

1. Liệt kê mọi file trong `raw-spec/` (bỏ qua `README`, file ẩn).
2. Với mỗi file:
   - **Đuôi `.pdf`** → dùng skill **pdf-reading** để trích xuất nội dung.
   - **Đuôi khác** (`.xlsx`, `.docx`, `.pptx`, `.csv`, `.html`, `.epub`,
     `.ipynb`, `.msg`, ảnh, ...) → dùng skill **markitdown**.
3. Ghi kết quả ra `parsed-spec/<tên-file-gốc-không-đuôi>.md`.
   - Ví dụ: `spec.pdf` → `parsed-spec/spec.md`;
     `requirement.xlsx` → `parsed-spec/requirement.md`.
   - Nếu trùng tên (vd `a.pdf` và `a.docx`) → thêm hậu tố đuôi gốc:
     `a-pdf.md`, `a-docx.md`.
4. Mỗi file output mở đầu bằng dòng:
   `> Nguồn: raw-spec/<tên file gốc> — parsed <YYYY-MM-DD>`
5. Ghi đè nếu file parsed đã tồn tại.

## Output

- Mỗi raw file → 1 file `.md` cùng tên trong `parsed-spec/`.
- Cuối cùng in bảng tổng kết: file gốc | skill dùng | file output | trạng thái.

## Ràng buộc

- Chỉ ghi vào `parsed-spec/` — không sửa file trong `raw-spec/`.
- Không suy diễn, bịa nội dung — chỉ trích xuất từ file gốc.
- Không cài package; markitdown/pdf-reading tự xử lý.
