# Getting Started Notes

Simplified reference from the official Getting Started guide.

## Setup Steps

1. Clone repo: `git clone https://github.com/it-awesomeree/awesomeree-web-app.git`
2. Install: `npm install`
3. Copy env: `cp .env.example .env.local` (ask Agnes for credentials)
4. Run: `npm run dev` → opens at http://localhost:3000
5. Verify: `npm run lint && npm test && npm run build`

## Project Structure (Key Folders)

```
app/           → Pages + API routes (most work happens here)
  api/         → Backend endpoints (183 routes)
components/    → Reusable React components (shadcn/ui)
lib/           → Server-side code
  db.ts        → MySQL connection pool
  auth.ts      → Auth helpers
  services/    → Business logic layer
hooks/         → React hooks (client-side)
types/         → TypeScript type definitions
services/      → Client-side API service (apiService.ts)
```

## How a Page Works

```
Browser → app/[module]/page.tsx → /api/[endpoint] → lib/services/[service].ts → MySQL → response
```

## Key Files

| File | Purpose |
|------|---------|
| `app/api/*/route.ts` | API endpoints |
| `app/*/page.tsx` | Page components |
| `components/*.tsx` | Reusable UI |
| `lib/db.ts` | DB connection pool |
| `lib/services/*.ts` | Business logic |
| `middleware.ts` | Auth + RBAC |

## PR Workflow (12 Steps)

1. Read Jira ticket fully
2. Understand the affected code
3. Plan your approach
4. Create branch from `test`
5. Write code (follow existing patterns)
6. Test locally (`lint`, `test`, `build`)
7. Commit with meaningful messages
8. Push and create PR → target `test`
9. Respond to agent/reviewer feedback
10. Auto-merge by swarm (Agent 4)
11. Verify on test environment
12. Update Jira → move to TO REVIEW

## Common Commands

```bash
npm run dev            # Start dev server
npm run build          # Production build
npm run lint           # Check lint errors
npm test               # Run tests
npm run clean-build    # Clean rebuild
```

## What NOT to Do

- Push directly to `main` or `test`
- Commit `.env` or secrets
- Use `any` type without justification
- Build SQL with string interpolation
- Skip testing
- Merge your own PR
- Move ticket to DONE (move to TO REVIEW instead)
