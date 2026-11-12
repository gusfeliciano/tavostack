# Set up Tavo Stack in a project without the plugin

These steps copy Tavo Stack's files into one project. They are written for an agent (Claude Code) working in that project, with this folder available as the **source**. The project is the **target**. Nothing here installs a plugin, runs a script from the source, commits, or pushes.

Codex is not covered yet: the copied workflows are tested in Claude Code only.

## Step 1. Look first, change nothing

In the target, check:

- Git status, and whether the target is a Git repository at all.
- Whether any of these exist: `.serel-memory.json`, `.basecamp.json`, `.serel-kit.json`, `bin/serel-memory`.
- Whether a command file exists with the name of a workflow this setup would bring: `.claude/commands/<name>.md`, for every workflow in the core set and in each group the user chose (Step 3).
- Each path in the tables in Step 3.
- Whether `.claude`, `.claude/skills`, `memory-bank`, or any other folder on the way to a target path is a symbolic link, and whether every target path, fully resolved, stays inside the project.
- Instruction files: `AGENTS.md`, and any `CLAUDE.md`, `.claude/CLAUDE.md` or `CLAUDE.local.md`, in the project or in a folder above it.

## Step 2. Stop if the project already uses Serel Memory

If any file from the second bullet above exists, **stop and write nothing.** Say that this project was set up with Serel Memory, that moving it to Tavo Stack is a separate migration, and that copying these files in would change which workflow `/start` runs. Do not offer to continue.

If any command file from the third bullet exists, also **stop and write nothing**, whoever wrote it. Claude Code runs a workflow folder in place of a command file of the same name, so creating the folder would silently replace what that command does. Name the files and say the user decides what happens to them; setup deletes and disables nothing.

If any folder on the way to a target path is a symbolic link, or any target path resolves outside the project, **stop and write nothing**. Say which path, and that setup only writes inside the project itself.

## Step 3. Build the plan

**Choose what to copy.** The core set is always copied. The four groups below are optional: ask the user once which they want, unless they already said. If they have no preference, copy the core set only. A group whose folders already exist in the target counts as chosen.

| Group | Workflows | Needs |
|---|---|---|
| Planning and review | `breakdown`, `review`, `ship`, `security-check`, `ask-codex` | `ask-codex` needs the Codex command-line tool |
| Writing | `polish` | nothing |
| Verification | `verify-map` | an initialized memory bank |
| Pull requests | `file-pr`, `babysit-pr`, `modernize-pr`, `pr-comment` | the GitHub command-line tool `gh`; `babysit-pr` also needs `python3` |

A group is copied whole or not at all, because its workflows call each other.

**Core set.** Compare each source path with its target path.

| Source (in this folder) | Target (in the project) |
|---|---|
| `skills/start/` | `.claude/skills/start/` |
| `skills/update-memory/` | `.claude/skills/update-memory/` |
| `skills/discover/` | `.claude/skills/discover/` |
| `skills/from-prd/` | `.claude/skills/from-prd/` |
| `skills/init-memory/` | `.claude/skills/init-memory/` |
| `skills/_shared/` | `.claude/skills/_shared/` |
| `skills/_shared/templates/memory-bank/<each of the seven files>` | `memory-bank/<same name>` |
| `skills/_shared/templates/dot-rules` | `.rules` |
| `skills/_shared/templates/AGENTS.md` | `AGENTS.md` |

**Chosen groups.** Each workflow in a chosen group adds one row: `skills/<name>/` to `.claude/skills/<name>/`, copied with everything inside it.

Give every target path one status:

- **CREATE**: the target does not exist.
- **SAME**: the target exists and is identical. Nothing to do.
- **KEEP**: the target exists and differs. It is left exactly as it is.

Rules that have no exceptions:

- A file under `memory-bank/`, `.rules`, or `AGENTS.md` that already exists is always **KEEP** or **SAME**. It is never replaced, merged, or edited by setup.
- A workflow folder under `.claude/skills/` that already exists and differs is **KEEP**. Show what differs (`diff -r`), and say the user can replace it themselves after reviewing; setup does not.
- Each set is created whole or not at all. The core set is one set, and each group is one set. If any workflow folder in a set is **KEEP**, no workflow folder of that set is created: its workflows call each other, and a mixed set could call a file that is not there. Report the set as blocked, show what differs, and let the user decide. Other sets are not affected.
- The workflows depend on `.claude/skills/_shared/`. If that folder exists and differs, it is **KEEP**, and then no workflow folder is created either: say that the existing `_shared` folder does not match this version, show what differs, and stop for the user to decide. A workflow is only ever created next to a `_shared` folder that is **CREATE** or **SAME**.
- Also compare by purpose, not only by name: if the project has its own session opener or memory updater under another name (in `.claude/skills/`, `.claude/commands/` or `.agents/skills/`), report it in the plan. Setup deletes and disables nothing.

Instruction files:

- If `AGENTS.md` exists, it is **KEEP**. Show the "Available workflows" section of the template and suggest the user add it themselves if they want it.
- If a `CLAUDE.md`, `.claude/CLAUDE.md` or `CLAUDE.local.md` exists, in the project or in a folder above it, Claude Code reads that file and may not read `AGENTS.md`. Leave it as it is. For one inside the project, propose in the plan one added line for the user to approve separately: `Read AGENTS.md at the start of every session.` Show it as a diff. For one above the project, only tell the user and show the line; never edit a file outside the project. Never create a `CLAUDE.md`.

## Step 4. Show the plan and wait

Show one list: every target path, its status, and whether Git will see it (tracked, untracked, or ignored). Then ask: **"Create these files?"** and wait. Write nothing before a clear yes.

If every path is **SAME** or **KEEP**, say there is nothing to create and stop.

## Step 5. Create only what was approved

- Copy each **CREATE** path. Create parent folders as needed. Keep `rotate-check.sh` and `babysit-pr/scripts/pr-wait` executable, and do not edit `pr-wait`.
- Touch nothing else. Do not edit `.gitignore`, Git settings, or any existing file, unless the user approved the one-line `CLAUDE.md` change in Step 3.
- Do not commit or push.

## Step 6. Report and hand over

List what was created. Then say:

- Claude Code finds new workflows when a session starts, so **open a new session** in this project.
- In the new session, run `/start`. A blank bank routes to guided setup, which picks the right seed workflow (`discover`, `from-prd` or `init-memory`) and shows its proposals before writing anything.
- The other workflows are called the same way, by name: `/review`, `/polish` and so on.
- To update later, or to add a group, run the same setup prompt again: unchanged files report **SAME**, and changed ones are shown as differences for the user to decide.
