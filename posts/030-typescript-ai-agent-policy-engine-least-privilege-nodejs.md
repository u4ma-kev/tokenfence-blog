# Ship Least-Privilege AI Agents in TypeScript: The TokenFence Policy Engine Hits Node.js

*Published: 2026-03-22 | 8 min read*

*Tags: TypeScript, Node.js, AgentGuard, Policy Engine, AI Safety, Least Privilege, TokenFence*

---

The AgentGuard Policy engine that Python developers have been using is now available in TypeScript. Define allow/deny/requireApproval patterns, get full audit trails, and enforce least-privilege for your AI agents — all with zero dependencies.

## The Policy Engine Crosses the Language Barrier

Two days ago, we shipped the AgentGuard Policy engine in TokenFence's Python SDK. The response was immediate: "When is this coming to Node.js?"

Today. Right now. `npm install tokenfence@0.2.0`.

If you're building AI agents in TypeScript — whether with the Vercel AI SDK, LangChain.js, or raw OpenAI/Anthropic clients — you now have the same least-privilege enforcement that Python developers get. Zero dependencies. Full TypeScript types. 53 tests passing.

## Why AI Agents Need Policies, Not Just Prompts

The pattern is becoming painfully clear in 2026:

1. Developer builds an AI agent with tool access
2. Agent gets system prompt: "You can read the database but never delete anything"
3. Agent encounters an edge case the prompt didn't cover
4. Data is lost, emails are sent, files are deleted

Prompts are *suggestions*. Policies are *enforcement*. The Meta AI agent incident, the Grigorev database wipe — both would have been prevented by a deny list that the agent literally cannot bypass.

## How It Works in TypeScript

```typescript
import { Policy } from "tokenfence";

const policy = new Policy({
  name: "database-agent",
  allow: ["db_read_*", "db_list_*", "db_count_*"],
  deny: ["db_delete_*", "db_drop_*", "db_truncate_*"],
  requireApproval: ["db_write_*", "db_update_*"],
  default: "deny",
});

const result = policy.check("db_read_users");
console.log(result.allowed);  // true

const dangerous = policy.check("db_drop_table");
console.log(dangerous.denied);  // true
```

## Enforce Mode: Throw on Denied

```typescript
import { Policy, ToolDenied } from "tokenfence";

const policy = new Policy({
  allow: ["read_*", "search_*"],
  deny: ["delete_*", "drop_*"],
});

try {
  policy.enforce("delete_user_data");
} catch (e) {
  if (e instanceof ToolDenied) {
    console.log(e.tool);        // "delete_user_data"
    console.log(e.reason);      // "denied by pattern: delete_*"
  }
}
```

## Full Python + Node.js Parity

| Feature | Python SDK | Node.js SDK |
|---------|-----------|-------------|
| Allow/Deny/RequireApproval | ✅ v0.3.0 | ✅ v0.2.0 |
| Glob wildcards (* and ?) | ✅ | ✅ |
| Deny-by-default | ✅ | ✅ |
| Approval callbacks | ✅ | ✅ |
| Full audit trail | ✅ | ✅ |
| enforce() with exceptions | ✅ | ✅ |
| Serialization | ✅ | ✅ |
| Zero dependencies | ✅ | ✅ |
| Full type annotations | ✅ | ✅ |

## Get Started

```bash
npm install tokenfence
```

TokenFence v0.2.0: cost circuit breaker + least-privilege policy engine. Because your AI agents should have budgets *and* boundaries.

---

*[TokenFence](https://tokenfence.dev) — the cost circuit breaker and policy engine for AI agents.*
