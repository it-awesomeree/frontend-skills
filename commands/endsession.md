---
description: End-of-session checklist — push WORKLOG, ERRORLOG, sync CLAUDE.md, and recap
---

# End-of-Session Checklist

Mandatory checklist before closing any session. Ensures all work, errors, and rule changes are documented and pushed to GitHub.

## Steps

### Step 1 — WORKLOG
- Review what was done this session
- Add new entries to `C:/Users/User/Documents/GitHub/frontend-skills/WORKLOG.md`
- Include: ticket IDs, what was fixed/built, tools used, key decisions

### Step 2 — ERRORLOG
- Review errors encountered this session
- Add new entries to `C:/Users/User/Documents/GitHub/frontend-skills/ERRORLOG.md`
- Include: error description, root cause, fix, prevention

### Step 3 — CLAUDE.md Sync
- Check if `~/.claude/CLAUDE.md` was modified this session
- If yes, copy updated version to `C:/Users/User/Documents/GitHub/frontend-skills/CLAUDE.md`

### Step 4 — Commit & Push
```bash
cd /c/Users/User/Documents/GitHub/frontend-skills && git add . && git status
```
- Only commit files that have actual changes
- Use descriptive commit message summarizing what was documented
- Push to GitHub

### Step 5 — Recap
Give Kelly a brief summary:
- What was logged in WORKLOG
- What was logged in ERRORLOG
- Whether CLAUDE.md was synced
- Confirm push succeeded

## Rules
- Do NOT skip any step — check each one even if you think nothing changed
- Do NOT assume previous sessions already logged everything
- If nothing changed for a step, say "no changes" and move on
- Push everything in a single commit to `frontend-skills` repo
