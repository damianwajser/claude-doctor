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
| `/audit-full <path>` | Full examination — 6 auditors in parallel + prioritized fix plan. Offers to apply Phase 1 fixes inline |
| `/fix <path> [--phase N]` | Apply fixes phase by phase — reuses persisted audit results, with confirmation and backups |
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
flowchart TD
    USER(["fa:fa-terminal /audit-full path"]):::cmd --> SCANNER

    subgraph SCAN ["Phase 1 — Discovery"]
        SCANNER["fa:fa-search project-scanner"]:::scanner
    end

    SCANNER --> CONFIG & SKILLS & AGENTS & HOOKS & MCP & MULTI

    subgraph AUDIT ["Phase 2 — Parallel Auditors (read-only)"]
        CONFIG["fa:fa-file-alt config"]:::auditor
        SKILLS["fa:fa-bolt skills"]:::auditor
        AGENTS["fa:fa-robot agents"]:::auditor
        HOOKS["fa:fa-link hooks"]:::auditor
        MCP["fa:fa-server mcp"]:::auditor
        MULTI["fa:fa-sitemap multi-project"]:::auditor
    end

    CONFIG & SKILLS & AGENTS & HOOKS & MCP & MULTI --> PLAN

    subgraph SYNTH ["Phase 3 — Synthesis"]
        PLAN["fa:fa-list-ol plan-generator"]:::plan
    end

    PLAN --> PERSIST["fa:fa-save Persist report"]:::plan
    PERSIST --> REVIEW{{"fa:fa-user User reviews plan"}}:::review
    REVIEW -- "apply now" --> REWRITER
    REVIEW -- "later → /fix" --> FIX_LATER(["fa:fa-clock Reads persisted report"]):::cmd

    subgraph APPLY ["Phase 4 — Apply"]
        REWRITER["fa:fa-wrench rewriter"]:::writer
    end

    FIX_LATER --> REWRITER
    REWRITER --> DONE(["fa:fa-check-circle Optimized project"]):::done

    classDef cmd fill:#1a1a2e,stroke:#4A90D9,stroke-width:2px,color:#4A90D9
    classDef scanner fill:#2d2b55,stroke:#7B68EE,stroke-width:2px,color:#e0e0ff
    classDef auditor fill:#2c2417,stroke:#E67E22,stroke-width:2px,color:#f5c77e
    classDef plan fill:#1a2e1a,stroke:#2ECC71,stroke-width:2px,color:#8ef5b0
    classDef review fill:#2e2a17,stroke:#F39C12,stroke-width:2px,color:#f5d78e
    classDef writer fill:#2e1a1a,stroke:#E74C3C,stroke-width:2px,color:#f5a0a0
    classDef done fill:#1a2e1a,stroke:#27AE60,stroke-width:2px,color:#8ef5b0

    style SCAN fill:none,stroke:#7B68EE,stroke-width:1px,stroke-dasharray:5 5,color:#7B68EE
    style AUDIT fill:none,stroke:#E67E22,stroke-width:1px,stroke-dasharray:5 5,color:#E67E22
    style SYNTH fill:none,stroke:#2ECC71,stroke-width:1px,stroke-dasharray:5 5,color:#2ECC71
    style APPLY fill:none,stroke:#E74C3C,stroke-width:1px,stroke-dasharray:5 5,color:#E74C3C
```

9 specialized agents, each focused on one area. Auditors are **read-only**. Only the rewriter can modify files — and only after you approve.

### Typical Workflow

```mermaid
flowchart LR
    SCAN(["fa:fa-binoculars /scan"]):::blue
    AUDIT(["fa:fa-stethoscope /audit"]):::blue
    FULL(["fa:fa-microscope /audit-full"]):::purple
    FIX(["fa:fa-wrench /fix"]):::red
    VERIFY(["fa:fa-check /audit"]):::green
    REPORT(["fa:fa-file-medical /report"]):::green

    SCAN -- "what exists?" --> AUDIT
    AUDIT -- "quick score" --> FULL
    FULL -- "apply now?" --> APPLY_NOW{{"inline fix"}}:::red
    FULL -- "later" --> FIX
    APPLY_NOW --> VERIFY
    FIX -- "reads saved report" --> VERIFY
    VERIFY -- "confirmed" --> REPORT

    classDef blue fill:#1a1a2e,stroke:#3498DB,stroke-width:2px,color:#7ec8f5
    classDef purple fill:#1f1a2e,stroke:#8E44AD,stroke-width:2px,color:#c49ef5
    classDef red fill:#2e1a1a,stroke:#E74C3C,stroke-width:2px,color:#f5a0a0
    classDef green fill:#1a2e1a,stroke:#2ECC71,stroke-width:2px,color:#8ef5b0
```

### Monorepo Inheritance Analysis

```mermaid
flowchart TD
    START(["fa:fa-folder-open Receive target path"]):::cmd

    subgraph DISCOVER ["Discovery"]
        UP["fa:fa-arrow-up Walk UP\nfind parent projects"]:::scan
        ROOT["fa:fa-crown Identify\necosystem root"]:::scan
        DOWN["fa:fa-arrow-down Walk DOWN\nmap subprojects"]:::scan
        SIBLINGS["fa:fa-arrows-alt-h Map\nsiblings"]:::scan
    end

    START --> UP --> ROOT --> DOWN --> SIBLINGS

    SIBLINGS --> CHAIN

    subgraph ANALYZE ["Inheritance Analysis"]
        CHAIN["fa:fa-link Build instruction\ninheritance chain"]:::analyze
    end

    CHAIN --> CHECK1 & CHECK2 & CHECK3 & CHECK4

    subgraph CHECKS ["Context Loss Detection"]
        CHECK1["fa:fa-eye-slash Parent CLAUDE.md\nnot inherited?"]:::check
        CHECK2["fa:fa-ban claudeMdExcludes\nhiding rules?"]:::check
        CHECK3["fa:fa-copy Siblings\nduplicating config?"]:::check
        CHECK4["fa:fa-exclamation-triangle Child contradicts\nparent?"]:::check
    end

    CHECK1 & CHECK2 & CHECK3 & CHECK4 --> REPORT

    REPORT(["fa:fa-map Ecosystem map\n+ findings"]):::done

    classDef cmd fill:#1a1a2e,stroke:#4A90D9,stroke-width:2px,color:#7ec8f5
    classDef scan fill:#2d2b55,stroke:#7B68EE,stroke-width:2px,color:#e0e0ff
    classDef analyze fill:#2c2417,stroke:#E67E22,stroke-width:2px,color:#f5c77e
    classDef check fill:#2e1a1a,stroke:#E74C3C,stroke-width:2px,color:#f5a0a0
    classDef done fill:#1a2e1a,stroke:#27AE60,stroke-width:2px,color:#8ef5b0

    style DISCOVER fill:none,stroke:#7B68EE,stroke-width:1px,stroke-dasharray:5 5,color:#7B68EE
    style ANALYZE fill:none,stroke:#E67E22,stroke-width:1px,stroke-dasharray:5 5,color:#E67E22
    style CHECKS fill:none,stroke:#E74C3C,stroke-width:1px,stroke-dasharray:5 5,color:#E74C3C
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
