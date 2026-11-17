---
name: grill
description: "Interview the user relentlessly about a plan, decision, or idea until nothing is left silently assumed. Use when the user asks to be grilled, wants to stress-test a plan, decision, or idea, or says 'grill me'."
---

# Grill

Interview the user relentlessly until you both understand the plan the same
way. Every question comes with your recommended answer, and nothing is acted
on until the user says the understanding is shared.

Adapted from mattpocock/skills by Matt Pocock (MIT); see LICENSE-mattpocock.

Call it as `/grill` in Claude Code or `$grill` in Codex, with the thing to
grill. For example:

```text
grill me on this migration plan
grill the decision to drop offline mode
```

## Trigger

Use it when the user asks to be grilled, wants a plan, decision, or idea
stress-tested, or says "grill me".

Do not use it to produce a plan (`breakdown` does that) or to review a diff
(`review`).

## Required reads

1. The plan, decision, or idea, as the user stated it.
2. `decisionLog.md`, `activeContext.md`, and `.rules` in the memory bank,
   when the project has one, so you do not reopen what is already decided
   without saying so.
3. Whatever in the project answers a factual question (see Facts).

## Allowed writes

None. This workflow is read-only on the project and the memory bank.

## Workflow

Map the subject as a **design tree**: every decision branches into the
decisions that hang off it.

Work the tree in **rounds**. The **frontier** is every decision whose
prerequisites are already settled: the questions you can ask now without
guessing at answers you have not heard yet. Ask the whole frontier in one
round: number each question and give your recommended answer. Then wait for
the user's answers before the next round.

Format a round like this:

```text
Q1 - <question title>: <question body, which may run to several paragraphs and include choices>

Recommended: <your recommended answer>

---

Q2 - <question title>: <question body>

Recommended: <your recommended answer>
```

Word each question so that "yes" accepts your recommended answer.

Each round of answers reshapes the tree: settled decisions push the frontier
outward and unblock the questions that depended on them. Work out the new
frontier and ask the next round. A question whose answer depends on another
question still open in this round belongs to a later round, not this one.

### Facts

Finding facts is your job, never the user's. When a frontier question needs
a fact from the environment (the filesystem, a tool, the memory bank), look
it up; when your harness has sub-agents, send one with a fresh context to
find it. Do not ask the user for anything you could look up yourself.

Do not block on a lookup. A lookup still running is an unsettled
prerequisite, so only the questions downstream of it wait; ask the rest of
the frontier now.

The decisions are the user's. Put each one to them and wait.

### Done

The grilling is done when the frontier is empty: every branch of the tree
visited, nothing left silently assumed. Give the summary in the Output
contract and ask the user to confirm that it is the shared understanding. Do
not act on it until they do.

### Recording decisions

After the user confirms, offer once to record the settled decisions through
the `update-memory` workflow
(`${CLAUDE_SKILL_DIR}/../update-memory/SKILL.md`). Decisions the user made
go to `decisionLog.md`, recorded as theirs. A recommendation they did not
accept is not a decision. Do not write any of it yourself; `update-memory`
shows the diff and waits for a yes.

## Output contract

Each round: the numbered questions in the format above, and nothing else
apart from facts you looked up that change a question.

At the end:

```text
GRILL: <subject>
SETTLED:
- <decision>: <the user's answer>
LOOKED UP:
- <fact>: <where it came from> (or: none)
STILL OPEN:
- <anything the user chose to leave undecided> (or: none)
```

Then ask: "Is this the shared understanding?" Once it is confirmed, ask:
"Record these decisions with update-memory?"

## Stop conditions

- After each round, stop and wait for the user's answers. Never answer a
  question for them, and never treat silence as a yes.
- If the subject is too vague to draw a tree from, ask what is being decided
  before the first round.
- If an answer contradicts an accepted entry in `decisionLog.md`, say so and
  ask whether the earlier decision is being replaced.
- If the user ends the grilling early, give the summary with what is still
  open, and do not act on it.
