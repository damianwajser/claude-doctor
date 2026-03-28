---
name: init-target
description: Bootstrap an optimal Claude Code configuration from scratch for a target project. Use when someone says "initialize project", "bootstrap config", "setup claude code", "create config from scratch", or wants to set up Claude Code in a project that has none.
argument-hint: <target-project-path>
allowed-tools: Read, Grep, Glob, Agent
disable-model-invocation: true
context: fork
effort: high
---

# Bootstrap Claude Code Configuration

Create an optimal Claude Code setup for the project at `$ARGUMENTS`.

## Steps

### Phase 1: Understand the project

1. **Scan current state**: Use the `project-scanner` agent to map what exists (if anything).

2. **Detect project characteristics**:
   - Language/framework (read package.json, go.mod, pom.xml, Cargo.toml, pyproject.toml, etc.)
   - Build system (npm, gradle, maven, cargo, make, etc.)
   - Test framework (jest, pytest, go test, junit, etc.)
   - Linting tools (eslint, prettier, black, golint, etc.)
   - Is it a monorepo? (workspaces, multiple build files)
   - Is it a child of a larger project? (parent CLAUDE.md, parent .claude/)
   - Existing README.md for project description
   - Existing CI/CD for build/test commands

3. **Check for other AI tool configs**: Read `.cursorrules`, `.cursor/rules/`, `.github/copilot-instructions.md` — extract relevant instructions to incorporate.

### Phase 2: Generate the plan

Present a plan to the user before creating anything:

```
## Proposed Claude Code Setup for [project-name]

### Files to create:
| File | Purpose |
|------|---------|
| CLAUDE.md | Project instructions ([estimated] lines) |
| .claude/settings.json | Permissions and hooks |
| .claude/rules/[name].md | [description] |
| .claude/agents/[name].md | [description] — if beneficial |
| .claude/skills/[name]/SKILL.md | [description] — if beneficial |

### CLAUDE.md will include:
- Build: `[command]`
- Test: `[command]` / Single test: `[command]`
- Lint: `[command]`
- Architecture: [key points]
- Conventions: [detected patterns]

### Settings will include:
- Deny rules: [list]
- Sandbox: [enabled/disabled based on project needs]

### Agents recommended: [list with reasons]
### Skills recommended: [list with reasons]

Proceed? (y/n)
```

### Phase 3: Apply (after approval)

Use the `rewriter` agent to create all approved files. Pass the complete plan as context.

### Phase 4: Verify

Run the `project-scanner` agent again to confirm the setup is complete and correct.

Present the final state:
```
## Setup Complete

### Created:
| File | Lines | Status |
|------|-------|--------|

### Verification
- CLAUDE.md: [score]
- Settings: [score]

### Next steps:
- Review and customize the generated files
- Run `/audit $ARGUMENTS` to validate the setup
- Add project-specific agents and skills as needed
```

## Guidelines for generated config

### CLAUDE.md
- Under 200 lines
- Build/test/lint commands from detected tooling
- Architecture section only if project has non-obvious structure
- No generic advice
- No README.md duplication
- Acknowledge parent project if child of monorepo

### Settings
- Always include deny rules for destructive operations
- Enable sandbox for projects that don't need full filesystem access
- Add `.gitignore` entries for `settings.local.json` and `.mcp.local.json`

### Agents — only create if the project benefits
- Code reviewer (for projects with >10 source files)
- Test runner (for projects with test suites)
- Don't create agents "just because" — each must have clear value

### Skills — only create if the project benefits
- Don't create skills for standard operations Claude already handles well
- Focus on project-specific workflows

## Important
- ALWAYS show the plan and get approval before creating files
- Never overwrite existing Claude Code configuration without explicit approval
- If existing config found, recommend `/audit` instead of `/init-target`
- Merge insights from .cursorrules and copilot instructions into CLAUDE.md
