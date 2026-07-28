# Child keys — subagents under scope, budget, and audit

Mint a scope-narrowed child API key for any subagent you spawn. Every tool call the subagent makes runs under the child key; audit rows trace through the chain back to the originating human (`originSub`); its effective scopes are a strict subset of yours; its budget is decremented atomically from yours.

## Endpoint

```
POST https://api.agenticcontrolplane.com/api/v1/keys/child
Authorization: Bearer <parent key>
Content-Type: application/json
{
  "profileId": string,          // required; profile must have delegatable: true
  "scopes": string[]?,          // optional further narrowing beyond profile.scopes
  "maxBudgetCents": number?,    // optional cap; clamped to min(parent, profile, request)
  "ttlSeconds": number?,        // optional 60..86400, default 3600
  "reason": string?             // optional audit label
}
```

Response (`201`):

```
{
  "apiKey": "gsk_...",          // shown ONCE — use it, never persist it
  "keyId": "...",
  "expiresAt": "ISO timestamp",
  "effectiveScopes": ["..."],   // post-intersection, frozen at mint
  "effectiveTools": ["..."],
  "remainingBudgetCents": number,
  "chain": {
    "originSub": "...",
    "depth": number,
    "agentProfileId": "...",
    "agentRunId": "...",
    "parentKeyId": "..."
  }
}
```

## Error codes

| Code | Meaning | What to do |
|---|---|---|
| `400 validation_failed` | Body shape wrong | Check `details` |
| `403 profile_not_delegatable` | Profile lacks `delegatable: true` | Ask the human to enable it on the profile |
| `404 profile_not_found` | Typo, or wrong tenant | Verify the profile id |
| `409 delegation_cycle` | Profile already in this chain | Pick a different profile or restructure |
| `409 delegation_depth_exceeded` | Chain at the depth cap (5) | Restructure |
| `410 parent_key_already_expired` | Parent key expired | Re-mint the parent key first |
| `429 child_mint_rate_limit` | 30 mints/hour per parent | Back off |

## Rules

- **Never** run a subagent on the parent key. Narrowed scopes, per-subagent budget, and chain attribution only exist if the child key is what the subagent uses.
- **Never** persist a child key past the subagent's session. It expires on its own; mint a fresh one for a later run.
- **Never** pass `originSub` in the request body. The originating identity comes from the parent's chain; the endpoint rejects unknown fields.
- **Don't** skip minting because the subagent's scopes match yours — even identical scopes get per-subagent budget and audit correlation for one HTTP round-trip.
- Narrower is always better: request only the scopes, budget, and TTL the task needs.

## After the run

Audit rows produced by the subagent carry `originSub` (the originating human), `depth` (≥ 2 for the subagent), and `chain` (agent profile ids through the chain) — verify via `acp_audit` or the console.

Quickstart and SDK helpers: https://agenticcontrolplane.com/agents/quickstart/
Full field-level reference: https://agenticcontrolplane.com/agents/configure-as-code/
