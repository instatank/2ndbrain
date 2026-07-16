# Second-brain source map, YouTube pipeline, and the save-to-brain skill

- Date: 2026-07-16 (IST)
- Project/repo(s): instatank42 (branch `claude/second-brain-architecture-7ayd8r`), 2ndbrain (created and seeded)
- Where it ran: cloud (claude.ai/code)

## What this session was about

The founder asked for a complete map of every data source that should feed
his second brain, a plain-language explanation of how a huge file-based
memory coexists with the model's small context window, and then — mid
session — to actually build two of the newly mapped sources: the YouTube
tagged-videos pipeline (in full) and the save-to-brain skill for Claude
Code sessions (this file is its first output).

## Decisions made

- **Full source map lives in `docs/BACKLOG.md`** — one table from live
  banks down to explicitly excluded sources, with scoping entries for
  Gmail, Google Calendar, Telegram exports, Kindle highlights, finance
  statements, and a people file. Suggested build order after current
  deploys: Gmail → Drive notes → Calendar, because they answer the most
  real weekly questions per unit of effort.
- **YouTube: tagging over history.** Watch history stays excluded (fails
  the weekly-use test); deliberate capture is in — sharing a link to the
  Telegram bot IS the tag. Chosen over playlist/API plumbing for zero
  setup and zero cost.
- **DayOS learning-log links auto-fetch silently, daily** (founder's
  explicit call, option 2 of the fork offered; also his call that the 2h
  sync cadence would be overkill). This is the one deliberate exception to
  the confirm-first ingestion rule, on the grounds that logging a link in
  his own learning log is already the act of curation. Recorded in
  `docs/ROADMAP.md`'s decision log so no future session reverts it.
- **Claude Code sessions enter the brain via a skill, not an export
  script.** The founder's sessions happen in the desktop app and may
  execute in the cloud, so a Mac-filesystem export can't be assumed to see
  them. A skill that has Claude itself condense the session and git-push
  the digest works identically local or cloud. The `instatank/2ndbrain`
  repo (created by the founder today) is the storehouse; the bot will
  mirror it read-only like the playbook bank.
- **Transcript scraping is hand-rolled stdlib** (oEmbed + watch-page
  caption tracks), not youtube-transcript-api/yt-dlp — same raw-HTTP ethos
  as the DayOS client, and those libraries break against IP blocks just
  the same.

## Insights & learnings

- The sandbox proxy blocks youtube.com entirely (tunnel 403), so the
  transcript scrape is validated only against the known page format —
  the first real fetch happens on the VPS. If YouTube blocks the VPS IP
  too, the designed paste-fallback path is the norm, not a bug.
- `ingest()` rewriting the shared status file mid-run silently clobbered
  the auto-fetch retry bookkeeping kept in memory — fixed by making every
  bookkeeping write load-modify-save. Pattern to remember for any module
  sharing a status file with another writer.
- A silent auto-ingest path still needs a two-tier failure policy: a
  crashed run is bank breakage (health banner), but one caption-less video
  is not — it retries 3 runs then parks, visible in `/sync` only.

## What shipped / state of the work

All on `claude/second-brain-architecture-7ayd8r` (auto-merges to `main`),
offline-tested, awaiting the founder's VPS `git pull` + `setup_vps.sh`
re-run:

- `docs/HOW_IT_WORKS.md` — founder explainer (library/desk analogy) for
  how the memory banks stay decoupled from the model's context.
- Source map + six new scoping entries in `docs/BACKLOG.md`.
- YouTube bank end-to-end: `youtube_ingest.py`, `youtube_store.py`, bot
  tools `search_youtube`/`youtube_video`, confirm-first buttons,
  paste-transcript AND paste-summary fallbacks, multi-link batch messages,
  `youtube_autofetch.py` + daily 06:30 IST timer + `/sync` scan,
  `tests/test_youtube.py` (all repo suites green; handler flows driven
  with fake Telegram objects).
- `instatank/2ndbrain` seeded: README, canonical
  `.claude/skills/save-to-brain/SKILL.md`, this digest as the first entry.

## Open items

- Founder: VPS `git pull` + re-run `setup_vps.sh`, then first WhatsApp
  export, first YouTube link, `/digest` (walkthrough `deploy/DEPLOY.md`).
- Bot-side mirror of 2ndbrain (read tools + sync, playbook-bank pattern) —
  the next build step for this source; scoped in `docs/BACKLOG.md`.
- Founder: one-time `cp` of the skill into `~/.claude/skills/` for local
  Mac sessions (README has the command).
- Still pending from before: Phase-1 staleness drill, Wispr Flow local Mac
  run, GitHub default-branch flip to `main`.
