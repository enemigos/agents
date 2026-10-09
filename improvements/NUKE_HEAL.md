# Nuke heal

## Problem

`skills/nuke/SKILL.md` step 7 says: if a defined check is red, apply the smallest change that makes it green. The dispatcher heals in both `changes` and `architecture` mode. Three risks follow.

1. **Scope creep.** The step heals every red check, including checks that were red before the change. A review of a 10-line diff can turn into a fix of unrelated code. The reviewer then reviews the heal, not the change.
2. **Weakened checks.** The smallest change that turns a check green is often to skip a test, delete an assertion, update a snapshot, add `// eslint-disable`, add `@ts-expect-error`, cast to `any`, or loosen `tsconfig`. Step 7 has no explicit protection against these.
3. **Edits during an audit.** `architecture` mode audits current code, but heal edits that code. The proposal below makes `architecture` mode read-only.

A related risk: when the external `thermo-nuclear-code-quality-review` skill is missing, the Complexity lane records `Cannot approve`. Nuke then can never approve on that machine.

## Fix

### 1. Heal only the failures the change caused

Compare individual failures, such as failing test names, lint rule and file pairs, or type errors with file and line. Exit codes alone are not enough. A suite can fail test A at the base and fail tests A and B now. B is a new failure.

Replace step 7 with:

```md
7. Run each defined check. A defined check is a test, lint, format-check, or type command the project already has. Record command names, exit codes, output text, and the list of individual failures as check evidence.
   In `changes` mode, when a check is red, run it again at `$BASE` in a temporary worktree (`git worktree add --detach <tmp> $BASE`). Install the same dependency versions and use the same environment. Remove the worktree after the run. Compare the two failure lists:
   - A failure in both lists, with no change to the code it covers: pre-existing. Do not heal it. Record it in Caveats.
   - A failure in the current list only, or in both lists when the change touched the code it covers: introduced. Heal it.
   When the baseline run cannot start or its failures cannot be compared, the failures are unconfirmed. Do not heal them. Report `Verdict: Cannot approve` with the check evidence in the section 3 order, then stop.
   After a heal, rerun the defined checks and compare again. Then refresh the `changes` diff and untracked files, so the lanes review the healed files.
   In `architecture` mode, do not heal. Record each red check as a Defects finding.
   Done: every introduced failure is healed, every pre-existing failure is recorded, and the diff is refreshed.
```

A pre-existing failure then does not block the review. A new failure inside a red check still blocks it. A failure that nuke cannot classify blocks the review and gets no heal.

### 2. Forbid heals that weaken a check

Add after step 7:

```md
A heal changes product code only. A heal must not:
- skip, focus, or delete a test, or remove or loosen an assertion
- update a snapshot or a golden file
- add a lint, format, or type suppression (`eslint-disable`, `@ts-ignore`, `@ts-expect-error`, `//nolint`)
- add `any`, an unchecked cast, or a non-null assertion
- change check configuration, check commands, or dependency versions
When only a forbidden change turns the check green, stop with `Verdict: Cannot approve` and name the check.
Stop after 3 heal attempts on one check.
```

Exception: when the agreed task changes the tested behavior, a test or snapshot update is part of the change, not a heal. The Defects lane reviews it against `REVIEW_CONTEXT`.

### 3. Update every gate that reads step 7

These lines still require every check to be green. Change each one to "every introduced failure is healed":

- Line 26: the step 7 `Done:` line.
- Line 28: "If a defined check stays red after heal". Change it to "If an introduced failure stays red after heal".
- Line 50: "Start this section only when every defined check is green."
- Line 128: "Heal red defined checks in this session."
- Line 132: section 4 step 3, "Launch fresh review lanes when every defined check is green."

Add the failure classification to the shared brief check evidence: introduced failures healed, pre-existing failures recorded. A lane then does not report a pre-existing failure as a missing check.

### 4. Show each heal

List each heal in the Summary with the check, the file, and the reason. The lanes already see heals in the refreshed diff. The user also needs to see them without a diff review.

### 5. Do not block on a missing external skill

Replace section 2 step 2:

```md
2. If that read fails, run Complexity without it. Record `thermo-nuclear skill missing` in Caveats.
```

Change Lane 3, line 108, to match:

```md
When the brief gives a thermo-nuclear path, read that skill. Keep its items only when they concern extra structure. Read `~/.dud/rules/PRINCIPLES.md` only for Subtract Before You Add, Laziness Protocol, and Deep modules.
```

Change line 81 to: "Put the resolved thermo-nuclear path only in the Complexity brief, when the read succeeded."

Alternative: copy the skill into this kit under `skills/`, so install never depends on the upstream repo.

## Files to change

- `skills/nuke/SKILL.md`: lines 24-28 (step 7, heal limits, stop rule), line 50, shared brief, line 60-61 (section 2 steps 1-2), line 81, line 108, Summary format, lines 128 and 132.
