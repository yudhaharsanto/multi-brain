# Multi Brain Memory Layout

## Purpose

`.multibrain/` is lightweight shared memory across agents. Keep the top index short, route reads through named buckets, expand into context files only when needed — and keep the whole thing from rotting.

## Folder structure

```text
.multibrain/
├── session.md          # state + index of indexes
├── indexes/
│   ├── agents.md
│   ├── auth.md
│   └── deploy.md
└── context/
    ├── 2026-05-11-1735-codex-multi-brain-skill.md
    └── 2026-05-12-0910-claude-login-page.md
```

Repo root may also carry:

```text
AGENTS.md
CLAUDE.md
.gitattributes
```

These point agents at the startup flow. Never replace repo-specific rules to add Multi Brain.

## `session.md` rules

An index of indexes, plus state. Not a diary.

Two sections, in this order:

1. `## State` — phase, focus, last verified command, blockers, next action.
2. `## Buckets` — one line per bucket.

Required parts per bucket entry:

- Bucket name
- Short scope description
- Last updated timestamp
- Pointer to the sub-index file

```md
- `auth` — login, sessions, tokens. Last updated: 2026-05-11 17:35 WIB -> .multibrain/indexes/auth.md
```

Guidelines:

- One line per bucket, one file under 40 lines total.
- New line only when creating a new bucket.
- Refresh `Last updated` when appending work.
- **Rewrite `## State` after every write session.** A stale State block misleads worse than no block.
- No file lists, no command output.

## Init rules

`multi brain init` is a non-destructive bootstrap.

1. Create the `.multibrain/` structure if missing.
2. Create `.multibrain/indexes/agents.md`.
3. Append the `.gitattributes` merge rules.
4. Merge Multi Brain startup guidance into `AGENTS.md` / `CLAUDE.md` — insert, never replace.
5. Run `scripts/mb-check`.

## Named bucket rules

Each file in `indexes/` is the log for one task area.

Good names: `agents`, `auth`, `deploy`, `testing`, `ui`, `billing`. Topic names, not repo-specific ones.

Entry format:

```md
- 2026-05-12 09:10 WIB — [CC]: fixed the login page and form validation -> .multibrain/context/2026-05-12-0910-claude-login-page.md
```

Guidelines:

- Newest first, one line each.
- **Search before adding.** Same file or decision already listed → edit that line.
- Soft cap 25 entries. Over cap → compress the oldest half into one context note, leave one summary line pointing at it.
- Markers last: `[stale]`, `[done]`.

## Context file rules

Open one only when the task needs detail.

Sections: `## Goal`, `## Summary`, `## Decisions`, `## Changes`, `## Files`, `## Verification`, `## Next`.

Write one when the work has a decision, a blocker, a verification command worth re-running, or more than 3 files touched. Otherwise the index line is enough.

Purpose is recall and handoff, not a transcript.

## Decay rules

- >30 days old, not re-verified → append `[stale]`.
- `[stale]` with no live pointer → delete.
- `[done]` >30 days → compress into a context summary, keep the pointer.
- Compress before deleting. Never drop history that still has a live pointer.

## Concurrency

`.gitattributes`:

```text
.multibrain/session.md      merge=union
.multibrain/indexes/*.md    merge=union
.multibrain/context/*.md    merge=union
```

Prepend-only writes plus `union` merge means two agents' entries both survive. On a real conflict, keep both entries and dedup afterwards.

## Read strategy

Before starting:

1. Read `.multibrain/session.md`.
2. Read `## State`.
3. Read the one matching bucket, first ~15 entries.
4. Read pointed context files only when they decide the task.

After work:

1. Dedup check on the target bucket.
2. Prepend one line.
3. Rewrite `## State`; refresh `Last updated`.
4. Context note if warranted.
5. Compress if the bucket is over cap.
6. Run `scripts/mb-check`.

## Validation

`scripts/mb-check [root]` checks: missing structure, missing `## State`, dead `->` pointers, absolute or out-of-tree pointers, over-cap buckets, duplicate entries, stale counts.

Exit 1 = broken memory. Fix before continuing.
