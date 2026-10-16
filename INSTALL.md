# Install

You install this kit into a project. The install can also copy the rules to your global agent instructions.

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

## Steps

1. Set kit to the directory that contains this file.
   Done: `kit/subagents/ORACLE.md` exists.
2. Set target to the named path, or to the current working directory.
   Done: target exists and is a directory.
3. Create `~/.dud/rules` when it is absent.
   Done: `~/.dud/rules` is a directory.
4. Copy each source in Destinations to its dest. Use the Rule.
   Done: each dest is a write, a skip, or a Conflicts entry.
5. Go to heading Global rules and apply it.
   Done: the Global rules completion line is met.
6. Go to heading MCP and apply it.
   Done: the MCP completion line is met.
7. Go to heading External skills and apply it.
   Done: the External skills completion line is met.
8. Go to heading OpenSpec and apply it.
   Done: the OpenSpec completion line is met.
9. Print Written, Skipped, and Conflicts.
   Done: every dest from steps 4 to 8 is in exactly one list.

A kept `oracle.md` that differs from the kit does not load these Oracle rules. Report that path in Conflicts.

## Global rules

Each agent reads its own global instructions file. The install copies the kit rules to the global file of each agent the user uses. When the user picks no agent, skip this section.

1. Ask the user which agents in the table they use.
2. Use the Rule to copy `kit/AGENTS.md` to the dest of each picked agent.

| Agent | Dest |
| --- | --- |
| Factory Droid | `~/.factory/AGENTS.md` |
| pi | `~/.pi/agent/AGENTS.md` |
| opencode | `~/.config/opencode/AGENTS.md` |
| Codex | `~/.codex/AGENTS.md` |
| Claude Code | `~/.claude/CLAUDE.md` |
| Gemini CLI | `~/.gemini/GEMINI.md` |
| Amp | `~/.config/amp/AGENTS.md` |
| Crush | `~/.config/crush/CRUSH.md` |
| Qwen Code | `~/.qwen/QWEN.md` |
| Cursor | None |

Do not create `~/.codex/AGENTS.override.md`. That file shadows the copied file. When that file exists, add its path to Conflicts.

Cursor has no global instructions file. When the user picks Cursor, tell them to paste the rules into Settings → Rules. Add Cursor to Skipped.

Done: each picked agent is a write, a skip, or a Conflicts entry. Each agent the user did not pick stays untouched.

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

## OpenSpec

OpenSpec adds the `opsx` commands to your agent. The flow uses `propose`, `apply`, and `archive`.

| Agent | Tool ID | Skills dir | Propose command |
| --- | --- | --- | --- |
| Claude Code | `claude` | `.claude/skills` | `/opsx:propose` |
| Droid | `factory` | `.factory/skills` | `/opsx-propose` |
| Cursor | `cursor` | `.cursor/skills` | `/opsx-propose` |
| Codex | `codex` | `.agents/skills` | `$openspec-propose` |

1. Check that Node.js 20.19.0 or later is on the PATH.
2. If it is not, add this heading to Conflicts.
3. If `openspec --version` fails, run `npm install -g @fission-ai/openspec@latest`.
4. If the install command fails, add the command and its exit status to Conflicts.
5. If the scope is personal, stop here.
6. If your agent is not in the table, add this heading to Conflicts.
7. Set Present to the paths that match `target/openspec` and `target/<skills dir>/openspec-*`.
8. If Present is not empty, add each path to Conflicts.
9. Run `openspec init --tools <tool id>` from target.
10. If `target/openspec/config.yaml` is absent, add its path to Conflicts.
11. If no file matches `target/<skills dir>/openspec-*/SKILL.md`, add the pattern to Conflicts.

After step 2, 4, 6, or 8 adds to Conflicts, skip all later steps. For personal scope, run steps 6 to 11 later in each project. Never run `openspec update`. It can overwrite generated files.

The default profile includes `propose`, `explore`, `apply`, `update`, `sync`, and `archive`.

Done: step 2, 4, 6, or 8 added to Conflicts. Or `openspec --version` prints a version and, for project scope, each file from steps 10 and 11 exists or Conflicts lists it.

## External skills

Run these commands from target. Add `-g` when the scope is personal. Add `-a <agent>` to install for one agent.

```sh
npx skills add enemigos/skills --skill cold-email naval nuke research so to-pr to-trello vercel -y
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
