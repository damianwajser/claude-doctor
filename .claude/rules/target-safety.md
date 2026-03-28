# Target Project Safety Rules

These rules apply whenever working with a target project (the project being audited):

## Read-Only by Default
- Audit and analysis operations NEVER modify target files
- Only the `rewriter` agent (invoked through `/fix`) can modify files
- Always get explicit user approval before any modification

## Backup Protocol
- Before modifying ANY file: copy original to `.claude-audit-backup/[timestamp]/`
- Backup path preserves original directory structure
- Never delete originals — move to backup

## Scope Control
- Never modify `.git/` directories
- Never modify `node_modules/`, `vendor/`, `target/`, `dist/`, `build/`
- Never modify files outside the approved scope (the plan)
- Never add extra "improvements" beyond what was approved

## Multi-Project Safety
- When auditing a monorepo, be explicit about which project you're analyzing
- Never modify a parent project when the user only approved changes to a child
- Never modify sibling projects unless explicitly approved
