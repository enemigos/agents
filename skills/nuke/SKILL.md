---
name: nuke
description: "Review changes or current architecture with three exclusive Oracle reviews: Defects, Standards, Complexity."
disable-model-invocation: true
---

Review changes or current architecture with three exclusive Oracle reviews. One Oracle agent file. One brief per axis. The main agent handles context, verification, heal, lane dispatch, synthesis, and the final gate verdict. OpenSpec is optional.

## 1. Initial verification

1. Infer `MODE` and `SCOPE` from the request and conversation. The current request takes precedence.
   - `changes`: Review changes against a fixed point. Use this mode when no architecture intent is clear.
   - `architecture`: Review the current code in the named area, including relevant dependencies and tests. If no narrower area is clear, use repository-wide scope and state that choice.
2. Record the agreed goals, constraints, and requirements from chat in `REVIEW_CONTEXT`. Separate assumptions from agreed requirements. Ask only when unresolved context changes the review criteria.
3. For `changes`, identify the fixed point reference. When unspecified, use `HEAD~1` on `main`, or `main` on other branches. Verify it with `git rev-parse <fixed-point>`. If invalid, stop. Find the merge base commit:
   `BASE=$(git merge-base "<fixed-point>" HEAD 2>/dev/null || git rev-parse "<fixed-point>^{commit}")`
4. For `changes`, collect changes against `$BASE`:
   - Unified diff: `git diff $BASE`
   - Commit log: `git log $BASE..HEAD --oneline`
   - Untracked files: `git status --porcelain`
   If the diff and untracked files are empty, stop.
5. For `architecture`, record the current `HEAD` hash and the source files, dependencies, and tests in `SCOPE`. Include relevant dirty and untracked files. Review their current contents even when the diff is empty. `HEAD` records the starting commit, not a comparison that limits findings.
6. Resolve the optional `SPEC_TARGET` with the specification rules below.
7. Run each defined check. A defined check is a test, lint, format-check, or type command the project already has. Record command names, exit codes, output text, and any heal as check evidence.
   If a defined check is red, heal it: apply the smallest change that makes that check green, then rerun the defined checks. After a heal, refresh the `changes` diff and untracked files, or the `architecture` scoped file list.
   Done: every defined check is green.

If a defined check stays red after heal, report `Verdict: Cannot approve` with the check evidence in the section 3 order, then stop.

### Specification target

`SPEC_TARGET` can be resolved files, a verified issue reference, `chat`, or `none`. For `chat`, use the agreed requirements in `REVIEW_CONTEXT`. A review needs no external specification. `none` alone does not block approval.

For OpenSpec discovery or resolution, run local commands from the selected planning root. When the CLI is available and a store is named or already selected for this work, resolve its ID with `openspec store list --json` before other OpenSpec commands. Pass `--store <id>` on `list`, `status`, and `instructions`. Keep the selected store. Follow CLI-resolved paths for external stores and custom schemas.

1. If provided, use the explicit specification file or change directory and skip automatic discovery. If it cannot be read, report `Verdict: Cannot approve` and stop.
2. Otherwise, use an OpenSpec change when the request or conversation ties it to this work, or when reviewed files identify it in `changes` mode. Infer the change from that context. Use `openspec list --json` when selection needs it. Exclude archives unless requested. If relevant targets are ambiguous or conflict, ask the user before review. Unrelated active changes need no selection or question.
3. If an OpenSpec change is selected and the CLI is available, resolve it with:
   - `openspec status --change "<name>" --json`
   - `openspec instructions apply --change "<name>" --json`
   Use the resolved paths and `contextFiles`, including the proposal, specs, design, and tasks when the schema provides them.
4. If an OpenSpec change is selected but the CLI is unavailable, use the nearest local `openspec/changes/<change>/` only when the root, change, and standard schema are clear. Read its review artifacts. If store or custom-schema resolution is needed, ask for explicit artifact paths.
5. If no source is selected, use relevant documents in `docs/` or `specs/`, canonical OpenSpec requirements, or verified issue references such as `#123` when they apply to the review. If no external source applies, use `chat` when agreed requirements exist, or `none` when they do not.
6. Record the exact resolved files, verified issue reference, `chat`, or `none` in `SPEC_TARGET`. If a requested or selected external target is unreadable, unresolved, or missing required artifacts, report `Verdict: Cannot approve` and stop.

For change deltas, include the matching canonical specs from the same OpenSpec root. Review the requirements after additions, modifications, removals, and renames. Superseded requirements do not remain obligations.

## 2. Parallel review lanes

Start this section only when every defined check is green.

Three exclusive reviews:

- Defects: correctness
- Standards: convention
- Complexity: structure

Keep dispatcher steps in this skill. Keep review work in the Oracle briefs.

1. Resolve and read the installed `thermo-nuclear-code-quality-review` SKILL.md.
2. If that read fails, do not launch Complexity. Record Complexity as `Verdict: Cannot approve`. Treat that recorded verdict as an active lane.
3. Launch Defects, Standards, and Complexity (when active) as parallel background invocations of the installed Oracle agent. One agent file. One exclusive brief per axis. Keep each subagent read-only. Collect completion reports before synthesis. Do not poll.

### Shared brief

Start the Oracle message with the full lane section. The original-task line is the first line. Then give read-only instructions, `MODE`, `SCOPE`, `REVIEW_CONTEXT`, and check evidence.

