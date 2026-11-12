---
name: init-memory
description: "Initialize memory-bank files by analyzing an existing codebase. Use when the memory bank has been added to a repo with code already present and the memory-bank files are blank, template-only, or partly filled."
---

# Init Memory

Use this when Tavo Stack has just been added to an existing project and the memory bank is still uninitialized: the files are empty or template-only. You'll read the codebase and propose what each file should contain — the user reviews and edits before anything gets written.

**Existing bank content:** check each core file in the memory bank. If every one has real content beyond the template placeholders, the bank is initialized: do not re-seed it; point to `start` to resume or `update-memory` to correct it. If only some do (a partial bank, such as a `projectbrief.md` the user wrote), keep those files exactly as they are, read them first as given, and propose only the files that are missing, empty, or template-only; where the code disagrees with an existing file, say so instead of rewriting it.

**Setup context:** if setup already ran in this session (start's setup mode), treat its answers as given (stage, spec path, users) and ask only about what is still missing. The proposal and approval steps below are unchanged.

## Workflow

1. Read the repo structure. Map the top-level layout with fast shell tools (for example `ls` and `find`). Skip `node_modules/`, `.git/`, `dist/`, and `build/`.
2. Read the existing `README.md` and anything in `docs/` for stated purpose, setup, and architecture.
3. Read the package manifest (`package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `Gemfile`, etc.) for stack, dependencies, and scripts.
4. Skim the source tree to understand component boundaries. Don't read every file — sample enough to see the architecture.
5. Run `git log --oneline -20` for recent history and `git branch -a` to see active branches.
6. If the user named a PRD or spec, or setup found one, read it after the code. The code, tests, and docs about current behavior say what exists; the spec supplies intent and planned scope. Put spec-only items in the brief and product context, and in activeContext/progress as planned or not built yet, citing the spec; never record a spec requirement as working without evidence in the code.
7. Propose contents for each memory bank file (in a partial bank, only the files still missing, empty, or template-only), in this order:
   - **projectbrief.md** — what this project is, why it exists, success criteria. Infer from README, repo description, and code.
   - **productContext.md** — user problem and UX goals. Often the hardest to infer from code alone; flag what needs user input.
   - **systemPatterns.md** — architecture, key decisions, patterns in use. Visible from code structure and any architecture docs.
   - **techContext.md** — stack, dependencies, dev setup. Read directly from manifests and config files.
   - **decisionLog.md** — durable decisions already visible in docs, ADRs, architecture notes, or commits. If none are evident, say that no durable decisions have been recorded yet.
   - **activeContext.md** — current focus. Infer from recent commits, active branches, and open work in progress.
   - **progress.md** — what works, what's broken, status. Infer from existing tests, TODOs, open issues if visible.

For each file, present your proposal in this format:

```text
FILE: memory-bank/<name>.md
PROPOSAL:
<the proposed contents, preserving the file's heading structure (including the template's headings)>

CONFIDENCE: high / medium / low
REASONING: <one or two lines on what you based this on>
QUESTIONS FOR USER: <anything you genuinely couldn't infer — leave blank if none>
```

After all proposals, **stop and wait**. Do not write anything until the user reviews and approves (or revises).

Once the user confirms a proposal (with or without edits), write it to the file. Then move to the next.

## Rules

- If you can't infer something from code, docs, or history, say so explicitly. Don't fabricate to fill space.
- Prefer terse and accurate over comprehensive and speculative.
- Confidence should be honest: high only when you have direct evidence (README or code), low when guessing.
- If the project uses patterns or domain concepts you don't understand, flag them as questions. Don't pretend to understand.
- Treat code as the source of truth for current behavior.
- Treat the memory bank as the source of truth for intended behavior once initialized.
