---
name: retro
description: "Run a retrospective on a coding session and propose improvements to the agent's environment (instructions, rules, checks, tooling), not to the code."
disable-model-invocation: true
---

# Retro

Look back over a coding session and propose changes to the agent's
**environment** that would make the next run go better: its instructions,
its rules, the repository's automated checks, and its tools. The code itself
is not the subject.

Adapted from mattpocock/skills by Matt Pocock (MIT); see LICENSE-mattpocock.

Call it as `/retro` in Claude Code or `$retro` in Codex. With no session
named, it looks at the current one.

## Trigger

Only when the user starts it. Use it after a session that took longer than
it should have, repeated a mistake, or needed a correction from the user.

Do not use it to record what was done (`update-memory` does that) or to
review the code (`review`).

## Required reads

1. The session itself: the current conversation, or the session the user
   names. If the user names an earlier session and you cannot find its log,
   ask where it is.
2. The steering files: `AGENTS.md`, `CLAUDE.md` if present, and `.rules`.
3. The memory bank, as far as the session touched it: `systemPatterns.md`
   and `techContext.md` for what the agent should have been able to find,
   and any optional doc on the subject.
4. The repository's own checks: the lint, check, and test commands in its
   build files, its pre-commit hooks, and its CI configuration. Read these
   before proposing a check, so a check that exists but is not wired up or
   is silently broken is the finding, not a second copy of it.

## Allowed writes

None by default. The retro proposes.

When the user accepts a proposal that changes `.rules`, `AGENTS.md`,
`CLAUDE.md`, or a memory bank file, show that change as a diff and wait for
a yes before writing it. Memory bank and `.rules` changes follow the rules
of the `update-memory` workflow
(`${CLAUDE_SKILL_DIR}/../update-memory/SKILL.md`). A new check, hook, or CI
step is a code change: describe it, and build it only when the user asks.

## Workflow

1. Read the session and the files in Required reads.
2. Look for candidates in these categories:

   - **Navigation**: how easily did the agent find the right files? Are
     there hidden dependencies between files? Would a pointer in
     `AGENTS.md` or `systemPatterns.md` have made it easier? *Use when* the
     session took a long time to find a piece of information.
   - **Automated checks**: could a lint rule, a type check, a test, or a
     file-layout check have caught an error the agent made? A repository
     with no guardrail at all (no pre-commit hook and no CI job running its
     lint, type check, or tests) is itself a finding. *Use when* the agent
     made a mistake a check could have caught, or the repository has no
     guardrail.
   - **Coding standards**: should a rule be added for review to enforce, or
     an existing one removed or made clearer? Classify the violation first.
     A **mechanical** one (a fixed pattern, a banned API, an import shape, a
     file-location rule) gets a deterministic check: a rule in the
     repository's own linter, a pre-commit hook, or a CI step, whichever
     its existing setup makes cheapest. Default to the check over the
     written rule. Keep `.rules` for real **judgement calls**, each with a
     concrete failure example. *Use when* a review missed a mistake.
   - **Instructions that should become checks**: are there steering
     instructions in `AGENTS.md`, `CLAUDE.md`, or `.rules` that belong in
     an automated check instead? *Use when*
     a steering file is large, in the repository or in the user's global
     scope.
   - **Tool economy**: did the agent make expensive tool calls that could
     be made cheaper? Is any custom tooling (a CLI, an MCP server) wasteful
     with context? *Use when* the agent made an expensive tool call.
   - **No-ops**: instructions in the steering files that do not change what
     the agent does. *Use when* the steering files are large and unwieldy.
   - **Information access**: could the agent be given information it
     lacked, such as dev server logs written to a file or read-only access
     to a third-party service? *Use when* a crucial piece of information
     was not available to the agent.

3. For each candidate, write down the evidence from the session and the
   smallest change that would have prevented it.
4. Present the candidates in order of severity, most costly first, in the
   Output contract's shape. Then stop and ask which to take further.
5. For each one the user accepts, follow Allowed writes.

## Reference

### Implementation and review

Work goes through two stages. The implementing agent explores, writes code,
and debugs, so its context is under the most pressure. The reviewing agent
receives a diff and needs little exploration, so its context is under the
least. Put the enforcement of standards on review, not on implementation,
and keep what the implementing agent must always read short.

### Where things belong

- `AGENTS.md` and `CLAUDE.md` are loaded into every session in the
  repository. Use them sparingly, mostly for pointers to other files.
- `.rules` holds short lessons and is also read every session. Keep it to
  about 40 lines; prune it, and promote a settled convention to
  `systemPatterns.md`.
- The memory bank holds intent, architecture, and status. Look for an
  existing home before proposing a new file; a topic that outgrows the core
  files gets an optional doc that is read only when relevant.
- A check that runs is better than a sentence that asks. When a correction
  recurs, propose the test, hook, or CI step, and keep the written rule
  until the check exists and works.

### Writing proposed instructions

- Delete instructions that do not change behaviour.
- State the completion criterion: say what done looks like.
- Phrase positively: say what to do, not only what to avoid.
- Do not restate what the repository already shows.

## Output contract

```text
RETRO: <session> (<n> candidates)

1. [<category>] <one-line finding>
   EVIDENCE: <what happened in the session>
   PROPOSAL: <the smallest change: file and wording, or the check to add>
   COST IF LEFT: <what the next session loses>
2. ...

NOTHING FOUND IN: <categories with no candidate>
```

Then ask: "Which of these should I take further?" Write nothing until the
user answers.

## Stop conditions

Stop and ask when:

- The session the user named cannot be found or read.
- A proposal would change `.rules`, `AGENTS.md`, `CLAUDE.md`, or the memory
  bank: show the diff and wait for a yes.
- A proposal would add or change a check, a hook, a CI step, or a
  dependency: describe it and wait to be asked.
- The session shows a problem in the code, not in the environment. Mention
  it in one line and leave it out of the candidates.
