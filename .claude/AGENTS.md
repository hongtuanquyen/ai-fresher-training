# Project Context

## 🌐 Context chung (mọi role bắt buộc đọc)

- @.claude/project.md — kiến trúc tổng, domain, ràng buộc
- @.claude/glossary.md — thuật ngữ nghiệp vụ (dùng nhất quán)

## 🎨 Design system (Fe)

- @.claude/design/tokens.json
- @.claude/design/components.md

## Role-specific context

- **Fe role** → `.claude/conventions/*`, `.claude/issues-log.md`

## Workflow

Mọi feature đi qua 5 phase với gates:
**Intake → Analysis (BA → Fe) → Plan → Implement → Review**

Dùng slash commands:

- `/intake <task-id>` — Phase 1
- `/analyze <task-id> [--role=ba|fe]` — Phase 2
- `/plan <task-id>` — Phase 3
- `/implement <task-id> <sub-task>` — Phase 4
- `/review <task-id>` — Phase 5

Mỗi role có subagent riêng trong `.claude/agents/role-*.md` — slash commands sẽ
tự delegate sang subagent tương ứng, KHÔNG tự làm thay.

## Learning from past mistakes

- @.claude/issues-log.md — subagent Fe BẮT BUỘC đọc trước khi gen code

## Non-negotiables (tóm tắt, xem §14 trong framework)

- Scope lock: không sửa file ngoài allowlist của sub-task
- No auto-install: package mới phải qua §8 governance
- Context manifest: không load file ngoài manifest của role
- Commit riêng mỗi sub-task với format `[T-XX] <mô tả>`
- Issues-log: bug mất >15 phút fix → bắt buộc entry (§13)
