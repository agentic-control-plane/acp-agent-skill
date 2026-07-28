---
name: agentic-control-plane
description: Set up and use the Agentic Control Plane (ACP) — see, price, and control every tool call an agent makes. Use when installing or configuring guardrails for an agent, controlling which tool calls are allowed, setting up an audit trail, tracking or attributing agent costs and spend, checking whether a sensitive action is permitted before running it, spawning subagents under a scoped budget, or provisioning ACP for a user who has no account yet.
---

# Agentic Control Plane (ACP)

ACP lets you and your human see, price, and control every tool call an agent makes. Each call is checked against policy before it runs (allow / flag / deny / ask), priced — including subscription traffic at API rates — and written to an audit trail with verified identity on every call. If the ACP API is ever unreachable, the check fails open: an ACP outage never blocks the agent.

- API base: `https://api.agenticcontrolplane.com`
- Console (humans): `https://cloud.agenticcontrolplane.com`
- Agent-facing setup surface: https://agenticcontrolplane.com/AGENTS.md
- Any page on agenticcontrolplane.com is also served as raw markdown — append `.md` to the URL (3–5x fewer tokens than the HTML)

## Step 1 — Check whether ACP is already present

Try, in order:

1. **MCP tools.** If tools named `acp_status`, `acp_check`, etc. are in your tool list, ACP is connected. Call `acp_status` to confirm connectivity and budget state.
2. **Local install.** On a machine: `cat ~/.acp/credentials 2>/dev/null` — if the file exists, the local install is configured.

If either succeeds, skip to Step 3.

## Step 2 — Set it up (pick the path that matches your situation)

**You are a coding agent (Claude Code, Cursor, Codex CLI).** One command, no account, nothing leaves the machine:

```bash
curl -sf https://agenticcontrolplane.com/install.sh | bash -s -- --local
```

That gives the machine allow/ask/deny policy, a safety floor, and an on-device audit log. Run it without `--local` to also connect a cloud workspace (free for individuals) with dashboards, cost X-ray, and team policy.

**You are provisioning for a human who has no ACP account yet.** Use the no-signup sandbox flow:

```
POST https://api.agenticcontrolplane.com/v1/sandbox-tenants
```

The response includes `nextSteps.claimPageUrl` — hand that one link to your human mid-conversation. When they claim it, the workspace becomes permanently theirs, they get a fresh key, and your sandbox key is revoked. Workspace ownership always ends with a human confirming; don't route around that.

**You want to read a workspace's own audit, cost, and policy data.** Add the ACP MCP server (streamable HTTP; OAuth sign-in happens in the human's browser):

```
https://api.agenticcontrolplane.com/mcp
```

Details, credential storage, and verification: [references/setup.md](references/setup.md).

## Step 3 — Check before sensitive actions

Before executing a sensitive tool call (deploys, deletes, sends, purchases, anything with blast radius), call `acp_check` with the tool name and its input. It returns the allow/deny decision your workspace policy would make, so you can adapt before acting instead of being stopped mid-flight. If a call is denied, the decision comes with a reason — surface it to your human rather than retrying around it.

The other MCP tools cover introspection: `acp_status` (connectivity + budget), `acp_audit` (your recent decisions), `acp_cost` (spend by model), `acp_optimize` (cost composition), `acp_trace` (one run's steps), `acp_usage` (plan usage), `acp_recommendations` (policy findings). All nine tools: eight read-only, plus `acp_propose_rule` (Step 5).

## Step 4 — Spawning subagents? Mint a child key

Every agent your agent creates can run under your identity with scopes a strict subset of yours:

```
POST https://api.agenticcontrolplane.com/api/v1/keys/child
```

The gateway intersects scopes with the parent, decrements the parent's budget atomically, preserves `originSub`, and produces audit logs that trace the chain back to the originating human. Give the child key to the subagent; never hand a subagent the parent key. Request/response shape, error codes, and rules: [references/child-keys.md](references/child-keys.md). Quickstart: https://agenticcontrolplane.com/agents/quickstart/

## Step 5 — Policy changes go through a human, always

To turn repeated friction into a standing rule (e.g. "always allow this command", "ask before that one"), call `acp_propose_rule`. It drafts the rule for a human to review and confirm in the console — **it never applies policy itself, and neither do you.** Never edit policy config, credentials, or hook wiring directly; hand your human the proposed rule and let the console flow finish it. That's the product working as intended.

## Reference

- [references/setup.md](references/setup.md) — install paths in detail, credential storage, verification, MCP tool roster
- [references/child-keys.md](references/child-keys.md) — child key endpoint shape, error codes, delegation rules
- Config-as-code (every dashboard setting as a REST endpoint): https://agenticcontrolplane.com/agents/configure-as-code/
- Machine-readable site index: https://agenticcontrolplane.com/llms.txt
- Source (MIT open core, GatewayStack): https://github.com/agentic-control-plane
- Delegation chain spec (ADCS): https://agenticcontrolplane.com/spec/delegation-chain
