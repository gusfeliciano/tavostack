---
name: diagnose
description: "Diagnosis loop for hard bugs and performance regressions: build one fast command that goes red on the bug before forming any theory, then minimise, hypothesise, instrument, fix, and clean up. Use when the user says 'diagnose' or 'debug this', or reports something broken, throwing, failing, or slow."
---

# Diagnose

A discipline for hard bugs. The central rule: build one fast, deterministic
command that goes red on this bug before you form any hypothesis. Skip a phase
only when you can say why.

Adapted from mattpocock/skills by Matt Pocock (MIT); see LICENSE-mattpocock.

Call it as `/diagnose` in Claude Code or `$diagnose` in Codex, with the
symptom. For example:

```text
diagnose the export button throws on large files
diagnose why the nightly import got three times slower
```

## Trigger

Use it for a bug or performance regression whose cause is not obvious: the
user says "diagnose" or "debug this", or reports something broken, throwing,
failing, or slow.

Do not use it for a failure whose cause is already known and only needs the
fix, or for building a feature.

## Required reads

1. The symptom in the user's own words: the error message, the wrong output,
   or the slow timing. Later phases check the fix against exactly this.
2. From the memory bank, when the project has one: `systemPatterns.md` and
   `techContext.md` for how the modules in the area fit together and how the
   project is run and tested, `progress.md` for known issues, and `.rules`
   for gotchas already learned. If there is no memory bank, skip these reads
   and the Record step; do not create one.
3. The project's own test and run commands (README, build scripts, CI
   configuration), so the loop is built from commands that exist.
4. Only then the code on the bug's path, and only as far as building the loop
   needs.

## Allowed writes

- A reproduction: a test, script, or throwaway harness.
- Temporary instrumentation, every line tagged (see Phase 4).
- The fix and its regression test.
- After a shown diff and a yes: one `progress.md` note and, at most, one
  `.rules` line (see Record).

Nothing else. No commits, no pushes, and no other memory bank files.

## Redact

This workflow has you show commands, outputs, and captured artifacts. Redact
every secret first: write `<REDACTED>` in its place. Build loops against
environment variables, so the credential stays in the environment and not in
what you show. Captured artifacts carry auth headers: quote only the lines
that carry the signal.

If the redacted output is not enough to diagnose the bug, say so and ask the
user.

## Phase 1: Build a feedback loop

**This is the workflow.** Everything else is mechanical. If you have a tight
pass/fail signal for the bug, one that goes red on this bug, you will find
the cause; bisection, hypothesis testing, and instrumentation all just
consume it. If you do not have one, no amount of staring at code will save
you.

Spend disproportionate effort here. Be aggressive. Be creative. Refuse to
give up.

### Ways to construct one, in roughly this order

1. **Failing test** at whatever seam reaches the bug: unit, integration, or
   end to end.
2. **Curl or HTTP script** against a running dev server.
3. **CLI invocation** with a fixture input, diffing stdout against a
   known-good snapshot.
4. **Headless browser script** (Playwright or Puppeteer) that drives the UI
   and asserts on the DOM, console, or network.
5. **Replay a captured trace.** Save a real network request, payload, or
   event log to disk and replay it through the code path in isolation.
6. **Throwaway harness.** Spin up a minimal subset of the system (one
   service, mocked dependencies) that exercises the bug's code path with a
   single function call.
7. **Property or fuzz loop.** If the bug is "sometimes wrong output", run
   1000 random inputs and look for the failure mode.
8. **Bisection harness.** If the bug appeared between two known states
   (commit, dataset, version), automate "boot at state X, check, repeat" so
   you can `git bisect run` it.
9. **Differential loop.** Run the same input through the old version and the
   new one (or two configurations) and diff the outputs.
10. **Human-in-the-loop script.** Last resort. If a human must click, drive
    them with a copy of `${CLAUDE_SKILL_DIR}/scripts/hitl-loop.template.sh`
    so the loop is still structured. Its captured output feeds back to you.

Build the right feedback loop and the bug is 90% fixed.

### Tighten the loop

Treat the loop as a product. Once you have a loop, tighten it:

- Can it be faster? Cache setup, skip unrelated init, narrow the test scope.
- Can the signal be sharper? Assert on the specific symptom, not "did not
  crash".
- Can it be more deterministic? Pin time, seed the random generator, isolate
  the filesystem, freeze the network.

A 30-second flaky loop is barely better than no loop. A 2-second
deterministic one is tight.

### Non-deterministic bugs

The goal is not a clean reproduction but a higher reproduction rate. Loop the
trigger 100 times, run it in parallel, add stress, narrow timing windows,
inject sleeps. A bug that fails half the time is debuggable; one that fails
1% of the time is not, so keep raising the rate until it is.

### When you cannot build a loop

Stop and say so plainly. List what you tried. Ask the user for one of:

- access to an environment that reproduces it,
- a redacted captured artifact (HAR file, log dump, core dump, or a screen
  recording with timestamps),
- permission to add temporary production instrumentation.

Do not go on to hypothesise without a loop.

### Completion criterion: a tight loop that goes red

Phase 1 is done when you can name **one command** (a script path, a test
invocation, a curl) that you have already run at least once, showing the
invocation and its redacted output, and that is:

- [ ] **Red-capable**: it drives the actual bug code path and asserts the
      user's exact symptom, so it goes red on this bug and green once fixed.
      "Runs without erroring" is not enough; it must be able to catch this
      specific bug.
- [ ] **Deterministic**: the same verdict every run. For flaky bugs, a
      pinned, high reproduction rate, as above.
