---
name: report
description: Generate a markdown report file from the latest audit results. Use when someone says "generate report", "export audit", "save report", or wants a persistent record of the audit findings.
argument-hint: <target-project-path> [--output path/to/report.md]
allowed-tools: Read, Grep, Glob, Bash, Write
disable-model-invocation: true
---

# Generate Audit Report

Export the audit results for the project at `$0` as a standalone markdown file.

## Options
- `--output path/to/report.md`: Custom output path (default: `$0/.claude-audit-report.md`)

## Prerequisites

An audit must have been run in the current conversation. If not, inform the user to run `/audit $0` or `/audit-full $0` first.

## Report Structure

Generate a comprehensive markdown file:

```markdown
# Claude Code Audit Report

**Project**: [name]
**Path**: [absolute path]
**Date**: [current date]
**Auditor**: ProyectCreator v1.0

---

## Executive Summary

| Metric | Value |
|--------|-------|
| Health Score | [A-F] |
| Critical Issues | [N] |
| Warnings | [N] |
| Suggestions | [N] |
| Project Type | [type] |

## Project Structure

[Scanner output — project tree showing Claude Code config files]

## Findings

### Critical Issues
[Each with: description, file, impact, fix]

### Warnings
[Each with: description, file, impact, fix]

### Suggestions
[Each with: description, file, impact, fix]

## Improvement Plan

[Full prioritized plan from plan-generator]

## Appendix

### Files Audited
[Complete list of files reviewed]

### Audit Configuration
- Agents used: [list]
- Reference version: claude-code-reference.md
```

## Output

Write the report to the specified output path (or default). Confirm the path with the user before writing.

```
Report saved to: [path]

You can share this report with your team or use it as a checklist.
```
