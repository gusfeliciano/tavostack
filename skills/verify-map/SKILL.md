---
name: verify-map
description: "Create or refresh a feature's verification map in the memory bank — how to see the feature work by hand, and what someone actually saw the last time they looked. Use when the user asks for a verification map, a manual test recipe, a feature map, or how to prove a feature works."
---

# Verify map

Write down how to see one feature work with your own eyes, and what happened
the last time someone did. The result is one file in the memory bank that the
next session can follow.

Call it as `/verify-map <feature>` in Claude Code or `$verify-map <feature>`
in Codex. Usage:

```text
verify-map checkout
verify-map "reset a forgotten password"
verify-map                              # lists existing maps and asks
```

## Trigger

Use it when a feature has no scripted way to prove it behaves — a UI flow, a
job that has to actually run, an integration you can only watch. Use it again
after verifying by hand, to record what you saw.

Do not use it as a substitute for a test. If the behavior can be asserted in
code, write the test instead. A map is for what a test cannot reach.

## Required reads

1. The memory bank is `memory-bank/` at the project root. Everything else is
   relative to it.
2. `${CLAUDE_SKILL_DIR}/TEMPLATE.md` — the file shape. Read it every
   time; do not reproduce it from memory.
3. The existing `memory-bank/verification/<feature>.md`, if there is one.
   A refresh edits that file; it does not start over.
4. `activeContext.md` and `progress.md` in the memory bank, for what the
   feature is and whether it currently works.
5. The code paths the feature lives in, enough to write a Launch section that
   runs. Read the project README and any run scripts before inventing a
   command, so the Launch section contains commands that exist.

## Stop conditions

Stop before writing anything when:

- **There is no memory bank, or it is uninitialized** — `memory-bank/` is
  missing, or its core files are missing, empty, or still only template
  placeholders. Say which files, and ask the user to seed the bank first:
  `discover` if there is no code yet, `init-memory` if there is, `from-prd` if
  a spec exists. Do not create the bank yourself.
- **No feature was named.** List the maps already in
  `memory-bank/verification/`, suggest candidates from the bank, and ask
  which one. Never pick for the user.
- **You cannot tell how to launch the thing, so the launch procedure would be
  a guess.** Ask. A Launch section with a guessed command is worse than no
  map, because the next person will trust it.
- **The user is asking you to record a run you did not observe.** Say what you
  can and cannot attest to, then ask them for the observation.

## Allowed writes

Exactly one file: `memory-bank/verification/<feature>.md`.

Nothing else. Not `activeContext.md`, not `progress.md`, not `.rules`. If the map surfaces something those files should say, mention it and
let `update-memory` handle it.

The memory bank's rule applies: show the diff and wait for confirmation before
writing. A new file is shown as an **added-file diff** — the
`--- /dev/null` / `+++ b/<path>` header and every line prefixed with `+` — not
as a bare document body. The proposal then reads the same way whether the file
is new or being refreshed, and there is no way to mistake it for a file that
already exists.

## Workflow

1. Run the stop conditions.
2. Pick the filename: lowercase, hyphens, no spaces —
   `reset a forgotten password` becomes `reset-password.md`. Keep the human
   name inside the file as the heading.
3. Read the template (`${CLAUDE_SKILL_DIR}/TEMPLATE.md`), then the
   existing map if there is one.
4. Fill in the **Recipe**: Launch, Drive, Observe, Clean up. Every command
   must be one you found in the repo or one the user gave you. Every input
   must be concrete.
5. Fill in **Records** only from what was actually observed. If this run is
   documentation only, leave the records as they are and say the feature is
   unverified.
6. Refresh, don't rewrite: keep older records verbatim, add the new one on
   top, and update Status and Paths if they changed.
7. Show the diff — added-file form when the file is new — and ask for
   confirmation.
8. Write only after the user confirms. Then print the path and stop.

## Output contract

```text
VERIFY-MAP: <feature> — <new | refreshed>
BANK: memory-bank/
FILE: memory-bank/verification/<feature>.md

<the diff: an added-file diff when the file is new, a unified diff on refresh>

RECORDS: <n> (newest <date>) | none — this feature is unverified
UNVERIFIED CLAIMS: <anything in the Recipe you could not confirm yourself>
```

Then ask: "Write this file?" and stop until answered.

## What this is not

- A map is not a test. It never replaces running the tests, and a green map
  is not a green build.
- A missing map is not a product failure. It means nobody has written down how
  to check that feature yet.
- An empty Records section is not a bug. It is an honest "nobody has verified
  this."
