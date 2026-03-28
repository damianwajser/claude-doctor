---
paths:
  - ".claude/agents/*.md"
---

# Agent Design Rules

When creating or modifying agent definitions in this project:

- Every agent MUST have `name` (lowercase+hyphens) and `description` (keyword-rich)
- Audit agents (read-only) SHOULD restrict tools to: Read, Grep, Glob, and optionally Bash
- Only `rewriter` agent is allowed Write/Edit tools
- Always set `maxTurns` (20 for simple, 40 for complex tasks)
- Choose model deliberately: haiku=fast/simple, sonnet=balanced, opus=complex reasoning
- System prompt must follow single-responsibility principle
- Reference `docs/claude-code-reference.md` for validation rules
- Agent descriptions must be specific enough that Claude delegates accurately — vague descriptions cause mis-delegation
