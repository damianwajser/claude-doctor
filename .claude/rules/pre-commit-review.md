# Pre-Commit README Review

Before creating any git commit in this project, verify whether the changes require updates to `README.md`.

## Checklist

1. **Commands table**: If a skill was added, removed, or its behavior changed significantly — update the Commands table
2. **Diagrams**: If the pipeline flow, agent architecture, or typical workflow changed — update the Mermaid diagrams
3. **What It Checks table**: If auditors now check new areas or stopped checking something — update the table
4. **Safety section**: If permission model, backup behavior, or sandbox rules changed — update Safety
5. **Monorepo section**: If multi-project detection or inheritance logic changed — update Monorepo Support
6. **Architecture description**: If agents or skills were added/removed — update the "How It Works" intro text

## How to Apply

- Read `README.md` and compare against the changes being committed
- If any section is now outdated, update it before committing
- If changes are purely internal (refactoring, bug fixes, prompt tweaks that don't change external behavior), no README update is needed
- When in doubt, ask the user
