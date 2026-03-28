# Claude Code Best Practices Reference

This document is the authoritative knowledge base for ProyectCreator agents. All audit validations reference rules defined here.

---

## 1. CLAUDE.md Files

### Loading Behavior
- **Upward traversal**: Claude walks UP from cwd, loading all ancestor CLAUDE.md files at session start
- **Downward (on-demand)**: Subdirectory CLAUDE.md files load only when Claude accesses files in those dirs
- **Locations**: `./CLAUDE.md`, `./.claude/CLAUDE.md`, `~/.claude/CLAUDE.md`, managed policy paths
- **Survives compaction**: CLAUDE.md is fully re-injected after `/compact`

### Quality Checklist
- [ ] Under 200 lines (longer = worse adherence and more context consumed)
- [ ] Uses markdown headers and bullets for scannability
- [ ] Instructions are specific and verifiable ("Use 2-space indentation" not "Format properly")
- [ ] No contradictory instructions across files
- [ ] No generic advice that Claude already knows (don't say "write tests" or "handle errors")
- [ ] Build/test/lint commands are documented
- [ ] Architecture section focuses on non-obvious cross-file relationships
- [ ] Uses `@path/to/file` imports for large reference documents (max 5 levels deep)
- [ ] HTML block comments (`<!-- -->`) used for maintainer notes (stripped before injection)
- [ ] No secrets, API keys, or tokens

### Common Anti-Patterns
- Duplicating README.md content verbatim
- Listing every file in the project (discoverable via tools)
- Including generic development advice ("use meaningful variable names")
- Having a single 500+ line CLAUDE.md instead of splitting into rules/
- Contradicting parent CLAUDE.md instructions without awareness

---

## 2. .claude/rules/ Directory

### How Rules Work
- Files without `paths` frontmatter: loaded at session start (like CLAUDE.md)
- Files with `paths` frontmatter: loaded on-demand when matching files are accessed
- Support symlinks for sharing across projects
- User-level rules: `~/.claude/rules/` (lower priority than project rules)

### Path-Scoped Rule Format
```yaml
---
paths:
  - "src/api/**/*.ts"
  - "lib/**/*.ts"
---
# Rule content here
```

### Quality Checklist
- [ ] Each rule file has a single, clear topic
- [ ] Path patterns use proper glob syntax
- [ ] No overlapping/conflicting rules across files
- [ ] Path patterns actually match existing files in the project
- [ ] Rules are short and focused (not mini-CLAUDE.md files)

---

## 3. Settings Configuration

### Scope Precedence (highest to lowest)
1. Managed (organization-wide, cannot override)
2. CLI arguments (session-specific)
3. Local (`.claude/settings.local.json` — gitignored)
4. Project (`.claude/settings.json` — committed)
5. User (`~/.claude/settings.json` — personal)

### Key Settings
- `permissions.allow/deny/ask`: Tool access control (deny always wins)
- `sandbox.enabled`: Filesystem/network sandboxing
- `env`: Environment variables (use `$VAR` to reference shell env)
- `claudeMdExcludes`: Glob patterns to skip CLAUDE.md files
- `hooks`: Lifecycle automation
- `autoMemoryEnabled`: Auto memory toggle (default: true)

### Quality Checklist
- [ ] `settings.json` exists and is valid JSON
- [ ] Permissions use deny-first approach for dangerous operations
- [ ] `settings.local.json` exists for local overrides (gitignored)
- [ ] No secrets in `settings.json` (use env vars or local settings)
- [ ] `claudeMdExcludes` is intentional (not accidentally hiding parent instructions)
- [ ] Sandbox enabled for projects that don't need full filesystem access

---

## 4. Skills (Slash Commands)

### File Structure
```
.claude/skills/<skill-name>/
├── SKILL.md           # Required: frontmatter + instructions
├── templates/         # Optional: output templates
├── examples/          # Optional: example outputs
└── scripts/           # Optional: helper scripts
```

### Frontmatter Reference
| Field | Required | Purpose |
|-------|----------|---------|
| `name` | No (derived from dir) | Slash command identifier |
| `description` | Recommended | When Claude should auto-invoke |
| `disable-model-invocation` | No | true = only user can invoke |
| `user-invocable` | No | false = only Claude can invoke |
| `allowed-tools` | No | Tool restrictions when active |
| `model` | No | Model override |
| `effort` | No | Effort level override |
| `context` | No | `fork` = run in subagent |
| `agent` | No | Subagent type when context: fork |
| `argument-hint` | No | Autocomplete help text |
| `paths` | No | Glob patterns for auto-activation |

### Quality Checklist
- [ ] Every skill has a clear, keyword-rich `description`
- [ ] Skills with side effects use `disable-model-invocation: true`
- [ ] Background knowledge skills use `user-invocable: false`
- [ ] `allowed-tools` restricts access appropriately (read-only when possible)
- [ ] Instructions are directive/imperative, not conversational
- [ ] Output format is explicitly defined
- [ ] Under 500 lines per SKILL.md
- [ ] Supporting files referenced from SKILL.md
- [ ] `$ARGUMENTS` used for parameterized skills
- [ ] Dynamic context (`!`command``) used where live data is needed

### Common Anti-Patterns
- Missing `description` (Claude can't auto-detect when to use the skill)
- No tool restrictions on dangerous skills (deploy, commit)
- Overly broad descriptions that trigger on unrelated requests
- Not using `context: fork` for heavy operations (bloats main context)

---

## 5. Agents (Subagents)

### File Location
```
.claude/agents/<agent-name>.md   # Project-level
~/.claude/agents/<agent-name>.md # User-level
```

### Frontmatter Reference
| Field | Required | Purpose |
|-------|----------|---------|
| `name` | Yes | Unique identifier (lowercase + hyphens) |
| `description` | Yes | When to delegate (Claude reads this) |
| `model` | No | haiku/sonnet/opus/inherit |
| `tools` | No | Comma-separated allowlist |
| `disallowedTools` | No | Tools to deny |
| `maxTurns` | No | Max agentic turns |
| `memory` | No | user/project/local |
| `isolation` | No | `worktree` for isolated checkout |
| `effort` | No | low/medium/high/max |
| `skills` | No | Skills to inject into context |
| `mcpServers` | No | MCP servers scoped to agent |
| `hooks` | No | Hooks scoped to agent lifecycle |
| `background` | No | Auto-run as background task |
| `permissionMode` | No | Permission override |

### Quality Checklist
- [ ] `name` uses lowercase + hyphens
- [ ] `description` is specific enough for accurate auto-delegation
- [ ] `tools` restricted to minimum necessary (principle of least privilege)
- [ ] Read-only agents don't have Write/Edit
- [ ] `model` chosen appropriately (haiku for simple, sonnet for balanced, opus for complex)
- [ ] `maxTurns` set to prevent runaway agents
- [ ] System prompt (body) is focused on a single responsibility
- [ ] Not trying to spawn sub-subagents (max depth = 1)

### Common Anti-Patterns
- Overly broad tool access (giving all tools when only Read/Grep needed)
- Vague descriptions ("general purpose agent")
- No maxTurns limit (can run indefinitely)
- Duplicate functionality across agents
- System prompt too long or unfocused

---

## 6. Hooks

### Event Types
| Event | Fires When | Can Block? |
|-------|-----------|-----------|
| SessionStart | Session begins/resumes | Yes |
| UserPromptSubmit | User submits prompt | Yes |
| PreToolUse | Before tool executes | Yes (exit 2) |
| PostToolUse | After tool succeeds | No |
| PostToolUseFailure | After tool fails | No |
| Stop | Claude finishes responding | Yes |
| Notification | Claude sends notification | No |
| SubagentStart/Stop | Subagent lifecycle | No |
| FileChanged | Watched file changes | No |
| CwdChanged | Working directory changes | No |
| InstructionsLoaded | CLAUDE.md loaded | Yes |
| ConfigChange | Config file changes | Yes |

### Hook Types
- `command`: Shell script (most common)
- `http`: POST to endpoint
- `prompt`: Single LLM evaluation
- `agent`: Full subagent verification

### Quality Checklist
- [ ] Matchers narrow scope (don't fire on every tool use)
- [ ] Blocking hooks (exit 2) have clear stderr messages
- [ ] Scripts are executable (`chmod +x`)
- [ ] No infinite loops in Stop hooks (check `stop_hook_active`)
- [ ] Timeout values set for long-running hooks
- [ ] JSON input parsed correctly (use `jq`)

---

## 7. Memory System

### Auto Memory
- Location: `~/.claude/projects/<project>/memory/`
- `MEMORY.md`: Index file, first 200 lines / 25KB loaded at session start
- Topic files: loaded on-demand
- Survives `/compact`

### Quality Checklist
- [ ] MEMORY.md under 200 lines
- [ ] Each entry is one line, under 150 characters
- [ ] Topic files have proper frontmatter (name, description, type)
- [ ] No duplicate memories
- [ ] No stale/outdated memories
- [ ] Memory types used correctly (user, feedback, project, reference)
- [ ] No code patterns or architecture stored (derivable from code)
- [ ] No git history stored (use `git log`)

---

## 8. MCP Servers

### Configuration Files
- `.mcp.json` in cwd (highest priority)
- `.claude/.mcp.json` (project level)
- `~/.claude/.mcp.json` (user level)

### Quality Checklist
- [ ] Server definitions are valid JSON
- [ ] Environment variables use `$VAR` syntax (not hardcoded secrets)
- [ ] `alwaysAllow` only set for safe, read-only tools
- [ ] Local overrides in `.mcp.local.json` (gitignored)
- [ ] No duplicate server definitions across scopes

---

## 9. Multi-Project / Monorepo Specifics

### Inheritance Rules
- CLAUDE.md: walks UP directory tree automatically
- Settings: does NOT traverse parent directories (known limitation)
- Skills/agents: discovered in .claude/ of current project only
- Hooks: defined per project, not inherited from parent

### Quality Checklist
- [ ] Root CLAUDE.md exists with shared conventions
- [ ] Each subproject has its own CLAUDE.md for build/test commands
- [ ] Path-specific rules scope instructions to relevant packages
- [ ] `claudeMdExcludes` is NOT accidentally hiding parent instructions
- [ ] Shared rules use symlinks where appropriate
- [ ] Parent CLAUDE.md references ecosystem-wide decisions
- [ ] Subproject CLAUDE.md files acknowledge they're part of a larger system
- [ ] Root-level agents available for cross-project tasks
- [ ] No conflicting instructions between parent and child CLAUDE.md files
- [ ] Build commands documented per subproject (not just root)

### Common Anti-Patterns
- Subproject ignoring parent CLAUDE.md instructions
- Duplicating shared rules in every subproject
- No root CLAUDE.md in a monorepo
- Using `claudeMdExcludes` without understanding what's being excluded
- Assuming agents/skills from parent are available in child context
- No documentation of cross-project dependencies

---

## 10. Permission Patterns

### Bash Permission Syntax
```
Bash(npm *)         — any npm command
Bash(git commit *)  — git commit with any args
Read(/secrets/**)   — read files in secrets/
Edit(/src/**/*.ts)  — edit TypeScript in src/
```

### Recommended Deny Rules
```json
{
  "permissions": {
    "deny": [
      "Bash(rm -rf *)",
      "Bash(sudo *)",
      "Edit(.git/**)",
      "Edit(node_modules/**)"
    ]
  }
}
```

### Quality Checklist
- [ ] Deny rules exist for destructive operations
- [ ] Allow rules use minimum necessary scope
- [ ] No overly permissive rules (`Bash(*)` with no deny)
- [ ] MCP tools have appropriate allow/deny
- [ ] Agent tool restricted to known agent types where needed
