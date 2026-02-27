# Work Log

Kelly's activity log for the AWESOMEREE Web App. Entries are organized by work session, starting from 2026-02-16.

**Session ID Convention**: Use `MMDD-N` format (e.g., `0219-1`) where MMDD is the date and N is the session number for that day.

---

### Session 0226-1 (2026-02-26)

**GRBT-155 — Stock Adjustment Flow (Review & Close)**

- **Context**: Stock adjustment bot (`stockID.py` on VM5) was 100% broken since Feb 11. Ticket was in TO REVIEW — Jana (Janarthan Nair) submitted fixes. Agnes/Claude Code reviewed.
- **Workflow**: `/workflow-webapp` — Path B (dev-submitted work, review only)
- **Work done**:
  - Read full Jira ticket (description, 5 comments, 3 attachments)
  - Dev compliance check: verified all 7 of Agnes's action items from Feb 17
  - Code review of PR #22 (bot-scripts repo) — 903-line `stockID.py` Selenium scraper
  - Security audit: PASS (no hardcoded creds, parameterized SQL, .env usage)
  - Critical code review ("angry senior dev" style) — identified 12 issues including no session validation, no idempotency, fragile CSS selectors, self-merged PR
  - Posted technical review comment on Jira (Step 8B) — compliance matrix, security review, root cause classification
  - Created GitHub changelog `docs/changelog/GRBT-155.md` in bot-scripts repo (Step 11) — committed via PR (main is protected)
  - Posted non-technical CS team comment on Jira (Step 12B) — plain language for Myrsya
  - Transitioned ticket to DONE, ranked to top of Done column
- **Root cause**: Configuration/Environment Issue + System/Code Defect — expired SiteGiant session, rotated DB password, retry bug in WHERE clause
- **Verdict**: REVIEW PASSED — all 7 action items complete, bot operational at 67% (remainder explained)
- **Out of scope**: Stock reservation automation (needs separate ticket)
- **Tools used**: Claude-in-Chrome (Jira + GitHub), Jira REST API (comments, transitions, ranking), GitHub web UI

**GRBT-194 — Stock Adjustment (Started, incomplete)**

- **Context**: Follow-up to GRBT-155 — same bot, new issue. SiteGiant cookie consent banner overlaying confirm button.
- **Status**: Step 1 (Study Jira Ticket) started — read ticket details, Jana's comment, PR #27 linked. Session ended before completing review.
- **Will resume next session**

---
## AW-357 — Add duplicate submission prevention on Replacement & Review form

**Date:** 2026-02-25 to 2026-02-27
**Status:** Deployed to Production
**Assignee:** Kelly Ee
**Reporter:** Agnes Foong
**Labels:** frontend, backend

### Problem
The Replacement & Review form (`/customer-service/replacement-review`) had no protection against duplicate submissions. When the system was slow, users could double-click the submit button and create identical records (see GRBT-202). Audit log showed two identical records created 1 second apart — classic double-submit.

### What was done

**1. Frontend — Disable submit button on click**
- Added `submitting` state on the Add New form — disables the Create button and shows "Submitting…" while request is in-flight
- Re-enables on error so the user can retry
- Handles 409 Conflict response and shows user-friendly duplicate warning

**2. Backend — Duplicate order check**
- Added `checkRecentDuplicate()` function — checks if a record with the same `order_id` was created within the last 60 seconds
- Returns 409 Conflict if duplicate detected, blocking repeated submissions
- Implemented atomic duplicate check to prevent TOCTOU race condition (handles concurrent requests)

**3. Logging gap — Firebase auth headers**
- Passed Firebase auth headers (`x-firebase-uid`, `x-user-email`, `x-user-name`) from the replacement review form
- Audit logs now capture who created each record in `history_logs` for both create and update actions

### Files changed
- `lib/replacement-review.ts` — added `checkRecentDuplicate()`
- `app/api/replacement-review1/route.ts` — added duplicate check before insert
- `components/replacement-review1/controls.tsx` — added `submitting` state to prevent double-click

### Commits
- `3576461` — fix-add-duplicate-submission-prevention-on-Replacement-&-Review-form-(AW-357)
- `edd9480` — fix: atomic duplicate check to prevent TOCTOU race condition (AW-357)

