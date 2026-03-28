<!-- Intentionally global: audit output standards apply to all agents and skills, not just specific paths -->

# Audit Output Standards

All audit agents in this project must follow these output conventions:

## Severity Levels
- **CRITICAL**: Must fix — broken functionality, security issues, lost instructions
- **WARNING**: Should fix — suboptimal patterns, missing best practices
- **SUGGESTION**: Nice to have — improvements, missing opportunities

## Finding Format
Every finding must include:
1. **What**: Clear description of the issue
2. **Where**: Exact file path (and line numbers when relevant)
3. **Why**: Impact on Claude Code behavior
4. **Fix**: Specific, actionable recommendation

## Scoring
- **A**: No critical issues, ≤2 warnings
- **B**: No critical issues, some warnings
- **C**: 1-2 critical issues OR many warnings
- **D**: Multiple critical issues
- **F**: Missing CLAUDE.md or fundamentally broken

## Language
- Write findings in the same language the user is using
- Technical terms (CLAUDE.md, settings.json, frontmatter) stay in English
- Be direct and specific — no filler text