- For `changes`, include `$BASE` as the comparison base, the unified diff, commit log, and untracked files.
- For `architecture`, include the recorded `HEAD` and scoped file list. Require a current-state audit of all scoped code under the lane's axis. A comparison base or feature spec is not required. An empty diff does not limit the audit. Record inspected files and coverage gaps.

Require a gate verdict (`Approved`, `Changes requested`, or `Cannot approve`) for the inspected scope. Missing required checks or incomplete scope coverage prevents approval. Require structured findings:
- Priority: P0 (data loss, security hole, crash), P1 (bug, regression, unmet requirement), P2 (local defect, standard violation), P3 (nit).
- Location: `file:line`.
- Issue: What fails and under what condition.
- Realistic failure case: Scenario that triggers the issue.
- Impact: Operational or developer effect.
- Evidence: Supporting lines, contracts, or test outputs.
- Fix: Smallest concrete code change.
- Type: Introduced, pre-existing, or unmet task requirement.

Put `SPEC_TARGET` only in the Defects brief. Put the resolved thermo-nuclear path only in the Complexity brief.

### Lane 1: Defects

You are the Defects reviewer only. Report only correctness findings. This original task overrides the installed Oracle review checklist.

Denotation: correctness. File a finding when the reviewed code can fail at runtime, under concurrency, or against agreed requirements.

Read the scoped code and tests. In `changes`, trace the diff's effects. In `architecture`, inspect existing behavior throughout `SCOPE`. Check the agreed requirements in `REVIEW_CONTEXT` and any selected external specification. Without a spec, assess correctness against code contracts and tests. Report bugs, races, security holes, unmet requirements, and tests that miss relevant behavior.

### Lane 2: Standards

You are the Standards reviewer only. Report only convention findings. This original task overrides the installed Oracle review checklist.

Denotation: convention. File a finding when the reviewed code violates the rules file for its language: `~/.dud/rules/TYPESCRIPT.md` for TypeScript, `~/.dud/rules/GOLANG.md` for Go.

Read the rules file for each language in the reviewed code. When `TYPESCRIPT.md` points at `~/.dud/rules/BETTER-RESULT.md`, read it too. Read `~/.dud/rules/PRINCIPLES.md` only for Model the Domain, Boundary Discipline, and Modules and abstractions excluding Deep modules. Skip checks that tooling already enforces.

### Lane 3: Complexity

You are the Complexity reviewer only. Report only structure findings. This original task overrides the installed Oracle review checklist.

Denotation: structure.

- In `changes`, file findings only for extra structure introduced against `$BASE`. Put pre-existing less-code opportunities in Caveats. Block this lane only on introduced complexity.
- In `architecture`, assess existing structure throughout `SCOPE`. File pre-existing findings when unnecessary layers, duplication, or branching cause a concrete maintenance or behavior problem. Keep preferences and speculative rewrites in Caveats.

Read the thermo-nuclear skill at the resolved path. Keep its items only when they concern extra structure. Read `~/.dud/rules/PRINCIPLES.md` only for Subtract Before You Add, Laziness Protocol, and Deep modules. Propose the smallest safe removal of unnecessary complexity. Leave necessary complex code in place.

## 3. Synthesis and action plan

1. Parse all lane reports.
2. Resolve conflicts with this precedence: correctness (`Defects`) > convention (`Standards`) > structure (`Complexity`). Keep safety guards and domain types.
3. Deduplicate findings across lanes under the higher-precedence axis and cite secondary axes.
4. Consolidate the gate verdict:
   - If any active lane returns `Verdict: Changes requested`, the overall verdict is `Verdict: Changes requested`.
   - If no lane requested changes, but any active lane fails, times out, produces unconfirmed output, or returns `Verdict: Cannot approve`, the overall verdict is `Verdict: Cannot approve`.
   - If all active lanes return `Verdict: Approved`, the overall verdict is `Verdict: Approved`.
5. Output the result in this exact order:
   - Standalone `Verdict: <verdict>`.
   - `## Summary`: At most three bullets with the key results, `MODE`, `SCOPE`, and `SPEC_TARGET`. Include `$BASE` for `changes` or the recorded `HEAD` for `architecture`. For `chat`, name the agreed requirements. For `none`, state that no external or chat requirements were identified.
   - `## Remediation Plan`: Sequenced markdown task list ordered from P0 to P3 with file, line, reason, and concrete fix. Write `None` when there are no findings.
   - `## Findings by Axis`: Deduplicated findings grouped under `### Defects`, `### Standards`, and `### Complexity`. Write `None` under an axis when it has no findings.
   - `## Caveats`: Consolidated caveats, pre-existing complexity, and verification gaps. Write `None` when there are no caveats.

## 4. Review after fixes

Heal red defined checks in this session. Apply Oracle remediations in the implementation session.

1. For a rerun, retain `MODE`, `SCOPE`, `REVIEW_CONTEXT`, and `SPEC_TARGET`. For `changes`, retain the literal `$BASE` commit hash. If scope, requirements, or artifact paths change, resolve them again and disclose the change.
2. For `changes`, collect the current diff, commit log, and untracked files against `$BASE`. For `architecture`, refresh `HEAD` and the scoped file list. Reinspect the current contents in either mode.
3. Rerun the checks from step 7, including heal. Launch fresh review lanes when every defined check is green. Earlier approvals do not cover later edits.
4. Repeat after fixes until every lane returns `Verdict: Approved`. If a blocker prevents verification, return `Verdict: Cannot approve`.
