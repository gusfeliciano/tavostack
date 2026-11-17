---
name: codex-loop
description: "Review a code change with the Codex CLI in a bounded loop: Codex reviews read-only, you fix or push back with evidence, and the loop repeats until both agree or the budget ends. Use when the user says \"codex loop\", \"review loop\" or \"sol loop\", or asks to review a change with Codex in a loop until both agree."
---

# Codex Loop

Make a code change more correct before (or sometimes after) filing a pull
request, by having the Codex command-line tool review it in a loop. Codex
reviews read-only. You answer each finding with the smallest fix or with
evidence, and ask again until both of you agree or the budget ends.

Adapted from tstack by Theo Browne (MIT); see LICENSE-tstack.

Make the smallest possible changes to answer the reviewer. Watch for scope
creep, and do not be afraid to push back.

Call it as `/codex-loop` in Claude Code, optionally with a PR number. For
example:

```text
codex-loop
codex-loop 482              # review the change in PR 482
codex-loop, up to 5 rounds
```

## Trigger

Use it when the user says "codex loop", "review loop" or "sol loop", or asks
to review a change with Codex in a loop until both agree. `full-send` and
`full-stop` also use it for their review step.

Do not use it for a single second opinion on a plan or a decision
(`${CLAUDE_SKILL_DIR}/../ask-codex/SKILL.md`), or for a one-pass code review
(`${CLAUDE_SKILL_DIR}/../review/SKILL.md`). It is written for Claude Code,
where Codex is the other model. If you are running inside Codex, stop and say
so: a workflow never calls its own CLI.

## Required reads

1. The working tree, the base, the diff, and the check results, as step 1
   of the workflow describes.
2. "Canonical invocation" in
   `${CLAUDE_SKILL_DIR}/../_shared/cross-agent-review.md`, for how to run
   Codex. This workflow differs from the rest of that file in three ways:
   its budget is 3 rounds, or 5 in the cases below; you edit files between
   rounds; and a missing Codex stops the loop instead of falling back to a
   local critique.

## Allowed writes

- Edits to the files in scope, between rounds only, to fix findings you
  agree with. Never while a round is running.
- One temporary prompt file per round, outside the repository, removed after
  the round.
- No commit and no push during the loop (step 5).
- Never `memory-bank/`, `.rules` or `AGENTS.md`. A finding about one of them
  is answered with a proposed diff the user approves, through `update-memory`
  (`${CLAUDE_SKILL_DIR}/../update-memory/SKILL.md`). This holds when
  `full-send` or `full-stop` runs the loop too.

Codex itself writes nothing: every round runs in a read-only sandbox.

## Budget

- Default: 3 rounds in total. Round 1 is a full review. Later rounds check
  only changed code and open findings.
- Extend to 5 only for security, permissions, auth, data loss, or
  state/concurrency, while a blocker or should stays open with evidence. Or
  when the user asks.
- Stop early if a round returns only nits. Declined nits, and fixes that
  touch only comments, docs, or formatting, need no new round.
- One reviewer only. Codex must not start subagents or other reviewers, and
  you never run two rounds at once.
- Keep a findings ledger: id, severity, outcome, and usage if Codex printed
  it. Never guess a value.

## Workflow

### 1. Scope

Run `git status --short`. Do not stash, discard, or include unrelated
changes. Limit the review to the paths of this change if they separate
cleanly. Otherwise stop and ask.

Find the base after `git fetch origin`:

- For a PR: `gh pr view <n> --json baseRefName,headRefOid`. Check out the
  branch only if the working tree is clean. Confirm `HEAD` is the PR head, or
  ahead of it only by this change. The base is
  `git merge-base HEAD origin/<baseRefName>`.
- For local work: `git merge-base HEAD origin/<default-branch>`. With no
  remote, use a base the user names, or `HEAD` when only uncommitted changes
  are under review.

The change is everything from the base to the working tree in those paths,
including untracked files. Use one diff command for all rounds. State the
intent and what is out of scope in one or two sentences. Run the checks that
cover the change. Do not send code that this change breaks. Note missing
checks, and checks that already failed before the change, in the brief.

### 2. Run Codex

Check the tool first:

```bash
codex --version
codex login status
```

If Codex is missing, not signed in, or blocked, stop and report it. Never
substitute another reviewer, a subagent, or your own critique, and never
describe the change as reviewed.

Each round is a new Codex run with no memory of earlier rounds, so every
brief must be complete without them. Do not put secrets, credentials, or
private data in a brief. List the untracked files in scope by path, after
checking that none is a secret, an environment file, or a data export; an
untracked file that is one of those is out of scope, and you say so. Write the brief to a temporary file and run Codex in
one shell invocation, read-only, with whatever model Codex is configured to
use:

