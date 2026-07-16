# 2ndbrain — the second brain's storehouse

**Private.** This repo is the durable store for knowledge the founder's
personal AI agent should remember — starting with digests of Claude Code
sessions. Plain markdown only; the agent's VPS mirrors this repo read-only
into its memory banks (the same git-mirror pattern as the playbook bank in
`instatank/instatank42`).

## Layout

```
sessions/<YYYY>/<YYYY-MM-DD>--<project>--<topic>.md   Claude Code session digests
.claude/skills/save-to-brain/SKILL.md                 the skill that writes them (canonical copy)
```

Future lanes (other kinds of brain content) get their own top-level folders
as they earn them — one folder per lane, markdown only.

## How digests get here: the `/save-to-brain` skill

At the end of any Claude Code session worth keeping, say **`/save-to-brain`**
(or "save this session to the brain"). Claude condenses the session —
decisions, insights, state of the work, open items; no transcript noise —
and pushes the digest to this repo's `main`. Works from cloud sessions
(claude.ai/code — add this repo to the session if it isn't already) and
local sessions alike.

### One-time install for local Mac sessions

Local Claude Code sessions look for personal skills in `~/.claude/skills/`.
From a clone of this repo:

```
mkdir -p ~/.claude/skills
cp -r .claude/skills/save-to-brain ~/.claude/skills/
```

After that, `/save-to-brain` is available in every local session, whatever
repo it's in. (Cloud sessions get the skill automatically whenever this
repo is added to the session; `instatank/instatank42` also carries a copy.)

## Backfilling past sessions (one-time)

`/save-to-brain` captures sessions from here on. To bring in sessions you had
*before* installing the skill, there are two ways:

- **A few sessions:** reopen each one and run `/save-to-brain`. In the CLI:
  `claude --resume` lists past sessions to pick from; in the desktop app,
  reopen the conversation from history. Only works for sessions still saved
  and reopenable.
- **A lot of sessions:** `brain_backfill.py` (in the `instatank/instatank42`
  repo) automates it — it reads Claude Code's own on-disk session
  transcripts, condenses each into a digest, and commits them here. Run
  `python3 brain_backfill.py --list` first to see what it finds. **Important:
  it only sees sessions that ran *locally* on your Mac** (`~/.claude/projects/`);
  desktop-app sessions that executed in the cloud leave nothing local, so
  `--list` is also the test of whether backfill is possible at all. It writes
  a local commit and does not push until you review it (or pass `--push`).
  Full walkthrough: that script's header comment.

## Rules

- **The canonical copy of the skill lives here.** The copy in
  `instatank42/.claude/skills/` is a convenience mirror — edit here first.
- **Markdown only, no secrets.** Digests must never contain tokens, keys,
  or `.env` contents.
- **Never force-push.** This repo is memory — history is the point.
- **The agent only reads.** The bot's VPS mirror of this repo is read-only;
  writes happen through the skill (or by hand).
