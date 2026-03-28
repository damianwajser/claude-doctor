# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Identity

**claude-doctor** is a Claude Code meta-project: a suite of specialized agents, skills, and rules designed to audit, optimize, and rewrite the Claude Code configuration of any target project.

It acts as an expert consultant that deeply understands how Claude Code works internally — CLAUDE.md inheritance, settings scopes, skills, agents, hooks, rules, memory, MCP servers, permissions — and applies that knowledge to make target projects optimal for AI-assisted development.

## Core Capabilities

- **Project Scanning**: Discover and map all Claude Code configuration files in a target project (CLAUDE.md, .claude/, settings, agents, skills, hooks, rules, memory)
- **Config Auditing**: Evaluate CLAUDE.md quality, settings correctness, permission safety, and hook effectiveness
- **Multi-Project Awareness**: Detect monorepo structures, validate parent-child inheritance, find lost instructions across project boundaries
- **Skills & Agents Review**: Audit skill definitions for best practices (frontmatter, descriptions, tool restrictions, argument handling)
- **Improvement Plans**: Generate prioritized, actionable improvement plans
- **Auto-Fix**: Rewrite configurations following Anthropic's documented best practices

## Architecture

- **9 agents** in `.claude/agents/`: pipeline of scanner → 6 parallel auditors (config, skills, agents, hooks, mcp, multi-project) → plan-generator → rewriter
- **7 skills** in `.claude/skills/`: `/scan`, `/audit`, `/audit-full`, `/fix`, `/report`, `/compare`, `/init-target` — all run in forked context
- **4 rules** in `.claude/rules/`: 2 path-scoped (agent design, skill design) + 2 global (audit output, target safety)
- **Knowledge base** in `docs/claude-code-reference.md`: best practices from Anthropic docs, referenced by all auditors

## Workflow

1. User provides a target project path (or multiple paths for multi-project setups)
2. `project-scanner` maps the full structure including parent/child relationships
3. Domain-specific auditors run in parallel analyzing their areas
4. `plan-generator` synthesizes findings into a prioritized improvement plan
5. User reviews the plan
6. `rewriter` applies approved changes

## Validation

```bash
jq . .claude/settings.json          # Validate settings JSON
```

## Key Design Decisions

- **Agents are read-only by default** — auditors only use Read, Grep, Glob, Bash. Only `rewriter` can Edit/Write, runs with default permissionMode.
- **Skills run in forked context** — all skills use `context: fork` to prevent context bloat from multi-agent orchestration.
- **Multi-project is first-class** — the scanner always checks for parent directories, sibling projects, and monorepo patterns.
- **Knowledge base in docs/** — agents reference `docs/claude-code-reference.md` for best practices rather than relying on training data alone.
- **Target project path** — always passed via `$ARGUMENTS` to skills, or as the first message in conversation. Never assume the target is this project itself.

## Critical Rules

- When auditing a target project, ALWAYS check parent directories for CLAUDE.md files — instructions cascade down and may be critical context.
- When auditing a monorepo, map ALL subprojects before analyzing any single one — sibling context matters.
- Never modify the target project without explicit user approval — audit first, fix only when asked.
- Reference `docs/claude-code-reference.md` for all best-practice validations — don't rely on assumptions.
- CLAUDE.md files should be under 200 lines — recommend splitting to `.claude/rules/` when they exceed this.
- Always validate that skills have proper `description` fields — Claude uses these to decide when to invoke them.
- Always check for `claudeMdExcludes` that might be hiding important parent instructions.