```bash
PROMPT_FILE="$(mktemp)"
cat > "$PROMPT_FILE" <<'EOF'
...the full brief...
EOF
codex exec --cd "$PWD" --sandbox read-only - < "$PROMPT_FILE"
rm -f "$PROMPT_FILE"
```

Do not add model or effort flags. Give the command a timeout long enough for
a careful review; a long run is usually thoroughness, not a hang. Do not edit
files in scope while a round runs, and do not start a second reviewer. A
timeout, an error, or an empty reply is not approval. If a round fails, make
sure its process has stopped, then run the round again with the same brief
and ledger. A failed run does not use up a round.

Round 1 brief:

```text
Review only. Do not edit, commit, push, comment on PRs, or start subagents.
Repo: <abs path> Base: <sha> Intent: <intent> Out of scope: <scope> Checks: <results>
See: git diff <base> [-- <paths>]. New files not yet in Git: <explicit list of paths, or "none">.
Read only what the review needs. Do not open .env files, credentials, keys,
private data exports, or files outside the paths above unless a finding
depends on a caller there.
Read the surrounding code. Find bugs, broken callers, and mismatches with
the intent. Skip style unless it hides a bug. Hold the change to the smallest
one that works. Flag code, tests, or guards the intent does not need. Other
paths, rare cases, small performance costs, extra compatibility: nits.
blocker = the intended case is wrong. should = a real bug callers will hit,
within the intent. nit = everything else.
Reply:
VERDICT: APPROVE or CHANGES
F1 [blocker|should|nit] <file>:<line> <claim>. Fix: <suggestion> (stable ids)
```

### 3. Answer findings

Check each finding and its severity yourself, against the current code.
Another path, a rare case, or a small cost is a nit, whatever label it came
with. The exception is security, authorization, and data loss: a real defect
there keeps its severity even on a rare path, as
`${CLAUDE_SKILL_DIR}/../_shared/cross-agent-review.md` grades it, and is
never declined as a nit.

- **Agree:** make the smallest fix. Prefer deleting or narrowing code over a
  new lock, cache, helper, or option. Fix the case that happened. Add one
  focused test that fails without the fix, if the intent needs it.
- **Disagree:** say why, with evidence (file:line, a test, command output).
- **Out of scope or a nit:** decline in one line and list it as a follow-up.

Update the ledger. Each finding keeps its id (F1, F2, ...) in every round,
and a new finding takes the next number. Re-run the checks and
`git diff --shortstat <base>`. If the review grew the diff, cut first. Never
agree just to end the loop.

### 4. Next round

Start another round only if budget remains and one of these is true: a
blocker or should is open, code that changes behavior was edited, or the last
approval does not cover the current diff.

Later-round brief, in this order:

```text
<the round 1 brief, in full, with current check results>
This is round <n>. You have no memory of earlier rounds; the ledger below is
the record.
LEDGER:
F1 [severity] fixed: <what changed, file:line>
F2 [severity] disagree: <evidence>
F3 [severity] declined: <reason>
CHANGED SINCE LAST ROUND: <files and a one-line summary of each edit>
Review open ids and changed code only. Mark each earlier id
`resolved` or `still open: <reason>` before adding new findings. New findings
continue the numbering.
```

### 5. Stop and ship

- **Agreed:** Codex returned APPROVE on the current diff, no blocker or
  should is open, and you have no concern of your own.
- **Stuck:** an item is still disputed after two replies, or the budget
  ended. Show both sides in two or three lines each. The user decides. The
  end of the budget is not approval.

Do not push during the loop. Push after agreement only if the user asked to
push, ship, or update the PR. If you push to a PR, add this line to its
description, above the attribution line:

`Reviewed with codex-loop: <N> rounds with Codex. Codex verdict: approved`

Name the model or effort only if Codex's own output showed it.

## Output contract

```text
CODEX-LOOP: <agreed | stuck | stopped early on nits | blocked> (<n> of <budget> rounds)
REVIEWER: Codex CLI, read-only (model: <as printed by Codex | not shown>)
BASE: <sha>; diff before: <shortstat>; after: <shortstat>
CHANGED BY REVIEW: <what the review changed in the code> | nothing
FIXED: <id [severity] finding, fix> | none
DISAGREED: <id [severity] finding, evidence, resolved or still open> | none
DECLINED: <id [severity] finding, reason, listed as a follow-up> | none
USAGE: <as printed by Codex | not shown>
PUSHED: <yes, <ref> | no>
```

When the result is `stuck`, follow it with both sides of each open item.

## Stop conditions

Stop and report, without starting a round, when:

- Codex is missing, not signed in, or blocked. Say which, and that no review
  happened.
- You are running inside Codex.

Stop and ask when:

- The change does not separate cleanly from unrelated changes, or no base
  can be found.
- The loop is stuck (step 5), or a finding needs a product, scope, or
  architecture decision.
- A round fails twice in a row. Report the error.
