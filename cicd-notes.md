# CI/CD Pipeline Notes

How the 7-agent swarm pipeline works. You don't deploy manually — the pipeline handles it.

## Pipeline Flow

```
Your PR → Agent 1 (Code Review) → Agent 2 (Arch Review) → Agent 3 (QA/Tests)
       → Agent 4 (Auto-merge) → Agent 5 (Deploy) → Agent 6 (Canary) → Agent 7 (Verify)
```

## What Each Agent Does

| Agent | Job | Triggers On |
|-------|-----|-------------|
| 1 - Code Review | Checks correctness, security, performance, types, duplication | Every PR |
| 2 - Arch Review | Checks architecture patterns, file placement, API naming | New files, API changes, >300 lines, dependency changes |
| 3 - QA & Tests | Runs tests, checks 60% coverage threshold | After Agents 1 & 2 pass |
| 4 - Merge | Auto squash-merges the PR | After Agents 1 & 3 pass |
| 5 - Deploy | Builds and deploys to App Engine | After merge |
| 6 - Canary | Progressive rollout: 5% → 25% → 50% → 100% | After deploy to main |
| 7 - Verify | Smoke tests every 15 min | After canary completes |

## Key Rules

- **60% test coverage** required (enforced by Agent 3)
- Coverage can never decrease (ratchet rule)
- Auto-rollback if error rate > 5% during canary
- Override labels: `ai-review-override`, `ai-arch-override` (need Agnes approval)

## Your Job vs Pipeline's Job

| You | Pipeline |
|-----|----------|
| Write correct code | Review it |
| Write tests | Run them + check coverage |
| Create PR with good description | Merge when checks pass |
| Fix flagged issues | Deploy to test & production |
| Verify on test env | Canary rollout + health monitoring |
| Update Jira ticket | Post-deploy verification |

## How to Know Code is Live

After merge to `main` → Agent 6 does canary rollout → Agent 7 verifies.
Check production: https://employee.awesomeree.com.my
