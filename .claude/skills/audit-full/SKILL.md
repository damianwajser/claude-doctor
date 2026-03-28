---
name: audit-full
description: Comprehensive deep audit of a Claude Code project. Runs all specialized auditors (config, skills, agents, hooks, multi-project) in parallel and generates a complete improvement plan. Use when someone says "full audit", "deep audit", "comprehensive review", or wants a complete analysis.
argument-hint: <target-project-path> [--include-siblings] [--fix]
allowed-tools: Read, Grep, Glob, Bash, Agent
disable-model-invocation: true
effort: max
---

# Comprehensive Audit

Perform a full, deep audit of the Claude Code configuration for the project at `$0`.

## Options
- `--include-siblings`: Also audit sibling projects in the same parent directory
- `--fix`: After generating the plan, automatically apply Phase 1 (Foundation) fixes

## Steps

### Phase 1: Scan
Use the `project-scanner` agent to produce a complete structural map. Pass `$0` as the agent's prompt.

Wait for the scan to complete before proceeding.

### Phase 2: Parallel Audits

Based on the scan results, launch auditors **in parallel**:

1. **Always run**: `claude-config-auditor` — pass the scan report
2. **If skills found OR project could benefit from skills**: `skills-auditor` — pass scan report
3. **If agents found OR project could benefit from agents**: `agents-auditor` — pass scan report
4. **If hooks found OR settings.json exists**: `hooks-auditor` — pass scan report
5. **If monorepo/multi-project detected**: `multi-project-auditor` — pass scan report + full target path

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

### Multi-Project Audit
[If applicable — multi-project findings]

---

### Improvement Plan
[Full plan from plan-generator]

---

### Next Steps
- Review the plan above
- Run `/fix $0` to apply approved changes
- Run `/report $0` to export this audit as a markdown file
```

### Phase 5 (if --fix flag)

If `--fix` was specified:
1. Ask user to confirm Phase 1 changes
2. Use `rewriter` agent to apply Phase 1 only
3. Report changes made

## Important
- This is a LONG operation — set expectations with the user
- Run auditors in parallel whenever possible to save time
- If the project is large, focus on the most impactful areas first
- Reference `docs/claude-code-reference.md` for all validations
