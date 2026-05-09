# Self-Test Report

> Kết quả tự động: unit + component + e2e + a11y + i18n + perf. Kết thúc bằng traceability matrix.

## 1. Test execution summary

| Type | Total | Pass | Fail | Skipped | Coverage |
|---|---|---|---|---|---|
| Unit | <N> | <N> | <N> | <N> | <N>% |
| Component | <N> | <N> | <N> | <N> | <N>% |
| E2E | <N> | <N> | <N> | <N> | — |

## 2. Failed tests (nếu có)

| Test | Reason | Action |
|---|---|---|
| `should_xxx_when_yyy` | ... | fix / accepted |

## 3. Snapshot tests

- Updated: <list>
- Reviewed: Y/N

## 4. a11y check

| Check | Tool | Result |
|---|---|---|
| Touch target ≥ 44pt | manual + lint | ✅/❌ |
| VoiceOver/TalkBack labels | manual | ✅/❌ |
| Contrast WCAG AA | accessibility inspector | ✅/❌ |
| Focus order | manual | ✅/❌ |

## 5. i18n check

| Check | Tool | Result |
|---|---|---|
| Hard-coded string | ESLint | ✅/❌ |
| Missing key (en/ja) | script | ✅/❌ |
| Pseudo-locale | render | ✅/❌ |
| RTL (nếu support) | visual | ✅/❌ |

## 6. Performance

> Link tới `03-implementation/perf-<task>.md` nếu có hot path.

- Cold start: ✅/❌
- FPS: ✅/❌
- Bundle size: ✅/❌
- Memory: ✅/❌

## 7. Observability verification

- [ ] Analytics events có gọi đúng nơi (cho screen/flow mới)
- [ ] Crash breadcrumbs có cài cho navigation + API
- [ ] Performance traces có wire (cho hot path)

## 8. Scope lock verification

- [ ] Không có file nào nằm ngoài allowlist của các sub-task

## 9. Traceability matrix

| AC ID | Spec ref | Code files | Test files | Status |
|---|---|---|---|---|
| AC-01.1 | `ba.md` §4 | `App/.../X.tsx:12` | `__tests__/X.test.tsx` | ✅ |
| AC-01.2 | `ba.md` §4 | ... | ... | ✅ |
| AC-02.1 | `ba.md` §4 | ... | ... | ✅ |

> Nếu có dòng thiếu cột (code/test) → CHƯA Done.

## 10. Gate G5 readiness

- [ ] Traceability đủ
- [ ] Manual checklist (`manual-test-checklist.md`) đã xong trên real device
- [ ] a11y pass
- [ ] i18n pass
- [ ] Perf report vs budget pass
