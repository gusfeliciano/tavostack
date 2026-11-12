---
name: polish
description: "Rewrite the text you name to the project's house rules and show the diff. Proposes only; never writes anything. Use when the user asks to polish, tighten, clean up, de-slop, or plain-language a file, a section, a PR description, release notes, or pasted prose."
---

# Polish

Rewrite text so a reader gets the facts faster. This workflow proposes a
rewrite and stops. It does not write files.

Call it as `/polish <target>` in Claude Code or `$polish <target>` in Codex.
The target is a path, a path plus a heading, or text the user pastes. Usage:

```text
polish memory-bank/activeContext.md
polish docs/prd.md "## Goals"
polish            # then paste the text
```

## Trigger

Use it on prose someone has to act on: a memory bank entry, a PR description,
a README section, release notes, a handoff, a decision record, a commit body.

Do not use it on code, on generated files, or on text the user cannot change.
Do not use it as a review — it improves how a claim reads, not whether the
claim is true.

## Required reads

1. `${CLAUDE_SKILL_DIR}/RULES.md` — the house rules. Read the whole file
   on every run; do not work from memory of it.
2. The target the user named. A path, a path plus a heading, or pasted text.
   If they named a heading, read the whole file first so you can see what the
   section is allowed to assume.
3. If the target is a memory bank file, read "Retention" in
   `${CLAUDE_SKILL_DIR}/../_shared/workflow-contract.md`. It says which structure is load-bearing.

## When the target is memory bank content

`polish` works on any prose, with or without a memory bank. When the target
is bank content — a file under `memory-bank/`, `.rules`, or a bare name that
matches one of them — and the file is missing, empty, or still only template
placeholders, stop, name it, and point at `discover` (no code yet),
`init-memory` (code exists), or `from-prd` (a spec exists). Never create or
seed a bank.

Ordinary prose — a PR body, a README, release notes, pasted text — skips this
check. The rest of the workflow is identical either way.

## Allowed writes

None. This workflow writes nothing, ever. It ends after it shows the diff.

If the user wants the rewrite applied, that is a separate request they make
after reading it.

## Preserve

A rewrite that loses any of these is a failed rewrite, not a shorter one:

- Meaning, and every qualification and hedge that belongs to the author
  ("on macOS only", "we think", "not yet measured").
- Unresolved questions. An open question stays open and stays a question.
- Decision status and supersession links — `Accepted`, `Superseded by ADR-014`,
  dates, `result: pending`.
- Evidence pointers: file paths, line numbers, commit SHAs, ticket ids, links.
- Literal commands, paths, code, error strings, and anything inside a fenced
  code block. Fences are copied through untouched.
- Retention-significant structure in memory bank files: `## Recent changes`
  headings and their dated suffixes, `## Recent milestones`, the
  `Older entries: archive/...` pointer lines, and the order of entries.
  The retention step in `update-memory` reads these; changing them breaks rotation.
- Headings, list nesting, and anchors other files link to.

## Workflow

1. Resolve the target. If it is bank content, apply the check above first. If
   the name matches more than one file, stop and list the matches.
2. Read the rules, then the target.
3. Rewrite. Apply the rules in order; skip any rule that would cost a fact.
4. Diff your rewrite against the original.
5. Print the output below and stop.

## Output contract

Print exactly this shape, and nothing after it:

```text
POLISH: <target> — <n> change(s)

<a unified diff of the proposed rewrite>

WHY: <one line per change, keyed to a rule number>
KEPT: <one line per thing you deliberately left alone, and why>
```

Rules for the output:

- The diff is the proposal. Show real diff hunks with context, not a retyped
  copy of the file.
- `WHY` cites rule numbers from `RULES.md`, for example `3, 6 — split the
  compound sentence and cut "it is worth noting"`.
- `KEPT` is not optional. It is how the user checks you did not quietly drop a
  qualification. If you kept nothing notable, write `KEPT: nothing notable`.
- Close with one line: `Nothing was written.` Then stop. Do not offer to
  apply it, do not ask a follow-up question, do not summarize the file.

If the text already follows the rules, print `POLISH: <target> — 0 changes`,
one line saying why it is already fine, and stop. No change is a real answer.

## Stop conditions

Stop and ask, before producing any diff, when:

- The name the user gave matches more than one file. List the matches.
- The file does not exist, or the named heading is not in it.
- The text depends on facts you cannot check, and a faithful rewrite would
  force you to invent one. Quote the sentence and ask what it means.
- The rewrite would change what the text claims. That is an edit, not a
  polish, and it needs the author.
