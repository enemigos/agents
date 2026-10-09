---
name: to-pr
description: "Create a draft PR that targets `main`."
disable-model-invocation: true
---

Create a draft PR that targets `main`.

Use Conventional Commits for each commit message.

Name the branch `<type>/<short-kebab-description>`, such as `fix/map-address-unavailable`. Use a Conventional Commits type such as `feat`, `fix`, `docs`, `refactor`, `test`, `build`, `ci`, `chore`, `perf`, `style`, or `revert`. Do not add a coding-agent or harness prefix such as `agent/`, `codex/`, or `claude/`.

The project uses squash merge. Use a descriptive branch name.

Before you create the PR:

1. Update the branch with the latest changes from `main`.
2. Resolve all merge conflicts.
3. Make sure the branch is ready to merge.

Create the draft PR on GitHub with the body as its description.

1. Pass the body directly in the GitHub PR creation request.
2. Set the base branch to `main` and set the PR to draft.
3. If you use `gh`, pass the body with `--body`. Do not use `--body-file`.
4. Do not write or return the body as a file. Do not create a local or temporary PR-description artifact.
5. If GitHub access fails, report the error. Do not save the body locally as a workaround.

The PR description does not need Simplified Technical English (STE). Keep it minimal, casual, and free of jargon.

In the PR description:

1. Write exactly two sentences in two short paragraphs.
2. Sentence 1 says what was wrong or missing.
3. Sentence 2 says what changed.
4. Keep the full body at 40 words or fewer.
5. Use casual, spoken English. Sound like a teammate, not release notes.
6. Do not list test or verification results.
7. Use no headings, bullets, Markdown, setup text, or closing text.
8. Do not say `This PR` or `This change`.
9. Keep only the reason and user effect. Skip details shown by the diff.

## Writing style

1. Do not use openers, closing text, or filler.
2. Do not use smart quotes, em dashes, or ellipsis characters.
3. Do not use decorative Unicode. Use natural-language characters when the content requires them.
4. Do not add a preamble, summary, or closing remark.
5. Make sure all strings are safe for JSON serialization.

## Body

Before you submit the PR description, rewrite it if it has more than 40 words, more than two sentences, formal release-note language, filler, or unnecessary implementation detail.

Examples:

```
Discounted jobs showed the full price on the tip screen.

Pass the discount through checkout and reloads.
```

```
Big requests held every result in memory.

Write results in chunks, then drop each chunk.
```
