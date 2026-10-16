# Coding Agents

Agent kit for dud.cl. It holds the main agent rules, specialist prompts, and code rules.

Follow [INSTALL.md](./INSTALL.md) to install the kit.

## Contents

- [AGENTS.md](./AGENTS.md): main agent rules
- [ASSISTANT.md](./ASSISTANT.md): custom instructions for concise general chat in ChatGPT, Claude, and Gemini
- [mcp.json](./mcp.json): MCP servers
- [rules/](./rules/): design principles, TypeScript and Go standards, and `better-result` patterns. Install writes them to `~/.dud/rules/`.
- Skills: moved to [enemigos/skills](https://github.com/enemigos/skills). Install with `npx skills add enemigos/skills -g -y`.
- [subagents/](./subagents/): Oracle and Librarian system prompts
