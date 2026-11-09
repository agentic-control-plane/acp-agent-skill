# acp-agent-skill

An [Agent Skill](https://agenticcontrolplane.com/AGENTS.md) for the **Agentic Control Plane (ACP)**. It teaches an agent to: check whether ACP is present, set it up (one-command local install, no-signup sandbox provisioning, or MCP connection), call `acp_check` before risky actions, mint scoped child keys for subagents, and draft policy rules for you to confirm in the console. It works in any harness that supports the SKILL.md format.

## Install

**Claude Code** (personal, all projects):

```bash
git clone https://github.com/agentic-control-plane/acp-agent-skill ~/.claude/skills/agentic-control-plane
```

**Claude Code** (one project): clone into `.claude/skills/agentic-control-plane` inside the project.

**Other harnesses:** point your skill loader at [`SKILL.md`](SKILL.md); the `references/` files are progressive-disclosure detail.

## Contents

- [`SKILL.md`](SKILL.md): the skill itself (presence check, setup by situation, `acp_check` before risky actions, child keys for subagents, policy proposals you confirm)
- [`references/setup.md`](references/setup.md): the four setup paths in detail, credential storage, verification, the nine MCP tools
- [`references/child-keys.md`](references/child-keys.md): `POST /api/v1/keys/child` request/response shape, error codes, delegation rules

## Links

- Site: https://agenticcontrolplane.com · agent-facing surface: [/AGENTS.md](https://agenticcontrolplane.com/AGENTS.md)
- Console: https://cloud.agenticcontrolplane.com · API: https://api.agenticcontrolplane.com
- Open core (MIT): [GatewayStack](https://github.com/agentic-control-plane/GatewayStack)

## License

MIT
