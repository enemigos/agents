# Standards lane gap

## Problem

The `nuke` Standards lane reads the language rules file and only three sections of `rules/PRINCIPLES.md`: Model the Domain, Boundary Discipline, and Modules and abstractions. The Complexity lane reads three other sections.

`PRINCIPLES.md` ends with a `# Coding Standards` block that no lane reads:

- HTTP APIs (error schema, `Retry-After`, path naming, HTTP methods)
- Error Message Design
- Sensitive data, telemetry, and debugging
- Explicitness
- Comments

So `nuke` does not check these standards as conventions. For example, no lane reports a 400 response with no field errors, a vague error message, or a stray comment. The Defects lane can still report a leaked secret as a security hole.

The block also overlaps with `rules/TYPESCRIPT.md` "Write better error messages". The two sections give the same advice in different words. They can drift apart.

## Fix

Use a separate standards file. The `PRINCIPLES.md` file then holds only principles. The Standards lane reads `STANDARDS.md` whole, so a new standard needs no change to the lane brief. Both lanes still select sections of `PRINCIPLES.md`.

1. Move the `# Coding Standards` block from `rules/PRINCIPLES.md` to a new `rules/STANDARDS.md`.
2. Merge `PRINCIPLES.md` "Error Message Design" and `TYPESCRIPT.md` "Write better error messages" into one section in `STANDARDS.md`. Keep the UI-copy steps (Outcome, Cause, Reassurance, Action, Escape) and the internal-error rules. Delete the copy in `TYPESCRIPT.md`.
3. In `skills/nuke/SKILL.md` Lane 2, add `~/.dud/rules/STANDARDS.md` to the files the lane reads for every language.
4. In `subagents/ORACLE.md` Standards, add: "Also read `~/.dud/rules/STANDARDS.md`."
5. In `INSTALL.md` Destinations, add `kit/rules/STANDARDS.md` to `~/.dud/rules/STANDARDS.md`.
6. In `README.md`, add "cross-language standards" to the `rules/` line.

## Proposed Lane 2 text

```md
Denotation: convention. File a finding when the reviewed code violates `~/.dud/rules/STANDARDS.md`, or the rules file for its language: `~/.dud/rules/TYPESCRIPT.md` for TypeScript, `~/.dud/rules/GOLANG.md` for Go.

Read `STANDARDS.md` and the rules file for each language in the reviewed code. When `TYPESCRIPT.md` points at `~/.dud/rules/BETTER-RESULT.md`, read it too. Read `~/.dud/rules/PRINCIPLES.md` only for Model the Domain, Boundary Discipline, and Modules and abstractions excluding Deep modules. Skip checks that tooling already enforces.
```

## Cost

Oracle already reads all of `PRINCIPLES.md` on every call, so the move adds no context to an Oracle call. The Standards lane reads about 60 more lines.