### PRs & Deployment
- **PR #369**: `kelly/AW-357-duplicate-prevention-and-date-fix` → merged to `test` (Feb 26, 4:34 PM)
- **PR #364**: `test` → `main` (merged Feb 27, 12:32 AM) — bundled with 25 commits from multiple devs
- **Deployment issue**: GitHub Actions billing was locked, blocking all deployments after Feb 26 ~4 PM
- **Resolution**: Billing fixed by Agnes → manually triggered Agent 5 (Build & Deploy Staging) from GitHub Actions
- **Agent 5** (Run #154): Build (3m 24s) → Deploy to Staging (16m 42s) → Smoke Tests (59s) — all passed
- **Agent 6** (Run #162): Production Canary Rollout (16m 34s) — 5% → 25% → 50% → 100% traffic — all passed
- **Live in production**: Feb 27, 2026 ~10:35 AM at https://employee.awesomeree.com.my


### Session 0220-1 (2026-02-20)

**AW-298 — Comp. Analysis MY Bug Fix (Done)**

- **Context**: COMP Product count was not aligned with per-variation competitor display in Shopee MY History. Competitors were being deduped incorrectly when listings had multiple variations.
- **Root cause**: Dedup logic used `|||` magic string separator to group competitor+variation pairs — fragile and error-prone. Count aggregation didn't match the per-variation display logic.
- **Fix implemented**:
  - Added shared `getCompMonthlySales()` function in `lib/shared/shopee-history-calculations.ts` — single source of truth for comp monthly sales calculation
  - Replaced `|||` string separator with proper two-level `Map` data structure
  - Added `MAX_QUALIFIED_COMP_ROWS = 10` safety cap (prevents unbounded rendering if data is unexpectedly large)
  - Moved `formatPriceValue()` to shared location (was duplicated across files)
  - Removed unnecessary `as any` casts (TypeScript strictness)
- **Files changed**: `components/Shopee-MY-History/grouped-rows.tsx`, `components/ca-test/Shopee-MY-History/grouped-rows.tsx`, `lib/shared/shopee-history-calculations.ts`, `lib/shared/shopee-history-calculations.test.ts`
- **Commit**: `bf13282` — fix: align COMP Product count with per-variation competitor display

### Session 0219-2 (2026-02-19)

**AW-324 — Costing: Frontend Data Fetching to Database**

- **Context**: The costing module needed to pull comp analysis data from the frontend into the database for the selling price calculation pipeline.
- **Work done**:
  - Built frontend-to-database data fetching in `lib/services/shopee-products-enrichment.ts`
  - Tested syncing of comp analysis data to the costing module via `app/api/costing-excel/selling-price/route.ts`
- **Files changed**: `lib/services/shopee-products-enrichment.ts`, `app/api/costing-excel/selling-price/route.ts`
- **Commits**:
  - `49c6f94` — AW-324 frontend data fetching to database
  - `9979144` — testing the syncing of the comp analysis → costing
- **Status**: To Do (data fetching pipeline set up, further integration pending)

### Session 0219-1 (2026-02-19)

**AW-317 — Chatbot Conversation Log Viewer: Round 1 Improvements (In Progress)**

- **Context**: Building a premium UI/UX chatbot conversation log viewer. This session focused on the first round of major UI improvements to the viewer page.
- **Features implemented**:
  - Wider chat detail panel — responsive scaling on `md`/`lg` screens for better readability
  - Stats time breakdown — Today/Week/Month conversation counts displayed in metrics cards
  - Export CSV button — added to filter bar, client-side CSV generation
  - Search by message content — `EXISTS` subquery on `chatbot_messages` table for full-text search
  - Full pagination bar — rows per page selector, go-to-page input, page number navigation
  - Error handling + retry — error banner with Retry button on API failures
  - URL filter persistence — all filter states synced to URL search params (shareable/bookmarkable)
  - Date separators in chat — visual dividers between messages on different days
  - Full CSV export — fetches all filtered results in batches (not just current page)
- **Files changed**: `app/api/chatbot/conversations/route.ts`, `app/chatbots/overview/page.tsx`, `components/chatbot/conversation-viewer.tsx`
- **Commits**:
  - `c197bc6` — AW-317: Add chatbot viewer improvements — wider panel, time stats, CSV export, message search, full pagination
  - `2492fa2` — make improvement on the chatbots (half way done)
  - `a031942` — add on the pagination inside the bottom
- **Remaining TODO**: Connection pooling (`dbPool.executeWithRetry()`), auto-scroll to latest message, column sorting, product context display, mark as resolved action, real-time updates / auto-refresh
