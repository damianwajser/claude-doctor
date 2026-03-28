---
name: audit
description: Quick audit of a Claude Code project structure. Scans the target project, evaluates CLAUDE.md quality, and identifies critical issues. Use when someone says "audit", "review project", "check project structure", or provides a project path to analyze.
argument-hint: <target-project-path>
allowed-tools: Read, Grep, Glob, Agent
disable-model-invocation: true
context: fork
---

# Quick Audit

Perform a quick audit of the Claude Code configuration for the project at `$ARGUMENTS`.

## Steps

1. **Validate input**: Confirm `$ARGUMENTS` is a valid directory path. If empty, ask the user for the target project path.

2. **Scan the project**: Use the `project-scanner` agent to map the full Claude Code configuration structure at the target path. Pass the full path as the agent's prompt.

3. **Audit configuration**: Use the `claude-config-auditor` agent to evaluate CLAUDE.md files, settings, and rules. Pass the scanner's output as context.

4. **Quick findings**: Based on the scanner and config auditor results, present a summary:

```
## Quick Audit: [project-name]

### Health Score: [A-F]

### Critical Issues (fix now)
- ...

### Top 5 Improvements
1. ...
2. ...
3. ...
4. ...
5. ...

### Run `/audit-full $ARGUMENTS` for a comprehensive analysis
```

## Important
- Do NOT modify any files in the target project
- If the target is part of a monorepo, mention it and recommend `/audit-full` for multi-project analysis
- Keep output concise — this is a quick scan, not a deep dive
- Reference `docs/claude-code-reference.md` for best practice validations
