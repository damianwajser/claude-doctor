---
name: compare
description: Compare the Claude Code configuration of two projects side-by-side. Use when someone says "compare projects", "diff configs", "compare structures", or wants to see differences between two project setups.
argument-hint: <project-path-1> <project-path-2>
allowed-tools: Read, Grep, Glob, Agent
disable-model-invocation: true
context: fork
---

# Compare Projects

Compare the Claude Code configuration of two projects side-by-side.

## Input Parsing
- `$ARGUMENTS` should contain two paths separated by space
- If only one path provided, ask the user for the second path

## Steps

1. **Scan both projects**: Use the `project-scanner` agent for each project in parallel. Pass each path as a separate agent invocation.

2. **Build comparison matrix**:

```
## Configuration Comparison

### Overview
| Aspect | Project A | Project B |
|--------|-----------|-----------|
| Path | ... | ... |
| Type | single/monorepo | single/monorepo |
| CLAUDE.md | [lines] | [lines] |
| Agents | [count] | [count] |
| Skills | [count] | [count] |
| Rules | [count] | [count] |
| Hooks | [count] | [count] |
| MCP servers | [count] | [count] |
| Settings | exists/missing | exists/missing |

### CLAUDE.md Comparison
| Topic | Project A | Project B |
|-------|-----------|-----------|
| Build commands | documented/missing | documented/missing |
| Architecture | documented/missing | documented/missing |
| Line count | [N] | [N] |
| Uses imports | yes/no | yes/no |

### Agents Comparison
| Agent concept | Project A | Project B |
|--------------|-----------|-----------|
| [role] | agent-name / missing | agent-name / missing |

### Skills Comparison
| Skill concept | Project A | Project B |
|--------------|-----------|-----------|
| [purpose] | /skill-name / missing | /skill-name / missing |

### Configuration Differences
[Key differences in settings, permissions, hooks]

### What Project A has that B doesn't
- ...

### What Project B has that A doesn't
- ...

### Shared patterns
- ...

### Recommendations
- For Project A: [what to adopt from B]
- For Project B: [what to adopt from A]
```

## Important
- This is READ-ONLY — no modifications to either project
- Focus on meaningful differences, not trivial ones
- Highlight what each project could learn from the other
- If projects are parent-child, note the inheritance relationship
