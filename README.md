# acp-agent-skill

An [Agent Skill](https://agenticcontrolplane.com/AGENTS.md) for the **Agentic Control Plane (ACP)** — see, price, and control every tool call your AI agents make.

Install this skill into any harness that supports the SKILL.md format and your agent knows how to: check whether ACP is present, set it up (one-command local install, no-signup sandbox provisioning, or MCP connection), check sensitive actions before running them, mint scoped child keys for subagents, and propose policy rules that a human confirms in the console.

## Install

**Claude Code** (personal, all projects):

```bash
git clone https://github.com/agentic-control-plane/acp-agent-skill ~/.claude/skills/agentic-control-plane
```

**Claude Code** (one project): clone into `.claude/skills/agentic-control-plane` inside the project.

**Other harnesses:** point your skill loader at [`SKILL.md`](SKILL.md); the `references/` files are progressive-disclosure detail.

## Contents

- [`SKILL.md`](SKILL.md) — the skill: presence check, setup by situation, `acp_check` before blast-radius actions, child keys for subagents, human-confirmed policy proposals
- [`references/setup.md`](references/setup.md) — the four setup paths in detail, credential storage, verification, the nine MCP tools
- [`references/child-keys.md`](references/child-keys.md) — `POST /api/v1/keys/child` request/response shape, error codes, delegation rules

## Links

- Site: https://agenticcontrolplane.com · agent-facing surface: [/AGENTS.md](https://agenticcontrolplane.com/AGENTS.md)
- Console: https://cloud.agenticcontrolplane.com · API: https://api.agenticcontrolplane.com
- Open core (MIT): [GatewayStack](https://github.com/agentic-control-plane/GatewayStack)

## License

MIT
