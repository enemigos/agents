---
name: update-verify
description: "Create or update MAP.md, the project feature map that verify reads, by reading source and driving each stale feature in a browser. Edits only MAP.md."
disable-model-invocation: true
---

Keep `MAP.md` at the repository root true to the running app. The `verify` skill reads this file. Write only `MAP.md` and files in `.verify/`. Never edit product code.

A product gap is app behavior that differs from the intended behavior. Report it. Never write it into `MAP.md` as expected behavior.

## MAP.md format

```md
# MAP

## Setup
- Start: <command>, ready at <URL>
- Login: <role, and the seed script or env var that holds its credentials>
- Data: <what the app touches>. Local only.

## Features

### <Feature name>
Entry: <one way a user reaches it>. Repeat this line for each way.
Proof: <visible result that shows it works>
Gotchas: <traps that waste or invalidate a run>
Verified: <short commit hash>
```

Rules:

- Use one `###` section for each feature. Keep the field order. Omit `Gotchas` when it is empty. Omit `Verified` until the section is stamped.
- A feature is anything with a visible outcome. Do not map internals.
- Write only what a user sees and touches: labels, routes, keys, roles. Do not write commands (except in Setup), selectors, or source paths.
- Never write a secret.
- List every `Entry`. Proof through one entry does not cover the others.
- Write a `Proof` that an observer can see. For saved data, include a second view, such as the value after a reload.
- Only this skill writes `Verified`.

## 1. Choose the scope

1. Read `MAP.md`.
2. Put in the scope each feature or surface that the request names. This includes each `Unmapped` or `Map drift` line from a `verify` run. `all` means every feature in the map.
3. When the request names nothing and does not say `all`, add the stale features. A feature is stale when it has no `Verified` line. It is also stale when its commit does not exist (`git cat-file -e <hash>^{commit}`). It is also stale when the changes since that commit can affect it.
4. Get the changed paths with `git diff --name-only <hash>` and `git status --porcelain`. Ignore `MAP.md` and `.verify/`. A change to shared code, config, dependencies, or seed data can affect every feature. When unsure, mark the feature stale.
5. When the request names nothing, says `all`, or `MAP.md` is absent, find missing features. Scan the changed paths since the oldest valid `Verified` commit for user-facing surfaces that `MAP.md` does not list. Require a source path for each. When no `Verified` commit is valid, scan the whole app. Add each missing feature to the scope.
6. For a new map, keep every named feature. Add up to 5 features reachable from the main navigation or home route. The map stays partial until later runs add the rest.
7. When the scope is empty, stop with `Clean: no stale features`.

## 2. Prepare the session

1. Follow the session setup in the `verify` skill. Use the session name `update-verify`.
2. Take each Setup value the request states. Take the other values from `MAP.md`. When a value is absent from both, derive it from package scripts, the Makefile, the README, env examples, and seed scripts.
3. Never drive production or a shared environment.
4. When the app does not respond at the Setup URL, stop with `Blocked: app not running at <URL>. Start it with <command>, then run again.` Start the app only when the request says so.
5. Make sure the server process uses this repository as its working directory. If it does not, stop with `Blocked: app does not run this checkout`.
6. Make sure the app runs the current code. It must be a dev server with reload, or a build that started after the newest modification time of the changed paths from section 1 (of all tracked files when there is no changed path). If not, stop with `Blocked: app may run old code`.
7. When the app cannot reach the Setup state (login fails, seed data is absent), stop with `Blocked: Setup state: <reason>`.
8. When a required input is missing and the repository cannot supply it, stop with `Blocked: <missing input>`.

## 3. Read the source

Launch one read-only subagent for each in-scope feature, in parallel. Use the `explore` type at `medium` thoroughness. A subagent has no conversation history. Write each brief so a subagent with repository access and nothing else can do it.

Give each brief:

- The repository path and the feature name.
- The current `MAP.md` section, or `none`.
- The task: explain how the feature works from the user side, list every entry point, and flag differences from the section with `file:line`.
- The limits: do not drive the app, do not edit files.
- The return shape: `Summary`, `Entry points`, `Proof`, `Gotchas`, `Drift` (with `file:line`, or `none`), and `Recipe` (user-level steps for the live pass).

The reports are unconfirmed. Merge recipes that share an app state. The live pass decides.

## 4. Drive every feature

The main agent drives. Use `snapshot -i` refs and user-level commands only. Do not use internal setters, test-only endpoints, or direct state edits. The Setup state is: logged in as the `Login` role, at the ready URL, with the seeded `Data`.

1. For each in-scope feature, drive every `Entry` in its section and every entry point a subagent reported. Start each one from the Setup state. Check `Proof`.
2. Record one result for each feature: `holds`, `differs: <observed against written>`, or `unreachable: <prerequisite and route tried>`. A feature `holds` only when every `Entry` and the `Proof` hold.
3. When a concrete prerequisite blocks the route, use `unreachable`. A concrete prerequisite is an account, role, entitlement, or external state.
4. After a failed drive, remove its leftover data, return to the Setup state, and retry the feature once. When the retry holds, record `holds` and report `Flaky: <feature>`. Do not stamp that feature.
5. Save a screenshot of each key state in `.verify/`.
6. At the end, close the session. Clean up disposable data. Do not stop a server you did not start.

## 5. Triage and edit

Evidence of intent is a commit message, an issue, or the user's task that names the new behavior. A diff alone is not evidence. Without evidence of intent, report a product gap and leave the section unchanged.

- `holds`: keep the section.
- `differs` with evidence of intent: when the observed behavior matches the behavior the evidence names, update the section to it and cite the evidence. Otherwise report a product gap.
- `differs` without evidence of intent: report a product gap with the screenshot path. To accept the behavior, the user runs this skill again and states the intent in the request.
- A feature or entry found live and missing from the map: add it. Write `Proof` from the source report, the docs, or the task. When the live result differs from that `Proof`, or shows an error or an empty state, report a product gap. Do not add or stamp the feature.
- A feature or entry absent live: when it is also absent from source and evidence of intent exists, remove it. Otherwise report a product gap.
- `unreachable`: when the source or docs show the prerequisite, add it to `Gotchas`. Otherwise treat the result as `differs`. For a feature that is not in `MAP.md`, do not add it. Report it as `unreachable` and count it as not driven.

Add a `Gotchas` line only for a trap in the harness or the environment (login, seed data, timing, viewport) that a drive or a source citation shows. Report app misbehavior as a product gap.

When `MAP.md` is new, write Setup from what the live pass proved. When the request states a new Setup value and the live pass proves it, update that Setup line.

Write `Verified: <git rev-parse --short=12 HEAD>` on a section only when all of these are true:

1. The live pass drove every `Entry` in it.
2. Every `Entry` held as the section now reads.
3. `git status --porcelain` shows no change outside `MAP.md` and `.verify/`.

Otherwise keep the old line and report `Unstamped: <feature and reason>`. When the reason is uncommitted changes, run again after you commit.

Re-read `MAP.md`. Make sure each section follows the format and no two sections share a name.

## 6. Report

Leave the edits uncommitted. On separate lines, list each edit, product gap, `unreachable` feature, `Flaky` feature, and `Unstamped` line. For a new map, say that it is partial. End with exactly one of these lines:

- `Clean: <features checked>`. Use this when every in-scope feature holds and only `Verified` lines changed.
- `Changed: <sections added, updated, or removed>`. Use this when no product gap is open and every in-scope feature was driven.
- `Blocked: <features or input, and reason>`. Use this when the run cannot start. Also use it when any in-scope feature has an open product gap or was not driven.
