# Tavo Stack

A project memory bank for Claude Code, with the workflows that read and update it.

An agent's memory resets between sessions. Tavo Stack keeps what matters in a small set of Markdown files in your repository, and gives Claude Code workflows to open a session from them and to update them, with diffs shown first, when the work is done.

## What you get

**In your project** (plain files you own and can read):

- `memory-bank/`: seven Markdown files covering the brief, product context, architecture, tech setup, decisions, current focus and progress.
- `.rules`: short project-specific lessons and preferences.
- `AGENTS.md`: tells the agent to read the bank at the start of every session.

**Workflows:**

| Workflow | What it does |
|---|---|
| `start` | Reads the bank, summarizes where things stand, and asks where to pick up. A blank bank routes to guided setup. |
| `update-memory` | Proposes updates to the bank from the session's work. Shows the diffs and waits before writing. |
| `discover` | Fills a blank bank from a rough idea, by asking a few questions. |
| `from-prd` | Fills a blank bank from an existing PRD, brief or spec. |
| `init-memory` | Fills a blank bank by reading an existing codebase. |

**More workflows** (always part of the plugin; optional groups when copied into a project):

| Group | Workflow | What it does |
|---|---|---|
| Planning and review | `breakdown` | Breaks a task into steps before any editing. |
| | `review` | Reviews the current branch or diff. |
| | `ship` | Runs a pre-merge readiness checklist and asks for proof the change works. |
| | `security-check` | A light security pass on the current change. |
| | `ask-codex` | Asks the Codex command-line tool for a second opinion. |
| Writing | `polish` | Rewrites prose so a reader gets the facts faster. Proposes; never writes. |
| Verification | `verify-map` | Records how to verify a feature by hand, and what was seen last time. |
| Pull requests | `file-pr` | Opens a pull request that starts with the problem. |
| | `babysit-pr` | Watches a pull request through checks and review, and fixes real findings. |
| | `modernize-pr` | Decides whether a stale pull request should be refreshed, reworked or closed. |
| | `pr-comment` | Rules for any comment posted on your behalf. |

The pull-request workflows need the GitHub command-line tool `gh`. `babysit-pr` includes the `pr-wait` script from [tstack](https://github.com/t3dotgg/tstack) (MIT, Theo Browne), unchanged.

## Two ways to use it

The same files work both ways. Pick one per project.

| | Plugin | Copied into the project |
|---|---|---|
| Install | Once per machine | Once per project |
| Updates | One update reaches every project | Per project, by running setup again |
| Works from a plain clone, with nothing installed | No | Yes |
| What you type | `/tavo:start` | `/start` |

### As a plugin

In Claude Code:

```text
/plugin install tavo --marketplace gusfeliciano/tavostack
```

The plugin brings the workflows only. In a project with no memory bank yet, run `/tavo:start`: it lists the blank files it would create (the bank, `.rules` and `AGENTS.md`) and waits for your yes.

### Copied into a project, no plugin

Open the project in Claude Code and paste this:

```text
Set up Tavo Stack in this project by copying its files in. Do not install a plugin.

Source: https://github.com/gusfeliciano/tavostack

If the source is a URL, clone it fresh into a temporary folder outside this project. If it is a folder, use it as it is and change nothing in it. Read SETUP.md in the source and follow it exactly. This project is the target.

Look first and change nothing. If this project already uses Serel Memory, stop and tell me; do not continue. Never replace, merge or edit an existing memory-bank/ file, .rules, AGENTS.md, CLAUDE.md or workflow: report those and leave them as they are. Do not run scripts from the source, edit .gitignore, commit or push. Show me every path you would create and whether Git will see it, and wait for my yes before writing anything.
```

The agent follows [SETUP.md](SETUP.md): it asks which optional groups you want, shows every file it would create, waits for your yes, and never replaces a file you already have. Then open a new session and run `/start`.

To update later, or to add a group, paste the same prompt again. Unchanged files are reported as the same, and changed ones are shown as differences for you to decide.

## Status

Early. Tested with the Claude Code command-line tool. Not yet tested in the Claude desktop app or in Codex.

Tavo Stack continues an earlier project, Serel Memory, and its companion Serel Kit, trimmed to the workflows that saw real use. A project already set up with Serel Memory is not handled by the setup above; a migration is planned.

One leftover from that history: `start` and `update-memory` run a project's `bin/serel-memory` drift checker when it exists. A new project reports `checker absent`.

## License

MIT. See [LICENSE](LICENSE).