- [ ] **Fast**: seconds, not minutes.
- [ ] **Agent-runnable**: you can run it unattended. A human takes part only
      through `${CLAUDE_SKILL_DIR}/scripts/hitl-loop.template.sh`.

If you catch yourself reading code to build a theory before this command
exists, stop: jumping straight to a hypothesis is the exact failure this
workflow prevents. No red-capable command, no Phase 2.

## Phase 2: Reproduce and minimise

Run the loop. Watch it go red as the bug appears.

Confirm:

- [ ] The loop produces the failure the **user** described, not a different
      failure that happens to be nearby. Wrong bug, wrong fix.
- [ ] The failure reproduces across several runs, or, for non-deterministic
      bugs, at a rate high enough to debug against.
- [ ] You have captured the exact symptom (error message, wrong output, slow
      timing) so later phases can check that the fix addresses it.

### Minimise

Once it is red, shrink the reproduction to the smallest scenario that still
goes red. Cut inputs, callers, configuration, data, and steps one at a time,
re-running the loop after each cut, and keep only what the failure needs.

A minimal reproduction leaves fewer moving parts to suspect in Phase 3 and
becomes the clean regression test in Phase 5.

Done when every remaining element is needed: removing any one of them makes
the loop go green.

Do not go on until you have reproduced **and** minimised.

## Phase 3: Hypothesise

Write **3 to 5 ranked hypotheses** before testing any of them. A single
hypothesis anchors you on the first plausible idea.

Each one must be **falsifiable**: state the prediction it makes.

> Format: "If <X> is the cause, then <changing Y> will make the bug disappear
> / <changing Z> will make it worse."

If you cannot state the prediction, the hypothesis is a hunch: discard it or
sharpen it.

**Show the ranked list to the user before testing.** They often know
something that re-ranks it at once ("we just deployed a change to number 3")
or have already ruled some out. Do not block on it; go on with your own
ranking if the user is away.

## Phase 4: Instrument

Each probe must map to one prediction from Phase 3. **Change one variable at
a time.**

Tool preference:

1. **Debugger or REPL inspection**, if the environment supports it. One
   breakpoint beats ten logs.
2. **Targeted logs** at the boundaries that tell the hypotheses apart.
3. Never "log everything and grep".

**Tag every debug log** with a unique prefix, for example `[DEBUG-a4f2]`.
Cleanup then becomes a single grep. Untagged logs survive; tagged logs get
removed.

**Performance regressions.** Logs are usually the wrong tool. Set a baseline
measurement first (a timing harness, a profiler, a query plan), then bisect.
Measure first, fix second.

## Phase 5: Fix and regression test

Write the regression test **before the fix**, but only if there is a
**correct seam** for it.

A correct seam is one where the test exercises the real bug pattern as it
happens at the call site. If the only available seam is too shallow (a
single-caller test when the bug needs several callers, or a unit test that
cannot reproduce the chain that triggered the bug), a regression test there
gives false confidence.

**If no correct seam exists, that is itself the finding.** Note it: the
structure of the code is preventing the bug from being locked down. Report
it in Cleanup.

If a correct seam exists:

1. Turn the minimised reproduction into a failing test at that seam.
2. Watch it fail. If you forced the red by changing code or a fixture, `diff`
   against a pristine copy to prove the change landed before you trust it.
3. Apply the fix.
4. Watch it pass.
5. Re-run the Phase 1 loop against the original, un-minimised scenario.

## Phase 6: Cleanup

Required before declaring done:

- [ ] The original reproduction no longer reproduces (re-run the Phase 1
      loop).
- [ ] The regression test passes, or the missing seam is reported.
- [ ] All `[DEBUG-...]` instrumentation is removed (`grep` the prefix).
- [ ] Throwaway prototypes are deleted, or moved to a clearly marked debug
      location.
- [ ] The hypothesis that turned out correct is stated in the report, ready
      for the commit or pull request message, so the next person learns
      from it.

## Record

When the project has a memory bank, propose recording what was learned.
Show each change as a diff and wait for a yes before writing:

- **`progress.md`**: a short note with the confirmed cause, the fix, and the
  reproduction command (redacted as above). If the bug is still open, put
  the note under known issues instead.
- **`.rules`**: one line, only if the lesson is reusable beyond this bug and
  clears the lesson bar in
  `${CLAUDE_SKILL_DIR}/../_shared/workflow-contract.md`. A regression test
  that already carries the lesson makes the line unnecessary.

The write rules are those of the `update-memory` workflow
(`${CLAUDE_SKILL_DIR}/../update-memory/SKILL.md`); follow them and do not
write around them. If the session changed more than this, say so and let
`update-memory` handle the rest.

## Output contract

```text
DIAGNOSE: <fixed | cause found, not fixed | blocked: no loop>
SYMPTOM: <the user's symptom, exactly>
LOOP: <the one command> (red before: <yes | no>, green after: <yes | no | not run>)
CAUSE: <the confirmed hypothesis and the prediction that held | not confirmed>
FIX: <files changed | none>
REGRESSION TEST: <path | no correct seam: <why>>
CLEANUP: <debug prefix removed, prototypes deleted | what remains>
RECORD: <proposed diffs for progress.md and .rules | nothing to record | no memory bank>
```

## Stop conditions

Stop and ask when:

- You cannot build a loop. List what you tried and ask for one of the three
  things under "When you cannot build a loop".
- Redacted output is not enough to diagnose the bug.
- The loop goes red on a failure other than the one the user described.
- Reproducing or instrumenting needs production access, real credentials,
  or a destructive action.
- The fix would change public behaviour, architecture, dependencies, or
  security posture beyond the bug itself.
- A `.rules` or `progress.md` change is proposed and not yet answered.
