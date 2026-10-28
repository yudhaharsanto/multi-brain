---
name: multi-brain
description: This skill should be used when work needs to be shared or remembered across [CC], OpenCode, Codex, or other agents through a two-level memory index in `.multibrain/session.md`, named sub-index files in `.multibrain/indexes/`, and selective deep context files in `.multibrain/context/`. Also covers `multi brain init`, resuming from another agent's memory, and validating or repairing a rotted `.multibrain/` tree.
---

# Multi Brain

## Overview

Shared working memory across agents without forcing every agent to read everything.

Three levels, three jobs:

| Level | Job | Budget |
|---|---|---|
| `.multibrain/session.md` | **State + index of indexes** | ≤ 40 lines |
| `.multibrain/indexes/*.md` | one-line entries per topic | ≤ 25 entries each |
| `.multibrain/context/*.md` | full reasoning, decisions, verification | no limit |

Memory fails in four ways. Every rule below exists to stop one of them:

1. **No state** → next agent cannot resume. Fixed by the `## State` block.
2. **Rot** → stale facts get trusted. Fixed by `[stale]` + TTL.
3. **Corruption** → pointers to deleted files. Fixed by `mb-check`.
4. **Conflict** → two agents prepend to the same file. Fixed by `merge=union` + prepend-only.

## Invariants

- `session.md` is scannable in under a minute. It is not a work log.
- Newest entry first. Always.
- Every `->` pointer must resolve to a real file.
- Never rewrite a bucket wholesale. Prepend one line, or edit one line in place.
- One bucket per topic. New bucket only when no existing one fits.
- Facts only. No stream of consciousness, no command output dumps.
- Nothing is permanent. Anything unverified is marked and eventually compressed.

## Workflow

### 0. `multi brain init`

Non-destructive bootstrap of the current repo:

1. Create `.multibrain/session.md` from `assets/session-template.md`.
2. Create `.multibrain/indexes/` and `.multibrain/context/`.
3. Create `.multibrain/indexes/agents.md` from `assets/sub-index-template.md`.
4. Append `.gitattributes` rules from `assets/gitattributes-snippet` (create the file if absent, append if present).
5. Read root `AGENTS.md` / `CLAUDE.md` before editing. Insert the Multi Brain section from `assets/agents-snippet.md` / `assets/claude-snippet.md`. **Never replace existing rules.**
6. If no root instruction file exists, create `AGENTS.md` only. Create `CLAUDE.md` only if the repo already uses that convention.
7. Run `scripts/mb-check` and report the result.

### 1. Resume — read in this order

1. Read `.multibrain/session.md` in full. It is small.
2. Read the `## State` block. This is the answer to "where are we".
3. Read the first ~15 entries of the one bucket matching the task.
4. Open a `context/` file only when it decides the current task.

Do not read every bucket. Do not read context files "for background".

### 2. Work

Keep scratch notes outside `.multibrain/`. Distill before writing.

### 3. Write back

**Dedup first.** Before adding an entry, search the target bucket for the same file, feature, or decision. If a line already covers it, edit that line instead of adding another.

Then, one line, newest first:

```md
- YYYY-MM-DD HH:MM TZ — [Agent]: what changed -> .multibrain/context/FILE.md
```

Then update `session.md`:

- Refresh that bucket's `Last updated`.
- Rewrite the `## State` block. This is mandatory — a stale State block is worse than none.

Write a `context/` file only when the work contains at least one of: a decision, a blocker, a verification command worth re-running, or more than 3 files touched. Otherwise the index line is enough.

### 4. Decay

- Entry older than 30 days and not re-verified → append `[stale]`.
- `[stale]` entry with no live pointer → delete it.
- `[done]` entry older than 30 days → compress into one summary line in a context note, keep the pointer.
- Bucket over 25 entries → compress the oldest half into one context note, replace with a single summary line that points to it.
- Compress before deleting. Never drop history that still has a live pointer.

Markers go at the end of the line: `... -> path [stale]`.

### 5. Validate

Run `scripts/mb-check` after any write session and after `init`.

```bash
bash .multibrain/../scripts/mb-check .   # or scripts/mb-check from the skill dir
```

It reports dead pointers, missing `## State`, over-cap buckets, duplicate entries, and stale counts. Exit 1 means the memory is broken — fix before continuing.

## Format rules

- Relative repo paths in pointers. Never absolute.
- Bucket names short and durable: `agents`, `auth`, `deploy`, `ui`, `billing`, `testing`.
- Markdown only. No JSON, YAML, or logs in indexes.
- Timestamps in the repo's local timezone, `YYYY-MM-DD HH:MM TZ`.
- `session.md` = index of indexes plus state. The log lives in buckets.

## Examples

`session.md`:

```md
# Multi Brain Master Index

## State

- Phase: building auth
- Focus: refresh-token rotation
- Verified: `pnpm test auth` green @ 2026-10-28 14:02 WIB
- Blockers: none
- Next: wire rotation into mobile client

## Buckets

- `auth` — login, sessions, tokens. Last updated: 2026-10-28 14:02 WIB -> .multibrain/indexes/auth.md
```

Bucket entry:

```md
- 2026-10-28 14:02 WIB — [CC]: added refresh-token rotation, 4 tests -> .multibrain/context/2026-10-28-1402-cc-token-rotation.md
```

## Resources

- `references/memory-layout.md` — full layout and read/write rules.
- `assets/session-template.md`, `assets/sub-index-template.md`, `assets/context-template.md` — templates.
- `assets/agents-snippet.md`, `assets/claude-snippet.md` — root instruction inserts.
- `assets/gitattributes-snippet` — conflict-free merge rules.
- `scripts/mb-check` — integrity validator.

Deliberate skip: no MCP server, no embeddings, no JSON schema, no hooks. Markdown plus one grep-based checker covers repo-local agent memory. Revisit only if memory spans many repos.
