# Approval deadlock

## Problem

Four rules can combine into a loop that does not end:

1. `AGENTS.md`: every change needs `Verdict: Approved` from a new Oracle before handoff or commit. The task is incomplete while an approval is missing.
2. `subagents/ORACLE.md` Evidence 8: Oracle does not approve when a required check is "not run".
3. `AGENTS.md` and the global rules: do not run a build unless the user asks.
4. Nothing defines "required check".

Results:

- When a build is the only check in a repository and Oracle treats it as required, Oracle returns `Cannot approve`. The agent may not run the build, so it asks Oracle again. The loop does not end. The rules do not force Oracle to treat the build as required, but nothing stops it.
- When a repository has no checks, Oracle can treat every change as missing evidence.
- A typo fix needs the same gate as an auth change. In a TypeScript project that uses `better-result`, each Oracle call reads about 770 lines of rules.

## Fix

Make four changes. Each change closes one part of the loop.

### 1. Define a required check in one place

Add to `AGENTS.md`, under Code:

```md
- A required check is a test, lint, format-check, or type-check command that the project defines and that runs without a build or a dev server. A build is a required check only when the user asks for it. When the project defines no checks, write `Checks: none defined`. That is complete evidence.
```

Use the same words in `ORACLE.md` and in `nuke` step 7, or point both files at the `AGENTS.md` rule.

### 2. Let Oracle accept a check that policy skips

Replace `ORACLE.md` Evidence 6 and 8:

```md
6. Record each required check as passed, failed, not run, unconfirmed, or not applicable. Use not applicable only when the project does not define the check, or when caller policy forbids it without a user request, such as `build: not applicable, user policy`. Reading a test does not establish that it passed.
8. Do not approve when a required check is failed, not run, or unconfirmed, or when other material evidence is missing. A not applicable check does not block approval.
```

The two allowed reasons stop "not applicable" from becoming a way to skip checks.

### 3. Scale the gate to the risk

Replace the Oracle bullet in `AGENTS.md`:

```md
- Oracle: Use when reasoning decides the outcome. Plans, behavior analysis, debugging, refactors, alternatives, disputed findings, and review of your own work. Before handoff or commit, apply the gate of the highest tier that the change touches:
  - High risk (auth, permissions, data writes, migrations, money, public API contracts, concurrency, and agent prompts or rules that change permissions, spending, or approval policy): run `/nuke`.
  - Behavior (code, config, and agent prompts or rules that change what an agent does): one new Oracle approval review. Only an explicit `Verdict: Approved` for the current target passes.
  - Text only (docs, comments, UI copy, and prompt edits that change wording but not behavior): no gate. Reread the diff.
```

The highest tier wins. A doc change and an auth change in one commit get `/nuke`. In this kit, most prompt edits change agent behavior, so they get one Oracle review.

Change the Hard rule to match:

```md
The task is incomplete while a subagent result or the approval that the gate requires is missing.
```

### 4. Add an exit

Add to `AGENTS.md`, under Agents:

```md
5. When Oracle returns `Cannot approve` and the only gaps are checks that you may not run, stop. Report `Unapproved: <gap>` and ask the user to run the check or waive it. Do not call Oracle again while the target and the evidence are unchanged. When the user supplies the check result or a waiver, get a new approval with that evidence.
6. Stop after 3 Oracle approval rounds in one task, including rounds on revised targets. Report the open findings.
```

## Result

```diagram
change ──▶ highest tier it touches?
             ├─ high risk ──▶ /nuke
             ├─ behavior ───▶ Oracle ──┬─ Approved ──────────▶ handoff
             │                         ├─ Changes requested ─▶ fix, Oracle again (max 3 per task)
             │                         └─ Cannot approve
             │                              └─ gap is policy-blocked ─▶ Unapproved: <gap>, ask user
             └─ text only ──▶ reread diff ──▶ handoff
```

## Files to change

- `AGENTS.md`: required-check rule, gate tiers, Hard rule, exit rule.
- `subagents/ORACLE.md`: Evidence 6 and 8.
- `skills/nuke/SKILL.md`: step 7 and the shared brief line "Missing required checks or incomplete scope coverage prevents approval". Use the required-check definition.
