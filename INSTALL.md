# Install

You install this kit into a Cursor project.

Kit is the directory that contains this file. Target is the named project path, or the current working directory.

## Rule

Detect each destination before you write.

1. Write the destination when it is absent.
2. Skip the destination when it matches the source.
3. Keep the destination when it differs. Add the path to Conflicts.

Create a parent directory when it is absent.

Default scope is project. If the user asks for a personal install, use the personal dest.

If target is kit, skip a copy whose dest is the source file. Continue with the other dests.

## Destinations

| Source | Project dest | Personal dest |
| --- | --- | --- |
| `kit/AGENTS.md` | `target/AGENTS.md` | `target/AGENTS.md` |
| `kit/rules/PRINCIPLES.md` | `~/.dud/rules/PRINCIPLES.md` | `~/.dud/rules/PRINCIPLES.md` |
| `kit/rules/TYPESCRIPT.md` | `~/.dud/rules/TYPESCRIPT.md` | `~/.dud/rules/TYPESCRIPT.md` |
| `kit/rules/GOLANG.md` | `~/.dud/rules/GOLANG.md` | `~/.dud/rules/GOLANG.md` |
| `kit/rules/BETTER-RESULT.md` | `~/.dud/rules/BETTER-RESULT.md` | `~/.dud/rules/BETTER-RESULT.md` |
| `kit/subagents/ORACLE.md` | `target/.cursor/agents/oracle.md` | `~/.cursor/agents/oracle.md` |
| `kit/subagents/LIBRARIAN.md` | `target/.cursor/agents/librarian.md` | `~/.cursor/agents/librarian.md` |
| `kit/skills/cold-email/` | `target/.cursor/skills/cold-email/` | `~/.cursor/skills/cold-email/` |
| `kit/skills/naval/` | `target/.cursor/skills/naval/` | `~/.cursor/skills/naval/` |
| `kit/skills/nuke/` | `target/.cursor/skills/nuke/` | `~/.cursor/skills/nuke/` |
| `kit/skills/research/` | `target/.cursor/skills/research/` | `~/.cursor/skills/research/` |
| `kit/skills/to-pr/` | `target/.cursor/skills/to-pr/` | `~/.cursor/skills/to-pr/` |
| `kit/skills/to-trello/` | `target/.cursor/skills/to-trello/` | `~/.cursor/skills/to-trello/` |
| `kit/skills/vercel/` | `target/.cursor/skills/vercel/` | `~/.cursor/skills/vercel/` |
| `kit/skills/verify/` | `target/.cursor/skills/verify/` | `~/.cursor/skills/verify/` |

Copy each skill directory as a whole.

## Steps

1. Set kit to the directory that contains this file.
   Done: `kit/subagents/ORACLE.md` exists.
2. Set target to the named path, or to the current working directory.
   Done: target exists and is a directory.
3. Create `~/.dud/rules` when it is absent.
   Done: `~/.dud/rules` is a directory.
4. Copy each source in Destinations to its dest. Use the Rule.
   Done: each dest is a write, a skip, or a Conflicts entry.
5. Go to heading MCP and apply it.
   Done: the MCP completion line is met.
6. Go to heading External skills and apply it.
   Done: the External skills completion line is met.
7. Print Written, Skipped, and Conflicts.
   Done: every dest from steps 4 to 6 is in exactly one list.

A kept `oracle.md` that differs from the kit does not load these Oracle rules. Report that path in Conflicts.

## MCP

MCP dest is `target/.cursor/mcp.json` for project scope. MCP dest is `~/.cursor/mcp.json` for personal scope.

Read `kit/mcp.json`. Cursor dest uses root key `mcpServers`.

1. Create the dest file with `{ "mcpServers": {} }` when it is absent.
2. Take the server map from `kit/mcp.json`. If that file has key `mcpServers`, use that object. If it does not, use the root object.
3. Add a kit server key when that key is absent from dest. When you add `exa`, set header `x-api-key` to `${env:EXA_API_KEY}`.
4. Keep a key when it already exists. Add dest to Conflicts when a kit key exists and differs.
5. When you compare `exa`, treat kit value `${EXA_API_KEY}` as a match for dest `${env:EXA_API_KEY}`.

Report when `EXA_API_KEY` is unset.

Done: dest contains every key from the kit server map, or Conflicts lists dest.

## External skills

Run these commands from target. Add `-g` when the scope is personal.

```sh
npx skills add mattpocock/skills --skill domain-modeling grill-me grill-with-docs grilling wizard writing-for-agents -a cursor -y
npx skills add vercel/ai@ai-sdk -a cursor -y
npx skills add vercel-labs/agent-skills --skill writing-guidelines vercel-react-best-practices -a cursor -y
npx skills add cloudflare/skills --skill cloudflare workers-best-practices -a cursor -y
npx skills add raphaelsalaja/userinterface-wiki@userinterface-wiki -a cursor -y
npx skills add superdesigndev/treg --skill treg -a cursor -y
npx skills add Jakubantalik/transitions.dev --skill transitions-dev -a cursor -y
npx skills add cursor/plugins --skill thermo-nuclear-code-quality-review deslop -a cursor -y
npx skills add motiondivision/ai-kit --skill motion -a cursor -y
npx skills add emilkowalski/skills --skill find-animation-opportunities -a cursor -y
npx skills add haowjy/creative-writing-skills --skill creative-writing-craft -a cursor -y
npx skills add dmmulroy/skills --skill bro -a cursor -y
npx skills add jakubkrehel/skills --skill better-writing better-interface -a cursor -y
npx skills add vercel-labs/agent-browser --skill agent-browser -a cursor -y
npx skills add crafter-station/skills --skill cli-build cli-audit surface-recon -a cursor -y
```

Done: each command exits 0, or Conflicts records the command and the exit status.

If the user names a subset, run only the matching commands. Keep the same flags.

Go to heading Grill when the user asks which grill skill to use.

## Grill

| Situation | Skill |
| --- | --- |
| No repo, or the idea is not about this codebase | `grill-me` |
| A change in a repo, or a repo with no domain docs | `grill-with-docs` |

`grill-me` writes no files. `grill-with-docs` writes terms to `CONTEXT.md` and hard decisions to `docs/adr/`.
