# claude-doctor

Your Claude Code projects deserve a checkup.

**claude-doctor** is a specialized toolkit that audits, diagnoses, and fixes the Claude Code configuration of any project. Think of it as a senior consultant that knows every detail of CLAUDE.md inheritance, settings scopes, skills, agents, hooks, rules, memory, MCP servers, and permissions — and uses that knowledge to make your projects work better with Claude Code.

Whether you have a single project with a basic CLAUDE.md or a complex monorepo with dozens of subprojects, claude-doctor finds what's missing, what's broken, and what could be better.

## Why?

Most Claude Code projects leave performance on the table:

- CLAUDE.md files that are too long, too vague, or missing entirely
- Skills without descriptions (Claude can't auto-detect when to use them)
- Agents with unrestricted tool access (violating least privilege)
- Monorepo subprojects that lose parent instructions without anyone noticing
- Missing deny rules for destructive operations
- Hooks that try to block on events that don't support blocking

claude-doctor catches all of this — and generates a prioritized plan to fix it.

## Quick Start

```bash
git clone https://github.com/damianwajser/claude-doctor.git
cd claude-doctor
claude
```

Then point it at any project:

```
/audit /path/to/your/project
```

## Commands

| Command | What it does |
|---------|-------------|
| `/scan <path>` | Map all Claude Code config files — just the facts, no judgment |
| `/audit <path>` | Quick checkup — health score (A-F) and top 5 issues |
| `/audit-full <path>` | Full examination — 6 auditors in parallel + prioritized fix plan |
| `/fix <path> [--phase N]` | Apply fixes phase by phase, with confirmation and backups |
| `/report <path>` | Export findings as a shareable markdown report |
| `/compare <path-A> <path-B>` | Side-by-side diff of two project configurations |
| `/init-target <path>` | Set up optimal Claude Code config from scratch |

## What It Checks

| Area | Examples |
|------|----------|
| **CLAUDE.md** | Size (<200 lines), structure, build/test commands, generic advice, contradictions |
| **Settings** | Deny-first permissions, sandbox, secrets in config, `claudeMdExcludes` accidents |
| **Skills** | Missing descriptions, unrestricted tools, missing `context: fork`, `$ARGUMENTS` usage |
| **Agents** | Tool access (least privilege), model selection, `maxTurns`, single responsibility |
| **Hooks** | Event types vs blocking capability, exit codes, timeouts, infinite loop risks |
| **MCP Servers** | Hardcoded secrets, `alwaysAllow` scope, localhost-only validation |
| **Rules** | Path pattern accuracy, frontmatter, cross-file conflicts |
| **Monorepo** | Parent-child inheritance, lost context, duplicated config, orphaned subprojects |

## How It Works

### Full Audit Pipeline

```mermaid
graph TD
    USER["/audit-full path"] --> SCANNER["project-scanner"]

    SCANNER --> CONFIG["claude-config-auditor"]
    SCANNER --> SKILLS["skills-auditor"]
    SCANNER --> AGENTS["agents-auditor"]
    SCANNER --> HOOKS["hooks-auditor"]
    SCANNER --> MCP["mcp-auditor"]
    SCANNER --> MULTI["multi-project-auditor"]

    CONFIG --> PLAN["plan-generator"]
    SKILLS --> PLAN
    AGENTS --> PLAN
    HOOKS --> PLAN
    MCP --> PLAN
    MULTI --> PLAN

    PLAN --> REVIEW{{"User reviews plan"}}
    REVIEW -->|approved| REWRITER["rewriter"]
    REWRITER --> DONE["Fixed project"]

    style USER fill:#4A90D9,color:#fff
    style SCANNER fill:#7B68EE,color:#fff
    style CONFIG fill:#E67E22,color:#fff
    style SKILLS fill:#E67E22,color:#fff
    style AGENTS fill:#E67E22,color:#fff
    style HOOKS fill:#E67E22,color:#fff
    style MCP fill:#E67E22,color:#fff
    style MULTI fill:#E67E22,color:#fff
    style PLAN fill:#2ECC71,color:#fff
    style REVIEW fill:#F39C12,color:#fff
    style REWRITER fill:#E74C3C,color:#fff
    style DONE fill:#27AE60,color:#fff
```

9 specialized agents, each focused on one area. Auditors are read-only (orange). Only the rewriter (red) can modify files — and only after you approve.

### Typical Workflow

```mermaid
graph LR
    SCAN["/scan"] --> AUDIT["/audit"]
    AUDIT --> FULL["/audit-full"]
    FULL --> FIX["/fix"]
    FIX --> VERIFY["/audit"]
    VERIFY --> REPORT["/report"]

    SCAN -.- S1["See what exists"]
    AUDIT -.- S2["Quick health check"]
    FULL -.- S3["Deep analysis + plan"]
    FIX -.- S4["Apply phase by phase"]
    VERIFY -.- S5["Confirm improvements"]
    REPORT -.- S6["Share with team"]

    style SCAN fill:#3498DB,color:#fff
    style AUDIT fill:#3498DB,color:#fff
    style FULL fill:#8E44AD,color:#fff
    style FIX fill:#E74C3C,color:#fff
    style VERIFY fill:#2ECC71,color:#fff
    style REPORT fill:#2ECC71,color:#fff
    style S1 fill:none,stroke:none,color:#888
    style S2 fill:none,stroke:none,color:#888
    style S3 fill:none,stroke:none,color:#888
    style S4 fill:none,stroke:none,color:#888
    style S5 fill:none,stroke:none,color:#888
    style S6 fill:none,stroke:none,color:#888
```

### Monorepo Scanning

```mermaid
graph TD
    START["project-scanner receives path"] --> UP["Walk UP: find parent projects"]
    UP --> ROOT["Identify ecosystem root"]
    ROOT --> DOWN["Walk DOWN: map all subprojects"]
    DOWN --> SIBLINGS["Map sibling projects"]

    SIBLINGS --> CHAIN["Build instruction inheritance chain"]

    CHAIN --> CHECK1["Check: parent CLAUDE.md inherited?"]
    CHAIN --> CHECK2["Check: claudeMdExcludes hiding rules?"]
    CHAIN --> CHECK3["Check: siblings duplicating config?"]
    CHAIN --> CHECK4["Check: child contradicts parent?"]

    CHECK1 --> REPORT["Ecosystem map + findings"]
    CHECK2 --> REPORT
    CHECK3 --> REPORT
    CHECK4 --> REPORT

    style START fill:#4A90D9,color:#fff
    style UP fill:#7B68EE,color:#fff
    style ROOT fill:#7B68EE,color:#fff
    style DOWN fill:#7B68EE,color:#fff
    style SIBLINGS fill:#7B68EE,color:#fff
    style CHAIN fill:#E67E22,color:#fff
    style CHECK1 fill:#E74C3C,color:#fff
    style CHECK2 fill:#E74C3C,color:#fff
    style CHECK3 fill:#E74C3C,color:#fff
    style CHECK4 fill:#E74C3C,color:#fff
    style REPORT fill:#2ECC71,color:#fff
```

## Monorepo Support

The `multi-project-auditor` specializes in the hardest problem — **instruction inheritance across project boundaries**:

- Walks UP to find parent CLAUDE.md files that cascade down
- Walks DOWN to map every subproject
- Detects 10 patterns of context loss (orphaned children, conflicting instructions, `claudeMdExcludes` hiding parent rules, siblings reinventing the same config)
- Always recommends root-level solutions over per-child duplication

```bash
/audit-full /path/to/monorepo/packages/api --include-siblings
```

## Safety

- **Read-only by default** — auditors can only read, never write
- **Single writer** — only the `rewriter` agent can modify files, and only after your explicit approval
- **Automatic backups** — originals saved to `.claude-audit-backup/` before any change
- **Sandboxed** — filesystem sandbox enabled
- **Deny-first** — destructive operations (`rm -rf`, `sudo`, `git push`, `git reset --hard`) are blocked at the permission level

## Requirements

- [Claude Code](https://claude.ai/code) installed and authenticated
- `jq` (for JSON validation hooks)

## License

MIT
