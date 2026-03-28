---
name: audit-full
description: Comprehensive deep audit of a Claude Code project. Runs all specialized auditors (config, skills, agents, hooks, mcp, multi-project) in parallel and generates a complete improvement plan. Use when someone says "full audit", "deep audit", "comprehensive review", or wants a complete analysis.
argument-hint: <target-project-path> [--include-siblings]
allowed-tools: Read, Grep, Glob, Agent
disable-model-invocation: true
effort: max
context: fork
---

# Comprehensive Audit

Perform a full, deep audit of the Claude Code configuration for the project at `$ARGUMENTS`.

## Options
- `--include-siblings`: Also audit sibling projects in the same parent directory

## Steps

### Phase 1: Scan
Use the `project-scanner` agent to produce a complete structural map. Pass `$ARGUMENTS` as the agent's prompt.

Wait for the scan to complete before proceeding.

### Phase 2: Parallel Audits

Based on the scan results, launch auditors **in parallel**:

1. **Always run**: `claude-config-auditor` — pass the scan report
2. **If skills found OR project could benefit from skills**: `skills-auditor` — pass scan report
3. **If agents found OR project could benefit from agents**: `agents-auditor` — pass scan report
4. **If hooks found OR settings.json exists**: `hooks-auditor` — pass scan report
5. **If MCP config found (.mcp.json, .mcp.local.json, .claude/.mcp.json)**: `mcp-auditor` — pass scan report
6. **If monorepo/multi-project detected**: `multi-project-auditor` — pass scan report + full target path

### Phase 3: Generate Plan

Once all auditors complete, use the `plan-generator` agent. Pass ALL audit reports as context.

### Phase 4: Present Results

Display the full audit report:

```
## Full Audit Report: [project-name]
### Generated: [date]

---

### Executive Summary
- Health Score: [A-F]
- Critical issues: [N]
- Warnings: [N]
- Suggestions: [N]
- Project type: [single/monorepo/child]

---

### Scan Results
[Condensed scanner output]

### Configuration Audit
[Config auditor findings]

### Skills Audit
[Skills auditor findings — or "No skills found. Recommendations below."]

### Agents Audit
[Agents auditor findings — or "No agents found. Recommendations below."]

### Hooks Audit
[Hooks auditor findings]

### MCP Audit
[If applicable — MCP auditor findings]

### Multi-Project Audit
[If applicable — multi-project findings]

---

### Improvement Plan
[Full plan from plan-generator]

---

### Next Steps
- Review the plan above
- Run `/fix $ARGUMENTS` to apply approved changes
- Run `/report $ARGUMENTS` to export this audit as a markdown file
```

## Important
- This is a LONG operation — set expectations with the user
- Run auditors in parallel whenever possible to save time
- If the project is large, focus on the most impactful areas first
- Reference `docs/claude-code-reference.md` for all validations
