---
name: start
description: "Start or resume a memory-bank project. Use: `start` (quick), `start full` (rich dashboard), or `start setup` (guided setup). Loads memory bank, summarizes state, asks where to pick up; routes a missing or blank bank to setup."
---

# Start

You are starting a new session on this project. Your memory has reset; the memory bank is your only continuity. Use this workflow to orient the session around the project memory bank.

Arguments the user gave with the call (Claude Code substitutes them here: [$ARGUMENTS]; in Codex they are in the user's message after the skill name).

**Mode:** Check those arguments for a mode keyword.

- If empty, `quick`, or `brief` → use **Quick mode** (compact output; both modes read the same inputs).
- If `full`, `onboard`, or `dashboard` → use **Full mode** (rich onboarding dashboard).
- If `setup` → use **Setup mode** (guided setup routing).

---

## Step 1 — Read the memory bank (both modes)

Do these in order, before anything else:

1. Read every file in `memory-bank/` in this order:
   - `projectbrief.md`
   - `productContext.md`
   - `systemPatterns.md`
   - `techContext.md`
   - `decisionLog.md`
   - `activeContext.md`
   - `progress.md`
2. Read `.rules`.
3. Look for optional docs under `memory-bank/` that clearly match the user's task or active context, and read only the relevant ones. Never read `memory-bank/archive/` at startup.
4. Run `git log --oneline -10` to see recent activity.
5. Run `git status` to see uncommitted changes.
6. If `bin/serel-memory` exists, run the read-only drift check and keep its summary line for the audit:

   ```bash
   "$(git rev-parse --show-toplevel)/bin/serel-memory" check
   ```

   Report its findings and summary line as they are — it never blocks the session. If the file is absent, say `checker absent`.

If any memory bank file is empty, missing, or still only template placeholders, surface it as `BLANK` or `UNINITIALIZED`. If the mode is `setup`, or the memory bank is missing or no core file has real content, use **Setup mode** instead of Step 2. Otherwise, when some files are blank (a partial bank), continue with Step 2 and offer to fill them: on yes, choose the seed workflow as Setup mode does; it proposes only the blank files and keeps the others exactly as they are.

---

## Step 2 — Produce the output

### Quick mode (default)

Produce a context audit and session summary in this exact format:

```text
CONTEXT AUDIT:
- Read: [memory-bank files and .rules]
- Optional docs read: [paths or "(none)"]
- Uninitialized: [missing, empty, or template-only files]
- Recent commits not reflected in memory: [yes/no/unknown]
- Working tree: [clean / dirty summary]
- Drift: [the drift checker's summary line, or "checker absent"]

PROJECT: [one sentence — what we're building]
PHASE: [from progress.md]
LAST SESSION: [from activeContext.md — what was being worked on]
CURRENT STATE: [what works / what's in progress / known issues — 2 lines max]
NEXT STEPS: [from activeContext.md — top 1–3]
OPEN QUESTIONS: [anything blocking or unresolved]
```

End with: **"Where do you want to pick up?"** Then wait.

### Full mode (`start full`)

Produce the full context audit (same as quick mode), then continue with all of the sections below.

#### Recent Progress (last 2–3 sessions)

Extract from `progress.md` recent milestones and `activeContext.md` recent changes:

- Session achievements with dates
- Key technical discoveries or architectural changes
- Milestones reached

#### Current State Analysis

**What Works**

- Pull from `progress.md` "What works" section — list each capability

**In Progress**

- Pull from `progress.md` "In progress" section
- Cross-reference with `activeContext.md` current focus

**Known Issues**

- Pull from `progress.md` "Known issues" section

**What's Left to Build**

- Pull from `progress.md` "What's left to build" section
- Include estimates where you can reasonably infer them

#### Open Decisions & Blockers

- Pull from `activeContext.md` open questions
- Pull from `decisionLog.md` any pending/draft decisions
- Flag any memory bank staleness (commits not reflected, files out of date)

#### Git Status

- Uncommitted changes that should be addressed
- Commits ahead/behind of remote
- Current branch info

#### Suggested Next Steps

Provide 3–5 specific, actionable recommendations organized by urgency:

🔴 **URGENT** — blocking progress or stale/broken
🟡 **HIGH-VALUE** — best use of this session based on momentum and impact
🟢 **STRATEGIC** — important but can wait

For each, include: clear action item, rough time estimate, expected outcome.

#### Direction Options

Present 3–4 options clearly:

- **Option A:** Continue recent momentum — [describe based on activeContext]
- **Option B:** Address top priority — [from what's left to build]
- **Option C:** Start next feature/initiative — [from what's left or open questions]
- **Option D:** Something else

End with: **"Which direction would you like to take for this session?"** Then wait.

### Setup mode (`start setup`, or a missing or blank bank)

Stay read-only until the user approves. An initialized bank is kept exactly as it is.

1. Orient: whether there is product code here (the memory bank, `.rules`, instruction files and anything under `.claude/` or `.agents/` do not count), a PRD or spec, which instruction files exist (`AGENTS.md`, `CLAUDE.md`), and the state of each bank file.
2. Ask only what you could not detect and the user has not already said: one short question per message, waiting for each answer, at most four in total.
3. Produce:

```text
SETUP PLAN:
- Bank: [missing / blank / partial / initialized]
- Stage: [idea / spec, no code / code / code + spec] (evidence: [paths])
- Instructions: [AGENTS.md and any Claude instruction files found]
- Blank files to create first: [none / the missing ones of memory-bank/*.md, .rules and AGENTS.md]
NEXT: [discover | from-prd <path> | init-memory | init-memory naming the spec] / resume
```

End with **"Continue with <next>?"** Then wait, unless the user has already told you to go ahead with this plan. If core bank files, `.rules` or `AGENTS.md` are missing, copy only the missing ones from `${CLAUDE_SKILL_DIR}/../_shared/templates/` (its `dot-rules` file becomes `.rules`) after approval and before seeding; never write over an existing file, and never create or edit a `CLAUDE.md`. If a `CLAUDE.md`, `.claude/CLAUDE.md` or `CLAUDE.local.md` exists, in the project or in a folder above it, say in the plan that Claude Code reads that file and may not read `AGENTS.md`, and show the one line the user can add to it themselves: `Read AGENTS.md at the start of every session.` A seed workflow runs in this session by following its own file, `${CLAUDE_SKILL_DIR}/../<workflow>/SKILL.md`, carrying the answers you already have. Its proposals and approval come before any bank write, and in a partial bank it fills only the blank files. For code plus a spec, the code describes what exists and the spec stays planned intent. An initialized bank is never re-seeded: end with **"Where do you want to pick up?"** instead.

---

Do not start working on anything until the user answers.
