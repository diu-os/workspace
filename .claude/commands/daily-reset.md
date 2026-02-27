Daily session reset and progress checkpoint for DIU OS.

Steps:

1. **Remind to check usage** (built-in commands, run manually):
   - Type `/cost` to see token costs for this session
   - Type `/usage` to see token usage breakdown
   - Note: these must be run manually before /clear

2. **Determine today's date** using the current date from the environment.

3. **Collect today's progress** before clearing context:
   - Run `git -C ~/projects/diu-os/diu-contracts log --oneline --since="24 hours ago"` — contract commits
   - Run `git -C ~/projects/diu-os/physics-tutorial log --oneline --since="24 hours ago"` — frontend commits
   - Run `git -C ~/projects/diu-os log --oneline --since="24 hours ago"` — workspace commits
   - Check `git -C ~/projects/diu-os/diu-contracts diff --stat HEAD` for uncommitted work in progress
   - Summarize any ongoing context from this session (open tasks, blockers, decisions made)

4. **Write daily log** to `~/projects/diu-os/_workspace/daily-log/YYYY-MM-DD.md` (replace with actual date):
   ```markdown
   # DIU OS — Daily Log YYYY-MM-DD

   ## Done Today
   - <list commits and completed tasks>

   ## In Progress
   - <uncommitted changes, open PRs, ongoing work>

   ## Blockers / Decisions
   - <anything unresolved>

   ## Plan for Tomorrow
   - <next steps from ROADMAP.md / current sprint>

   ## Notes
   - <any context worth preserving>
   ```
   Create `~/projects/diu-os/_workspace/daily-log/` directory if it doesn't exist.

5. **Load context for new session**:
   - Read `~/projects/diu-os/CLAUDE.md` — project rules and current status
   - Read `~/projects/diu-os/ROADMAP.md` — current sprint priorities
   - Read the previous day's log from `_workspace/daily-log/` (most recent file) if it exists

6. **Print session summary** in this format:
   ```
   ═══════════════════════════════════════════
   DIU OS — Daily Reset  [YYYY-MM-DD]
   ═══════════════════════════════════════════

   YESTERDAY / TODAY SO FAR
   ─────────────────────────
   <bullet points from git log and session>

   CURRENT PHASE: <from CLAUDE.md>
   <P0/P1/P2 tasks from sprint>

   PLAN FOR TODAY
   ──────────────
   <top 3 priorities from ROADMAP + blockers>

   MODEL: switch to Sonnet with /model claude-sonnet-4-6
   NEXT:  /clear  →  then start working
   ═══════════════════════════════════════════
   ```

7. **Remind the user**:
   - Run `/model claude-sonnet-4-6` to set default model for the session
   - Run `/clear` to reset context (log is already saved)
   - After /clear, run `/catchup` if resuming mid-branch work
