---
name: markitdown
description: >
  Converts uploaded files to Markdown using the markitdown CLI tool — fast, offline,
  zero token cost for conversion. Use this skill whenever the user uploads a file and
  asks to convert it to Markdown, read its contents, or extract text from it. Triggers
  on: "convert to markdown", "đọc file", "chuyển sang markdown", "extract content",
  "read this file", or any time the user uploads a .xlsx, .docx, .pptx, .pdf, .csv,
  .html, .epub, .ipynb, .msg, or image file and wants its content as text/markdown.
  Always prefer this skill over manual parsing with Python — it saves tokens and handles
  multi-sheet Excel, multi-slide PowerPoint, and complex PDFs automatically.
---

# markitdown Skill

Convert any supported file to Markdown in one bash call. No Python parsing, no token-heavy
manual extraction — just run the CLI and deliver the `.md` file.

## Supported Formats

| Category | Extensions |
|---|---|
| Office | `.xlsx`, `.docx`, `.pptx` |
| Documents | `.pdf`, `.epub` |
| Data | `.csv` |
| Web | `.html`, `.htm` |
| Code/Notebooks | `.ipynb` |
| Email | `.msg` |
| Media (caption only) | `.jpg`, `.jpeg`, `.png`, `.gif`, `.webp`, `.mp3`, `.wav` |
| Other | `.zip`, `.xml`, `.rss`, plain text |

## Workflow

### Step 1 — Install (if needed)

```bash
pip install markitdown --break-system-packages -q
```

### Step 2 — Convert

```bash
markitdown /mnt/user-data/uploads/<filename> -o /mnt/user-data/outputs/<filename>.md
sed -i '' 's/NaN//g' /mnt/user-data/outputs/<filename>.md   # macOS; dùng `sed -i` trên Linux
```

Output goes straight to `/mnt/user-data/outputs/`. Bước `sed` thay `NaN` (do pandas render từ ô Excel rỗng) thành chuỗi rỗng — luôn chạy mặc định.

### Step 3 — Present

Use `present_files` to share the `.md` file with the user.

---

## Edge Cases

**Multiple files at once**: Loop over each uploaded file:

```bash
for f in /mnt/user-data/uploads/*.xlsx; do
  base=$(basename "$f" .xlsx)
  markitdown "$f" -o "/mnt/user-data/outputs/${base}.md"
done
```

**Unknown/unsupported extension**: markitdown will attempt plain-text extraction — usually fine for `.txt`, `.log`, `.xml`. If it fails, fall back to `cat` for plain text or inform the user.

**Large files**: markitdown streams output so memory is not an issue. For very large Excel
files (100+ sheets), conversion may take 10–30 seconds — normal.

**Password-protected files**: markitdown cannot decrypt. Inform the user and ask for an
unprotected version.

**Images / audio**: markitdown will extract metadata/EXIF only (no OCR, no transcription)
unless the `-d` Document Intelligence flag is used with an Azure endpoint. For OCR needs,
suggest the `pdf-reading` skill instead.

---

## Output Naming Convention

Use the original filename stem + `.md`:

| Input | Output |
|---|---|
| `report.xlsx` | `report.md` |
| `slides.pptx` | `slides.md` |
| `doc.docx` | `doc.md` |

If the user requests a specific output name, use that instead.

---

## Common Issues

| Symptom | Fix |
|---|---|
| `markitdown: command not found` | Run `pip install markitdown --break-system-packages -q` first |
| Empty output | File may be image-only PDF or corrupted — try `pdftotext` as fallback |
| `NaN` values in Excel tables | Replace with empty string after conversion: `sed -i '' 's/NaN//g' <output.md>` (macOS) or `sed -i 's/NaN//g' <output.md>` (Linux). Always apply by default — empty cells should render as empty, not `NaN`. |
| Garbled encoding | Add `--charset utf-8` flag |
