# Brain mirror bank built: the Telegram bot can now search these session digests

- Date: 2026-07-16 (IST)
- Project/repo(s): instatank42
- Where it ran: cloud (claude.ai/code)

## What this session was about

Closing the standing "next build" from the backlog: the bot-side mirror of
this very repo (2ndbrain). The `/save-to-brain` skill was already pushing
Claude Code session digests here, but the Telegram agent had no way to read
them. This session gave the agent a read-only mirror of the repo and two
tools to search and read the digests — so questions like "what did we figure
out about X in that session last month?" now work from the phone.

## Decisions made

- **Reused the playbook bank's git-mirror pattern verbatim** (as scoped in
  the backlog) rather than inventing anything new: a shallow read-only git
  checkout under `memory/brain/repo/`, refreshed by the same 2-hour systemd
  timer and by `/sync`, with the same staleness-warning contract. Proven
  plumbing, one less pattern to maintain.
- **Configuration reuses the nightly backup's settings by default.** The
  nightly backup already points at this same 2ndbrain repo, so the new
  mirror falls back to `BACKUP_REPO_URL`/`BACKUP_REPO_TOKEN` when its own
  `BRAIN_REPO_*` variables are unset. Chosen so the founder has ZERO new
  configuration to do — updating the server and running `/sync` is enough.
  Explicit `BRAIN_REPO_*` overrides exist, and the token/URL travel as a
  pair so a backup token is never sent to a differently-configured URL.
- **The mirror reads only the repo's `sessions/` folder.** The repo's
  `memory/` folder is the bot's own nightly backup; letting the bot read
  its backup back in would be circular. The reverse loop is also safe:
  the backup job already skips nested git checkouts, so the new mirror
  never gets pushed back into this repo.
- **Tool names follow the backlog scoping**: `search_session_digests`
  (find a session or decision) and `session_digest` (read one digest in
  full, matched forgivingly by date, project, or topic words).

## Insights & learnings

- Search-result labels double as valid `session_digest` names because both
  are the digest filenames' stems (`YYYY-MM-DD--project--topic`) — the
  model can pass a search hit straight back to the read tool with no
  translation layer. Small design choice, but it removes a whole class of
  "tool couldn't find the thing the other tool just returned" failures.
- The digest-filename convention this repo's skill mandates (date, project,
  topic in the name) is what makes forgiving name-matching cheap: matching
  on the filename alone is enough, and no file contents need to be read to
  resolve a name.

## What shipped / state of the work

- New in `instatank/instatank42`: `brain_sync.py` (mirror orchestrator +
  `--status` CLI), `brain_store.py` (search/read side with staleness
  warnings and a system-prompt note), two bot tools wired into the tool
  list, the health banner, and `/sync`; a third entry in the 2-hour sync
  service so a brain failure can never block the DayOS sync; `.env.example`
  and `deploy/DEPLOY.md` § 8b (founder walkthrough); doc updates in
  CLAUDE.md and `docs/BACKLOG.md` (open item ticked).
- Tested offline in `tests/test_brain.py` against a local throwaway git
  repo shaped like this one — including proof that the `memory/` folder is
  unsearchable and that the backup-settings fallback and precedence work.
  The repo's full ten-file offline test suite passes.
- Committed on `claude/telegram-bot-mirror-xsbewh`, pushed, and merged to
  `main` (merge commit `02238d8`) per the repo's branch-flow rule.

## Open items

- **Founder, to take it live:** on the VPS run `git pull`, re-run
  `deploy/setup_vps.sh` (refreshes the sync service), then send `/sync` —
  the reply should include a line like "Brain: commit …, N session
  digests." Walkthrough: `deploy/DEPLOY.md` § 8b. No `.env` changes needed
  because the nightly backup is already configured.
- First live verification: ask the bot "what did we figure out about the
  YouTube pipeline in that session?" and check it quotes a digest from
  this repo.
