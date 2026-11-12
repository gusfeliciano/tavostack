# Workflow Contract

Tavo Stack workflows should be small, explicit, and portable across agents. Use this
contract when adding or revising a workflow.

## Required sections

Every workflow should make these things clear:

- **Trigger**: when to use it and when not to use it.
- **Required reads**: files, diffs, logs, or docs the agent must inspect first.
- **Allowed writes**: files the workflow may edit, and whether confirmation is required.
- **Output contract**: the exact shape users should expect back.
- **Stop conditions**: when the agent must pause for user input instead of guessing.

## The memory bank

Every reference to "the memory bank" or `.rules` in a workflow means the
project's `memory-bank/` folder and the `.rules` file at the project root.

## Defaults

- Read `AGENTS.md`, the relevant memory-bank files, `.rules`, and recent git history when project intent matters.
- Treat code as source of truth for current behavior.
- Treat the memory bank as source of truth for intended behavior once initialized.
- Show diffs before writing memory-bank files.
- Ask before changing product scope, architecture, dependencies, security posture, or public behavior.
- Prefer one focused workflow over a broad persona.
- Read optional `memory-bank/` docs only when the current task clearly touches that topic.

## Optional memory docs

Keep the core memory bank small. When a topic outgrows the core files, add focused
optional docs under `memory-bank/`, for example:

- `memory-bank/features/<feature>.md`
- `memory-bank/integrations/<service>.md`
- `memory-bank/ops/<runbook-context>.md`
- `memory-bank/testing.md`

Optional docs are not part of the required startup read. Agents should find and
read them only when relevant to the task.

## Memory writes

Use this promotion path:

1. Current task details go in the conversation or temporary plan.
2. Current session state goes in `memory-bank/activeContext.md`.
3. Completed status goes in `memory-bank/progress.md`.
4. Durable decisions go in `memory-bank/decisionLog.md` and, when useful, `docs/decisions/`.
   An entry says *decision / why / evidence / result* and keeps its date,
   status, and supersession link; an accepted-but-unbuilt decision records
   `result: pending`. ADRs still carry alternatives and consequences.
   Say in plain prose who decided: *proposed* (a suggestion, the agent's
   included) or *user-directed* / *user-approved*, with the source and date
   when known. Only the user accepts a decision: an agent cannot accept its
   own suggestion, and implementation alone is not approval. Explicit
   approval already given suffices, including a reviewed decision the user
   approved in a shown diff; do not ask again. An existing accepted entry
   without provenance stays accepted: mark provenance unknown, never demote it.
5. Reusable patterns and gotchas go in `.rules`. When a `.rules` entry is
   about to be written a second time, or a correction recurs, first ask why
   the existing guidance did not take (did it trigger? was it already there?
   wording or placement?), then propose the structural form — a test, a hook
   check, a CI step. Keep the prose rule until the mechanism exists and works;
   judgment rules stay prose with a concrete failure example.
6. Stable architecture goes in `memory-bank/systemPatterns.md`.

**Lesson bar.** A lesson (a gotcha, pattern, preference, or rejected
approach) is written to `.rules`, `systemPatterns.md`, or an optional doc
only when it is non-obvious, reusable beyond the current task, materially
costly or risky to rediscover, and not already adequately captured in code,
tests, comments, or docs. An artifact that already carries it makes bank
prose unnecessary. `update-memory` does not edit product code; it may suggest
a comment or test instead. The bar applies to lessons only: session state,
progress, status, and decisions are captured as usual.

Do not turn the memory bank into a journal. A line should survive because it helps
the next session make a better decision.

## Memory accuracy

Every `/update-memory` or `$update-memory` proposal includes two passes over
the selected effective bank, bounded to subjects changed or discovered in the
session. This is routine reconciliation, not a full repository audit.

- **Capture:** compare session decisions and observations with relevant git
  history, staged and unstaged diffs, and relevant new files. Identify durable
  changes missing from the bank: completed work, changed constraints, accepted,
  proposed or superseded decisions with their provenance, learnings that clear
  the lesson bar, unresolved problems, and the next action. A decision may
  have no code diff. Do not claim a test or manual verification ran without
  an observed result; distinguish committed work, uncommitted work, and
  released behavior.
- **Reconcile:** search the live bank and its own `.rules` for existing claims
  about those subjects. Read optional docs only when they match. Update current
  status, next steps, and obsolete blockers together; completing a feature also
  retires its pending task. Correct or remove stale facts in stable files even
  when product intent has not changed. Prefer one authoritative home for a
  volatile fact (release/completion status in `progress.md`, immediate work in
  `activeContext.md`) and references elsewhere over repeated status claims.

