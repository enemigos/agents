---
name: verify
description: "Verify a reported UI bug or a completed user flow by driving the real app in a browser and keeping proof. Reads MAP.md when the project has one."
disable-model-invocation: true
---

Verify behavior by driving the real app with agent-browser. Use its accessibility snapshots to find controls, its interaction commands to follow the user path, and its screenshots to preserve proof. Tests, code inspection, and screenshots alone do not verify the behavior.

## 1. Choose the claim

Read `MAP.md` at the repository root when it exists. Find each feature the task touches by name, route, or label.

Write one claim that the browser session can prove or disprove.

- **Bug report:** Turn the report, caption, and screenshots into a concrete claim about the observed behavior, location, inputs, and expected behavior. Mark missing details as hypotheses. Do not treat the reporter's diagnosis as fact.
- **Implemented flow:** State the complete user journey and each visible result or side effect that must occur. Example: submit the form, see confetti, then see the map centered on the submitted house.

When `MAP.md` lists the feature, read its `Gotchas` before you drive.

- For an implemented flow, take the user paths from its `Entry` lines and the required result from its `Proof` line. Keep any extra outcome the task requires.
- When the task intentionally changes what `Proof` states, take the required result from the task and report `Map drift`.
- For a bug report, drive the reported path first. Use the `Entry` lines only to vary it.

## 2. Prepare the session

1. Inspect every supplied screenshot. Record visible text, page identity, state, and clues needed to reach the same state.
2. Confirm the target app responds. Take its environment, account, seed data, and starting URL from `MAP.md` Setup. When `MAP.md` is absent or silent, identify them from the repository. Use a local or disposable environment. Never drive production or a shared environment.
3. Load the installed agent-browser skill from `vercel-labs/agent-browser`. If it is missing, install it with `npx skills add vercel-labs/agent-browser`. Check `agent-browser --help` when its command syntax is uncertain.
4. Create one named agent-browser session for the verification. Open the starting URL, set the required viewport, and run `snapshot -i` before acting.
5. Store screenshots in a stable gitignored artifact directory such as `.verify/`.
6. If the app or required data is unavailable, report the exact blocker. Do not replace browser verification with code inspection.

## 3. Drive the claim

Follow the path a user takes. Use the refs from `agent-browser snapshot -i` with `click`, `fill`, `type`, `select`, `check`, and other user-level commands. For targeted keyboard input, use `focus <ref>` and then `press <key>`. After navigation or a DOM change, wait for a visible result and take a new snapshot before the next action. Do not use internal setters, test-only endpoints, or direct state edits.

For a bug report:

1. Recreate the reported preconditions as closely as the evidence permits.
2. Attempt the shortest plausible reproduction.
3. If it does not reproduce, vary only evidence-backed unknowns such as viewport, role, input, or navigation order. Record each attempt. Do not invent an unlimited search.
4. Call the bug reproduced only when the observed behavior matches the report. Otherwise report `Not reproduced`, the paths tried, and the remaining unknowns.

For an implemented flow:

1. Start before the changed behavior and complete the whole journey.
2. Verify each required intermediate state and final result.
3. When the claim includes a durable side effect, verify it through a second user-visible view, such as reloading, reopening the record, or viewing it elsewhere in the app.
4. Exercise the relevant failure or boundary state when the acceptance criteria name one.
5. When `MAP.md` lists several entries for the feature, drive each entry the change can affect. Proof through one entry does not cover the others.

Prefer snapshot refs, ARIA labels, and stable data attributes over coordinates. Use `get url`, `get text`, `is visible`, `console`, or `errors` when needed to explain an observation. Browser state and logs can support the result, but the visible user flow decides the verdict.

## 4. Keep proof

Capture the key action and its result, not only the final screen. Use `agent-browser screenshot <path>` after the page reaches each decisive state. Save enough app identity and context to understand the image. Inspect each screenshot before citing it. Record the URL, session, environment, viewport when relevant, data used, and exact commands or steps.

If the flow mutates data, use disposable data and clean it up when safe. Close the named agent-browser session when verification ends. Do not stop a server you did not start.

## 5. Report

Report the claim, environment, exact browser path, observed result, and proof artifacts. Separate facts from hypotheses. Report skipped steps as skipped. Use `Verified` only when browser evidence proves every required outcome. If a required outcome fails, is blocked, is skipped, or remains unobserved, use `Not verified`.

Do not edit `MAP.md` in this skill. Before the final line, add each of these lines that applies:

- `Unmapped: <surface the work touches that MAP.md does not list>`
- `Map drift: <entry>, <intended by the task or unexpected>, <what the app did instead>`
- `No MAP.md: run update-verify to create it.`

End with exactly one of these lines:

- `Reproduced: <bug and the conditions that trigger it>`
- `Not reproduced: <attempts and remaining unknowns>`
- `Verified: <complete flow that worked>`
- `Not verified: <failed or blocked step and why>`
