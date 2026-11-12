# Project Agent Instructions

This project keeps a memory bank. Your memory resets between sessions, so the memory bank is your only continuity. Treat it as the source of truth for project intent.

## At the start of every session

Read these files, in this order, before doing anything else:

1. `memory-bank/projectbrief.md` — what we're building and why
2. `memory-bank/productContext.md` — the user problem and UX goals
3. `memory-bank/systemPatterns.md` — architecture and design decisions
4. `memory-bank/techContext.md` — stack, constraints, dependencies
5. `memory-bank/decisionLog.md` — durable decisions and ADR index
6. `memory-bank/activeContext.md` — current focus (most important)
7. `memory-bank/progress.md` — what works, what's broken
8. `.rules` — project-specific patterns and preferences

Then read the last 5–10 git commits for recent context.

If the task clearly touches a specific feature, integration, deployment path, testing strategy, or API, look for relevant optional docs under `memory-bank/` and read only the ones that apply. `memory-bank/archive/` holds rotated history. It is never part of the session read; open it only when a task needs that history.

If any memory bank file is missing, empty, or still only template placeholders, note it and ask the user before proceeding. A new bank starts blank by design. The `start` workflow routes a missing or blank bank to the right seed workflow: `discover` (no code yet), `from-prd` (a spec, no code yet), or `init-memory` (code exists, with or without a spec).

## While working

- Keep `activeContext.md` honest as the focus shifts during the session.
- For work that spans sessions, keep a `## Checkpoint` section in `activeContext.md` current: one resumable state (branch, what's done, the exact next step). Overwrite it, don't append; clear it when the work ships.
- When you discover a lesson that is non-obvious, reusable, costly to rediscover, and not already captured in code, tests, comments, or docs, append it to `.rules`.
- Before significant work, propose a plan and wait for confirmation. Once a plan or scope is approved, work autonomously inside that lane without re-asking per file; still stop for destructive actions, scope changes, outward-facing actions, and any approval gate a workflow itself defines.
- The memory bank is the source of truth for intent; code and observed results are the source of truth for current behavior. When they conflict, classify before acting: a stale current fact means the bank needs updating; code that breaks an accepted decision or intent is a possible regression. Flag either so the user can decide. Never rewrite accepted intent to match the code.

## Making changes

For non-trivial changes; use judgment on obvious one-liners. (Inspired by Andrej Karpathy's observations on LLM coding pitfalls and `multica-ai/andrej-karpathy-skills`, MIT.)

- **Think before coding.** Surface assumptions; if a request has more than one reasonable interpretation, present them instead of silently picking; push back when a simpler path exists; stop and ask when something is unclear.
- **Simplicity first.** Write the minimum that solves the problem: no speculative features, abstractions for single-use code, unrequested config, or error handling for impossible cases.
- **Surgical changes.** Touch only what the task needs; don't refactor or reformat adjacent code; match existing style; mention unrelated dead code rather than deleting it. One exception: when your fix repairs one occurrence of a defect that repeats at sibling sites, sweep for the siblings and fix or explicitly rule out each, bounded to the same root cause and the same safe fix.
- **Goal-driven execution.** Turn the task into a verifiable goal and loop until it's met. Judge gates (tests, typecheck, lint, build) by exit code, not by scanning their output for success strings.

## When the user says "update memory bank"

Refresh every file in `memory-bank/`. Focus especially on `activeContext.md`, `progress.md`, and `decisionLog.md` when decisions changed. A decision the user did not direct or approve is recorded as proposed. Show diffs before writing.

## Available workflows

Tavo Stack's workflows. Copied into this project they are called as `/name`; loaded from the Tavo Stack plugin they are `/tavo:name`:

- `start` — read the bank, summarize state, ask where to pick up (`start full` for a fuller dashboard, `start setup` for guided setup)
- `update-memory` — refresh the memory bank from this session's work, with diffs shown first
- `discover` — define a project from a rough idea and seed the bank (no code yet)
- `from-prd` — seed the bank from an existing PRD, brief, or spec
- `init-memory` — analyze the codebase and propose initial bank contents (code already exists)

## Ground rules

- Don't assume project context that isn't supported by the memory bank. Ask.
- Don't overwrite memory bank files silently. Show diffs first.
- Don't bloat the bank. It is signal, not a journal; one-offs don't belong.
- Code is the source of truth for what currently works. The bank is the source of truth for what we *meant* to build.
