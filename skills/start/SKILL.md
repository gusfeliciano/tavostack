---
name: start
description: "Start or resume a memory-bank project. Use: `start` (dashboard and full breakdown), `start quick` (short summary), or `start setup` (guided setup). Loads memory bank, shows the project state and what is waiting on the user, asks where to pick up; routes a missing or blank bank to setup."
---

# Start

You are starting a new session on this project. Your memory has reset; the memory bank is your only continuity. Use this workflow to orient the session around the project memory bank.

Arguments the user gave with the call (Claude Code substitutes them here: [$ARGUMENTS]; in Codex they are in the user's message after the skill name).

**Mode:** Check those arguments for a mode keyword.

- If empty, `full`, `onboard`, or `dashboard` → use **Full mode** (the dashboard and the full breakdown).
- If `quick` or `brief` → use **Quick mode** (compact output; both modes read the same inputs).
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

If any memory bank file is empty, missing, or still only template placeholders, surface it as `BLANK` or `UNINITIALIZED`. If the mode is `setup`, or the memory bank is missing or no core file has real content, use **Setup mode** instead of Step 2. Otherwise, when some files are blank (a partial bank), continue with Step 2 and offer to fill them: on yes, choose the seed workflow as Setup mode does; it proposes only the blank files and keeps the others exactly as they are.

---

## Step 2 — Produce the output

### Full mode (default)

Lead with the dashboard, so the state of the project can be read at a glance. Then the context audit, then the sections below, in this order. Keep every line short. Leave out a section that has nothing in it, apart from the dashboard and the audit.

#### Status Dashboard

```text
PROJECT: [one sentence — what we're building]
PHASE: [from progress.md]

IN PROGRESS:    [count] — [name the top 1–3]
NEXT UP:        [count] — [name the top 1–3]
WAITING ON YOU: [count] — [listed in full below]
LEFT TO BUILD:  [count]
KNOWN ISSUES:   [count]
```

Count from the bank: `progress.md` for in progress, left to build and known issues; `activeContext.md` for next steps; open questions and pending decisions for waiting on you. Never guess a count: when the bank does not list something, write `not tracked`.

If the project tracks work in GitHub Issues and `gh` is installed and signed in, add one line, `OPEN ISSUES: [count]`, from `gh issue list --state open`. If `gh` is missing or the call fails, leave the line out and carry on. Never restate issues in the bank or the bank in issues.

#### Context Audit

```text
CONTEXT AUDIT:
- Read: [memory-bank files and .rules]
- Optional docs read: [paths or "(none)"]
- Uninitialized: [missing, empty, or template-only files]
- Recent commits not reflected in memory: [yes/no/unknown]
- Working tree: [clean / dirty summary]
```

#### Waiting on You

List these first, before any suggestion, one line each with why it matters:

- Open questions in `activeContext.md` that only the user can answer
- Pending or draft decisions in `decisionLog.md`
- Steps only the user can take (an approval, a login, an account or a purchase)

Nothing an agent does can clear these, and they are easy to lose. If there are none, say "Nothing is waiting on you."

#### Recent Progress (last 2–3 sessions)

Extract from `progress.md` recent milestones and `activeContext.md` recent changes:

- Session achievements with dates
- Key technical discoveries or architectural changes
- Milestones reached

#### Current State

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

**Plans** (only if the bank holds plan documents)

- Sort them into active, ready to start, and blocked, and name the blocker
- Never call a plan ready, done or blocked from its status line alone. Check Git first (`git log --oneline --grep=<feature>`, or that the files it names exist): plan statuses go stale.

#### Git Status

- Current branch, and commits ahead of or behind the remote
- Uncommitted changes that should be addressed
- Memory bank staleness: commits not reflected, files out of date

#### Development Context

Two to four lines, only from what you read. Write `unknown` instead of guessing, and do not run the project's checks to find out:

- **Recent pattern:** how the last sessions were structured
- **Current focus:** the area getting attention
- **Checks:** whether tests and lint pass, as the bank last recorded it
- **Unreleased work:** anything finished but not shipped

#### Suggested Next Steps

Provide 3–5 specific, actionable recommendations organized by urgency:

🔴 **URGENT** — blocking progress or stale/broken, including anything a "Waiting on You" item is holding up
🟡 **HIGH-VALUE** — best use of this session based on momentum and impact
🟢 **STRATEGIC** — important but can wait

For each, include: clear action item, rough time estimate, expected outcome, and why now.

#### Direction Options

Present 3–4 options clearly:

- **Option A:** Continue recent momentum — [describe based on activeContext]
- **Option B:** Address top priority — [from what's left to build]
- **Option C:** Start next feature/initiative — [from what's left or open questions]
- **Option D:** Something else

End with: **"Which direction would you like to take for this session?"** Then wait.

### Quick mode (`start quick`)

Produce the context audit (the same block as full mode) and this summary, in this exact format:

```text
PROJECT: [one sentence — what we're building]
PHASE: [from progress.md]
LAST SESSION: [from activeContext.md — what was being worked on]
CURRENT STATE: [what works / what's in progress / known issues — 2 lines max]
NEXT STEPS: [from activeContext.md — top 1–3]
WAITING ON YOU: [open questions, pending decisions, steps only the user can take — or "nothing"]
```

End with: **"Where do you want to pick up?"** Then wait.

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
