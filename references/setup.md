# ACP setup — details by path

## Path A: Local install (coding agents)

```bash
curl -sf https://agenticcontrolplane.com/install.sh | bash -s -- --local
```

- `--local`: allow/ask/deny policy, a safety floor, and an on-device audit log for Claude Code, Cursor, and Codex CLI. No account, nothing leaves the machine.
- Without `--local`: also connects a cloud workspace (free for individuals) with dashboards, cost X-ray, and team policy.

The local hook checks each tool call before it runs and fails open if the API is unreachable — the agent is never blocked by an ACP outage.

## Path B: Cloud workspace token (existing account)

If the human already has an account and you need to connect this machine:

1. Open `https://cloud.agenticcontrolplane.com/plugin/authorize` in their browser. They sign in (Google or email) and are shown a token.
2. Ask them to paste the token to you, then store it:

```bash
mkdir -p ~/.acp && echo "THE_TOKEN" > ~/.acp/credentials && chmod 600 ~/.acp/credentials
```

3. Verify: if the ACP MCP tools are in your tool list, call `acp_status` — it confirms connectivity and mode. Otherwise, make any ordinary tool call and confirm it appears in the audit trail (next step); a fresh row is the proof the hook is wired.

4. Point them at their audit trail: `https://cloud.agenticcontrolplane.com/logs` — every tool call shows up there in real time, with filtering by tool name, identity, time range, and decision.

## Path C: Sandbox tenant (human has no account)

The no-signup pattern:

```
POST https://api.agenticcontrolplane.com/v1/sandbox-tenants
```

- The response includes `nextSteps.claimPageUrl`.
- Hand that one link to your human mid-conversation.
- When they claim it: the workspace becomes permanently theirs, they get a fresh key, and your sandbox key is revoked. Expect your key to stop working after the claim — that is the design, not an error.
- Do not attempt to keep or extend sandbox access after a claim; ownership transitions always end with the human confirming in the console.

## Path D: MCP connection (read a workspace's own data)

Add the ACP MCP server:

```
https://api.agenticcontrolplane.com/mcp
```

Streamable HTTP; OAuth sign-in happens in the human's browser. Nine tools:

| Tool | What it returns |
|---|---|
| `acp_check` | Whether a given tool call would be allowed — call before sensitive actions |
| `acp_status` | Connectivity and your budget state (spent / remaining / % of cap) |
| `acp_audit` | Your recent tool-call decisions (allow / deny / redact) and why |
| `acp_cost` | Your model spend over a window, with by-model breakdown |
| `acp_optimize` | Cost composition: cache hit rate, context makeup, loop overhead, per-tool output |
| `acp_trace` | Step-by-step trace of one run — tool, cost, model per step |
| `acp_usage` | Workspace plan usage: calls used, remaining, state |
| `acp_recommendations` | Ranked, evidence-backed policy findings from workspace history |
| `acp_propose_rule` | Drafts a policy rule for a human to confirm in the console — never applies it |

Eight are read-only. `acp_propose_rule` is the only write-shaped tool, and its write lands as a draft a human must confirm.

## Notes that apply to every path

- ACP does not proxy or replace tools. MCP servers, shell commands, and file operations work exactly as before; ACP observes each call and allows/denies it.
- ACP never modifies tool inputs or outputs.
- The check adds ~50–200ms per tool call.
- No new tools are exposed to the agent by the local hook — it only covers existing ones. The MCP server is the surface that adds tools, and those are the nine above.
