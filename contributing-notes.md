# Contributing Notes

Quick reference for branch naming, PR rules, and code standards.

## Branches

| Branch | Purpose | Deploys To |
|--------|---------|------------|
| `main` | Production | App Engine default |
| `test` | Sandbox | App Engine test |

Always branch from `test`. Never push directly to `main` or `test`.

## Branch Naming

```
feature/AW-XXX-short-description   → New feature
fix/GRBT-XXX-short-description     → Bug fix
ai/AW-XXX-short-description        → AI-assisted changes
```

Always include the Jira ticket number.

## Commit Messages

Format: `TICKET: What this does`

```bash
# Good
git commit -m "GRBT-155: Set default warehouse for resell stock adjustments"

# Bad
git commit -m "fix stuff"
```

## PR Requirements (Every PR Must Have)

1. Jira ticket link
2. What changed (plain English)
3. Why it changed
4. How to test (exact steps)
5. Risk level (low/medium/high)
6. Checklist completed

## What Gets PR Rejected

- AI-generated description with no human context
- Missing Jira ticket
- No testing evidence
- Committing secrets
- `any` types without justification
- SQL string interpolation

## Code Standards

- **TypeScript**: Strict mode, no `any`, define types in `types/`
- **Database**: Always use `executeWithRetry()` + `?` placeholders
- **Components**: Use shadcn/ui, one responsibility per component
- **Security**: Never commit secrets, never build SQL with string concat

## AI Usage Rules

- You must understand every line you commit
- You must review AI code for security
- You must write PR description yourself
- You must test AI-generated code
