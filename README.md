# claude-doctor

A Claude Code meta-project that audits, optimizes, and rewrites the Claude Code configuration of any target project.

It acts as an expert consultant that deeply understands how Claude Code works internally — CLAUDE.md inheritance, settings scopes, skills, agents, hooks, rules, memory, MCP servers, permissions — and applies that knowledge to make target projects optimal for AI-assisted development.

## Requirements

- [Claude Code](https://claude.ai/code) installed and authenticated
- `jq` installed (for JSON validation hooks)

## Quick Start

```bash
git clone https://github.com/damianwajser/claude-doctor.git
cd claude-doctor
claude
```

Then use any of the available commands:

```
/scan /path/to/your/project          # Map structure
/audit /path/to/your/project         # Quick audit
/audit-full /path/to/your/project    # Deep audit + improvement plan
/fix /path/to/your/project           # Apply approved improvements
```

## Commands

| Command | Purpose |
|---------|---------|
| `/scan <path>` | Map all Claude Code config files without judging quality |
| `/audit <path>` | Quick audit — health score + top 5 issues |
| `/audit-full <path>` | Deep audit — all auditors in parallel + prioritized improvement plan |
| `/fix <path> [--phase N]` | Apply improvements from the plan, one phase at a time with backups |
| `/report <path>` | Export audit results as a shareable markdown file |
| `/compare <path-A> <path-B>` | Side-by-side comparison of two projects |
| `/init-target <path>` | Bootstrap optimal Claude Code config from scratch |

## What It Audits

- **CLAUDE.md** — structure, size (<200 lines), scannability, specificity, build/test commands, architecture docs
- **Settings** — permissions (deny-first), sandbox, environment variables, `claudeMdExcludes`
- **Skills** — frontmatter completeness, descriptions, tool restrictions, `$ARGUMENTS` usage
- **Agents** — tool access (least privilege), model selection, `maxTurns`, single responsibility
- **Hooks** — event types, matchers, exit codes, timeouts, blocking behavior
- **MCP Servers** — secrets exposure, `alwaysAllow` safety, scope appropriateness
- **Rules** — path patterns, frontmatter, conflicts between files
- **Multi-Project / Monorepo** — parent-child instruction inheritance, lost context, duplicated config across siblings

## Architecture

```
.claude/
├── agents/         # 9 specialized subagents
│   ├── project-scanner         # Maps target project structure
│   ├── claude-config-auditor   # Audits CLAUDE.md, settings, rules
│   ├── skills-auditor          # Audits skill definitions
│   ├── agents-auditor          # Audits agent definitions
│   ├── hooks-auditor           # Audits hooks configuration
│   ├── mcp-auditor             # Audits MCP server configs
│   ├── multi-project-auditor   # Monorepo inheritance analysis
│   ├── plan-generator          # Synthesizes findings into plan
│   └── rewriter                # Applies approved changes (only writer)
├── skills/         # 7 user commands (/audit, /fix, etc.)
├── rules/          # Coding guidelines for this project
└── settings.json   # Permissions, sandbox, hooks
docs/
└── claude-code-reference.md    # Best practices knowledge base
```

### Pipeline

```
/audit-full path
    │
    ├─ project-scanner ──────────────────────────────┐
    │                                                 │
    ├─ claude-config-auditor ─┐                       │
    ├─ skills-auditor ────────┤                       │
    ├─ agents-auditor ────────┼── (parallel) ── plan-generator
    ├─ hooks-auditor ─────────┤
    ├─ mcp-auditor ───────────┤
    └─ multi-project-auditor ─┘
                                                      │
/fix path                                             │
    │                                                 │
    └─ rewriter ──── applies approved changes ────────┘
```

## Typical Workflow

```bash
# 1. See what config exists
/scan /path/to/project

# 2. Quick health check
/audit /path/to/project

# 3. Deep analysis with improvement plan
/audit-full /path/to/project

# 4. Apply fixes (phase by phase, with confirmation)
/fix /path/to/project

# 5. Verify improvements
/audit /path/to/project

# 6. Export report for team
/report /path/to/project
```

## Monorepo Support

The `multi-project-auditor` (model: opus, effort: max) specializes in the hardest problem — **instruction inheritance across project boundaries**:

- Walks UP to find parent CLAUDE.md files
- Walks DOWN to map all subprojects
- Detects 10 patterns of context loss (orphaned subprojects, conflicting instructions, hidden parent rules via `claudeMdExcludes`, etc.)
- Recommends root-level fixes over per-child duplication

```bash
# Audit with sibling projects included
/audit-full /path/to/monorepo/packages/api --include-siblings
```

## Safety

- **Read-only by default** — all auditors can only read, never write
- **Only `rewriter` can modify files** — and only after explicit user approval
- **Backups** — originals saved to `.claude-audit-backup/[timestamp]/` before any change
- **Sandboxed** — filesystem sandbox enabled in settings
- **Deny-first permissions** — destructive operations (`rm -rf`, `sudo`, `git push`, `git reset --hard`) are blocked

## Self-Audited

This project was recursively self-audited to convergence:

| Iteration | Findings | Result |
|-----------|----------|--------|
| 1 | 21 | Fixed |
| 2 | 7 | Fixed |
| 3 | 2 | Fixed |
| 4 | 0 | Clean |

## License

MIT
