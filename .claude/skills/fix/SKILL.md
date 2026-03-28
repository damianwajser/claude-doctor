---
name: fix
description: Apply improvements to a target Claude Code project based on a previously generated audit plan. Use when someone says "fix", "apply changes", "improve project", or "rewrite config". Requires a prior audit to have been run.
argument-hint: <target-project-path> [--phase N] [--item N]
allowed-tools: Read, Grep, Glob, Bash, Agent, Edit, Write
disable-model-invocation: true
effort: high
---

# Apply Fixes

Apply improvements to the Claude Code configuration at `$0`.

## Options
- `--phase N`: Apply only phase N from the plan (default: Phase 1)
- `--item N`: Apply only item N from the specified phase

## Prerequisites Check

1. Check if an audit has been run for this project in the current conversation
2. If no audit found, run `/audit-full $0` first and return the results before fixing

## Execution

1. **Confirm scope**: Show the user exactly what will be changed:
   ```
   ## Changes to Apply

   Phase [N]:
   | # | Change | File | Action |
   |---|--------|------|--------|
   | 1 | ... | ... | Create/Modify |

   Proceed? (y/n)
   ```

2. **Apply changes**: Use the `rewriter` agent to execute the approved changes. Pass:
   - The improvement plan
   - The phase/items to apply
   - The target project path

3. **Report results**: Show what was changed:
   ```
   ## Fix Report

   ### Applied
   | # | Change | Status |
   |---|--------|--------|

   ### Backups
   Originals saved to: [path]/.claude-audit-backup/[timestamp]/

   ### Verification
   [Re-scan summary showing improvement]
   ```

4. **Verify**: After applying, run a quick re-scan to confirm improvements.

## Important
- ALWAYS show the plan and ask for confirmation before modifying anything
- Create backups of all files before modification
- Apply one phase at a time — don't rush through all phases
- If a change fails, stop and report — don't continue blindly
- The rewriter agent handles the actual file modifications
