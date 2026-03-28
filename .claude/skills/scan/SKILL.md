---
name: scan
description: Scan and map the Claude Code configuration of a target project without auditing. Use when someone says "scan project", "show structure", "map config", or just wants to see what Claude Code configuration exists.
argument-hint: <target-project-path>
allowed-tools: Read, Grep, Glob, Agent
disable-model-invocation: true
context: fork
effort: medium
---

# Project Scan

Map the complete Claude Code configuration structure of the project at `$ARGUMENTS`.

## Steps

1. **Validate input**: Confirm `$ARGUMENTS` is a valid directory path. If empty, ask the user for the target project path.

2. **Run scanner**: Use the `project-scanner` agent to produce a full structural map. Pass the complete path as the agent's prompt.

3. **Present results**: Display the scanner's report directly to the user. Include:
   - Project type (single/monorepo/child)
   - Parent context (if any)
   - All CLAUDE.md files with line counts
   - Complete .claude/ inventory
   - Subprojects and siblings (if monorepo)
   - Quick potential issues

4. **Suggest next steps**:
   ```
   ### Next Steps
   - Run `/audit $ARGUMENTS` for a quick quality audit
   - Run `/audit-full $ARGUMENTS` for a comprehensive deep audit
   ```

## Important
- This is READ-ONLY — no modifications, no quality judgments
- Just map and report what exists
- If monorepo detected, show the full ecosystem map
