# Install

You install this kit into a project.

Kit is the directory that contains this file. Target is the named project path, or the current working directory. Agent dir is the config directory of the agent that runs this guide. Use the agent dir name that your agent reads.

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
| `kit/subagents/ORACLE.md` | `target/<agent dir>/agents/oracle.md` | `~/<agent dir>/agents/oracle.md` |
| `kit/subagents/LIBRARIAN.md` | `target/<agent dir>/agents/librarian.md` | `~/<agent dir>/agents/librarian.md` |
| `kit/skills/cold-email/` | `target/<agent dir>/skills/cold-email/` | `~/<agent dir>/skills/cold-email/` |
| `kit/skills/naval/` | `target/<agent dir>/skills/naval/` | `~/<agent dir>/skills/naval/` |
| `kit/skills/nuke/` | `target/<agent dir>/skills/nuke/` | `~/<agent dir>/skills/nuke/` |
| `kit/skills/research/` | `target/<agent dir>/skills/research/` | `~/<agent dir>/skills/research/` |
| `kit/skills/so/` | `target/<agent dir>/skills/so/` | `~/<agent dir>/skills/so/` |
| `kit/skills/to-pr/` | `target/<agent dir>/skills/to-pr/` | `~/<agent dir>/skills/to-pr/` |
| `kit/skills/to-trello/` | `target/<agent dir>/skills/to-trello/` | `~/<agent dir>/skills/to-trello/` |
| `kit/skills/update-verify/` | `target/<agent dir>/skills/update-verify/` | `~/<agent dir>/skills/update-verify/` |
| `kit/skills/vercel/` | `target/<agent dir>/skills/vercel/` | `~/<agent dir>/skills/vercel/` |
| `kit/skills/verify/` | `target/<agent dir>/skills/verify/` | `~/<agent dir>/skills/verify/` |

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

MCP dest is the MCP config file that your agent reads. Use `target/<agent dir>/mcp.json` for project scope and `~/<agent dir>/mcp.json` for personal scope. Use another path when your agent reads MCP config from there.

Read `kit/mcp.json`. Root key is the key that the dest format uses for the server map. Most agents use `mcpServers`.

1. Create the dest file with `{ "<root key>": {} }` when it is absent.
2. Take the server map from `kit/mcp.json`. If that file has key `mcpServers`, use that object. If it does not, use the root object.
3. Add a kit server key when that key is absent from dest. When you add `exa`, set header `x-api-key` to a reference to env var `EXA_API_KEY`. Use the env var syntax that the dest format uses.
4. Keep a key when it already exists. Add dest to Conflicts when a kit key exists and differs.
5. When you compare `exa`, treat any reference to env var `EXA_API_KEY` as a match, such as `${EXA_API_KEY}` or `${env:EXA_API_KEY}`.

Report when `EXA_API_KEY` is unset.

Done: dest contains every key from the kit server map, or Conflicts lists dest.

## External skills

Run these commands from target. Add `-g` when the scope is personal. Add `-a <agent>` to install for one agent.

```sh
npx skills add mattpocock/skills --skill codebase-design domain-modeling grill-me grill-with-docs grilling handoff improve-codebase-architecture wizard writing-for-agents -y
npx skills add vercel/ai@ai-sdk -y
npx skills add vercel-labs/agent-skills --skill writing-guidelines vercel-react-best-practices -y
npx skills add cloudflare/skills --skill cloudflare workers-best-practices -y
npx skills add raphaelsalaja/userinterface-wiki@userinterface-wiki -y
npx skills add superdesigndev/treg --skill treg -y
npx skills add Jakubantalik/transitions.dev --skill transitions-dev -y
npx skills add cursor/plugins --skill thermo-nuclear-code-quality-review deslop -y
npx skills add motiondivision/ai-kit --skill motion -y
npx skills add emilkowalski/skills --skill find-animation-opportunities mobile-native break-ui -y
npx skills add haowjy/creative-writing-skills --skill creative-writing-craft -y
npx skills add dmmulroy/skills --skill bro -y
npx skills add dmmulroy/better-result --skill adopt-better-result -y
npx skills add supabase/agent-skills --skill supabase supabase-postgres-best-practices -y
npx skills add jakubkrehel/skills --skill better-writing better-interface -y
npx skills add vercel-labs/agent-browser --skill agent-browser -y
npx skills add crafter-station/skills --skill cli-build cli-audit surface-recon -y
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
