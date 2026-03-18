# Backlog

Future tasks and follow-ups that are not completed in-session and not yet finalized into Jira.

**Session ID Convention**: Use `MMDD-N` format (e.g., `0219-1`) where MMDD is the date and N is the session number for that day.

---

### Session 0312-1 (2026-03-12)

- [ ] Fix GitHub MCP auth (`GITHUB_MCP_TOKEN`) so PRs can be created directly from MCP.
- [ ] Decide whether to patch `.github/workflows/swarm-agent-1-review.yml` to pass explicit `GH_TOKEN` / `github_token` into Claude action.
- [ ] After token fix, create PR `main <- codex/vvip-sg-from-test` and paste finalized Senior Review section.

---

### Session 0317-1 (2026-03-17)

- [ ] Commit and push SG/VVIP join hardening in `fix/http-500-vvip-sg-queries` (use indexed `cd.our_item_id_extracted` path consistently).
- [ ] Deploy updated branch to test and re-verify:
  - `/analytics/table/shopee-sg`
  - `/analytics/table/shopee-my-vvip`
- [ ] Capture before/after `rows_query_ms` and set acceptance target (< 2s steady-state).
- [ ] Add timeout-focused SQL telemetry/alerts for SG/VVIP products fetch paths.

---
