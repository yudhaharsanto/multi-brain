# Multi Brain

Shared working memory for coding agents — [CC], OpenCode, Codex, or anything that can read markdown.

Agents forget. Every new session re-reads the repo, re-derives decisions, re-discovers blockers. Multi Brain leaves a small, structured trail behind instead: a master index that carries current state, named topic buckets with one-line entries, and deeper context notes opened only when they matter.

Markdown files. No server, no database, no embeddings.

## Why

Naive agent memory fails four ways:

| Failure | Symptom | Fix |
|---|---|---|
| **No state** | Next agent can't tell what's done | `## State` block in `session.md` |
| **Rot** | Stale facts trusted as current | `[stale]` / `[done]` + 30-day TTL |
| **Corruption** | Pointers to deleted files | `scripts/mb-check` |
| **Conflict** | Two branches clobber the same log | `merge=union` + prepend-only writes |

Each rule in the skill exists to stop one of these. Nothing else.

## Install

Copy the folder into your agent's skill directory.

```bash
# Claude Code
git clone https://github.com/yudhaharsanto/multi-brain.git ~/.claude/skills/multi-brain

# OpenCode
git clone https://github.com/yudhaharsanto/multi-brain.git ~/.config/opencode/skill/multi-brain
```

Start a new session. The skill triggers on "multi brain init", "resume from another agent", "leave context for the next session", or any request to hand work off.

## Use

### Initialize a repo

> multi brain init

Creates `.multibrain/`, appends `.gitattributes` merge rules, merges startup guidance into `AGENTS.md` / `CLAUDE.md` **without replacing existing rules**, then runs the checker.

### Read before work

1. `.multibrain/session.md` — small, read it whole
2. `## State` — where the project actually stands
3. the one bucket matching the task, first ~15 entries
4. pointed `context/` files only when they decide the task

### Write after work

Search the bucket first — edit an existing line instead of duplicating it. Then:

- prepend one line, newest first
- rewrite `## State`, refresh `Last updated`
- write a context note only for a decision, a blocker, a verification command, or >3 files touched

### Decay

- over 25 entries per bucket → compress the oldest half into one context note, leave one summary line
- older than 30 days unverified → `[stale]`
- `[stale]` with no live pointer → delete
- compress before deleting; never drop history with a live pointer

### Validate

```bash
scripts/mb-check .        # exit 1 = broken memory
```

Catches dead pointers, missing `## State`, over-cap buckets, duplicate entries, stale counts. Runs in under a second.

## Layout

```text
.multibrain/
├── session.md          # state + index of indexes
├── indexes/
│   ├── agents.md
│   ├── auth.md
│   └── deploy.md
└── context/
    └── 2026-05-12-0910-claude-login-page.md
```

`session.md`:

```md
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

## Contents

| File | Purpose |
|---|---|
| `SKILL.md` | agent instructions — the actual skill |
| `references/memory-layout.md` | full layout, decay, concurrency rules |
| `assets/session-template.md` | master index + State |
| `assets/sub-index-template.md` | bucket |
| `assets/context-template.md` | deep note |
| `assets/agents-snippet.md` | insert for `AGENTS.md` |
| `assets/claude-snippet.md` | insert for `CLAUDE.md` |
| `assets/gitattributes-snippet` | conflict-free merge rules |
| `scripts/mb-check` | integrity validator |

## Non-goals

No MCP server. No embeddings or vector search. No JSON schema. No git hooks. Markdown plus one grep-based checker covers repo-local agent memory; anything heavier is a database with extra steps.

## License

MIT
