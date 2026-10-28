# Multi Brain Master Index

Master index plus current state. Keep this file under 40 lines.

Rules:
- `## State` is mandatory and must reflect reality after every write session
- One line per named bucket
- Update `Last updated` when a bucket receives an entry
- The work log lives in `.multibrain/indexes/*.md`, not here

## State

- Phase: <what stage the project is in>
- Focus: <the one thing being worked on now>
- Verified: <last command that passed> @ YYYY-MM-DD HH:MM TZ
- Blockers: <none | what is blocking>
- Next: <the immediate next action>

## Buckets

- `agents` — shared notes for agent workflows, prompts, and skill behavior. Last updated: YYYY-MM-DD HH:MM TZ -> .multibrain/indexes/agents.md