Classify discrepancies before proposing a correction: **current fact**,
**durable decision**, **intended future**, or **open question**. Code and
observed results establish current behavior; they do not supersede accepted
intent. If behavior violates a decision, report a possible regression for the
owner to resolve. A proposed decision is not accepted intent yet: treat it as
an open question, not a constraint. Missing evidence means unassessed or
unresolved, not false.
For external capability claims, record the source and applicable version or
verification date; re-check the source when relying on a changeable claim, or
say it could not be checked. A workaround for an external limitation may
also state, in plain language, the condition that would retire it (say, the
upstream fix it waits for), with the primary source and version or date when
applicable. The condition is optional. Meeting it is a reason to investigate
a safe removal within the existing scope and approvals, not proof that the
workaround, its explanation or accepted intent is stale: while the workaround
remains, an accurate explanation stays. Not meeting it does not by itself
prove the workaround correct.

Decisions are superseded with links, never silently deleted. Dated historical
entries, including recent changes and milestones, stay verbatim and rotate
under Retention; an old release mentioned as history is not a stale claim.

Before the proposed diffs, give a compact coverage summary: **Captured** (new
durable information and its evidence; any candidate lesson skipped under the
lesson bar, with the reason, when there were candidates), **Reconciled** (old
claims corrected and their locations), and **Unresolved** (missing evidence or
owner decisions; say none when there are none). This summary describes the subjects checked,
not a certification of the whole bank. Show diffs and use the existing
confirmation gate; reconciliation adds no separate approval step.

## Clean stop

When a session ends or context is about to compact, leave a state a
cold-start agent can resume from without redoing work:

- Finish or back out of the current atomic step. Never stop mid-edit in a
  known-broken state.
- The `## Checkpoint` records the branch and HEAD, what is uncommitted, what
  is verified (and how), and the first action on resume. That is the resume
  note; nothing else needs writing.
- Commit a `wip:` only when the user asked for a pause and only over changes
  this session authored. Never on compaction, never on "keep going", never
  over someone else's dirty files.
- On resume, the prior trail is authoritative for reasoning and completed
  investigation; inherited *completion claims* are re-verified on the real
  artifact before being relied on.

## Retention

Bank files are read every session, so their size is the framework's context
budget. Retention keeps the two volatile files bounded without losing history.

**Soft targets, per effective bank:**

- `activeContext.md`: at most 200 lines and 12,000 bytes.
- `progress.md`: `## Recent milestones` keeps its 10 newest entries.
- `.rules`: about 40 lines (existing rule; pruned, not rotated).

**Protected sections are never rotated** — they are current state, rewritten
in place by the normal update:

- activeContext: `## Current focus`, `## Checkpoint`, `## Next steps`,
  `## Open questions`, `## Notes for next session`. Move a note into
  `Recent changes` once it is done; age alone does not make it history.
- progress: `## Status`, `## What works`, `## In progress`,
  `## What's left to build`, `## Known issues`.

**Rotatable units**, oldest first (banks are newest-first, so oldest is last):

- activeContext: an entire `## Recent changes (<suffix>)` section — the
  heading travels with its body — or a top-level bullet (a line starting
  with `-`) with its continuation lines under an unsuffixed `## Recent changes`.
  The newest dated section is the exception: its heading stays live as the
  pointer's home and is *copied* into the archive (the helper reports it
  as `ARCHIVE-PREFIX`).
- progress: a top-level bullet under `## Recent milestones`.
- Any other structure is left untouched and reported as remaining overage.

**Rotation is lossless.** Selected units move verbatim to
`<effective bank>/archive/<file>-<YYYY-MM>.md` (append, or create with a
one-line header), and the live file keeps one pointer line,
`Older entries: archive/<file>-*.md`, directly under the first
`## Recent ...` heading. If protected content alone exceeds the target,
report it and rotate nothing further — never truncate current state.
Rotating again with nothing rotatable is a no-op.

**Mechanism.** `rotate-check.sh <file> <activeContext|progress>` (in this folder)
is read-only: it measures a file, lists rotatable units as line ranges,
predicts the result of rotating the fewest oldest units that meet the
target, and ends with `RESULT: NO-OP`, `RESULT: ROTATE n`, or
`RESULT: OVERAGE-REMAINS`. The `update-memory` workflow must run it
on the *proposed* files, fold any `ROTATE n` into the same confirmation as
the content diffs, and re-run it after writing. The approval gate is
unchanged — the helper selects, the user approves, the agent writes.

**Archives are not routine reads.** `archive/` is excluded from the session
read list, `start` and `update-memory`. Read an
archive only when the task needs that history.

