---
paths:
  - ".claude/skills/**/*.md"
---

# Skill Design Rules

When creating or modifying skills in this project:

- All user-facing skills MUST have `disable-model-invocation: true` (audit/fix operations should be explicit)
- Every skill MUST have a `description` with keywords matching natural language triggers
- Use `$ARGUMENTS` for the target project path (always the first argument)
- Use `allowed-tools` to restrict tool access (audit skills = read-only + Agent)
- Define explicit output format in the skill instructions
- Use directive/imperative language, not conversational
- Keep under 500 lines per SKILL.md
- `argument-hint` must show expected arguments for user clarity
