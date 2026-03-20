# How to Set Per-Workflow Budget Limits on OpenAI API Calls

*Published on the TokenFence blog — tokenfence.dev/blog*

---

OpenAI's spending limits are account-wide. When your rogue agent hits the limit, **every** customer on your platform gets a 429 error. That's not a guardrail — it's a single point of failure.

What you actually need is **per-workflow budget control**. Here's how to do it in Python and TypeScript with TokenFence.

## The Problem: Account-Level Limits Don't Scale

If you're running AI agents in production, you've probably already hit this:

- **Agent A** handles customer support queries ($0.02 each)
- **Agent B** does document analysis ($0.50 each)  
- **Agent C** runs multi-step research workflows ($2-5 each)

With OpenAI's account spending limit set to $100/day, one malfunctioning Agent C can burn through the entire daily budget in minutes — taking Agents A and B offline with it.

Rate limits (requests per minute) don't help either. A single GPT-4o request with a 128K context window costs more than 1,000 GPT-4o-mini requests. Counting requests is not counting dollars.

## The Solution: Per-Workflow Budget Caps

TokenFence wraps your existing OpenAI or Anthropic client with three layers of protection:

### Layer 1: Budget Cap
```python
from tokenfence import guard
import openai

client = guard(openai.OpenAI(), budget="$0.50")
```

Every API call is tracked against a per-workflow budget. When the budget is reached, the behaviour depends on your `on_limit` setting.

### Layer 2: Auto-Downgrade
```python
client = guard(
    openai.OpenAI(),
    budget="$0.50",
    fallback="gpt-4o-mini",  # At 80% budget, switch to cheaper model
)
```

When your workflow has used 80% of its budget, TokenFence automatically downgrades to a cheaper model. Your agent keeps working, just more efficiently. The threshold is configurable.

### Layer 3: Kill Switch
```python
client = guard(
    openai.OpenAI(),
    budget="$0.50",
    on_limit="stop",  # At 100%, return synthetic response
)
```

At budget cap, instead of crashing or making an expensive call:
- `"stop"` — Returns a synthetic response. Your code keeps running.
- `"raise"` — Throws `BudgetExceeded`. You handle it.
- `"warn"` — Logs a warning, allows the call through.

## TypeScript Version

```typescript
import { guard } from "tokenfence";
import OpenAI from "openai";

const client = guard(new OpenAI(), {
  budget: "$0.50",
  fallback: "gpt-4o-mini",
  onLimit: "stop",
});

const res = await client.chat.completions.create({
  model: "gpt-4o",
  messages: [{ role: "user", content: "Analyze this data..." }],
});

// Check spend anytime
console.log(client.tokenfence.spent);    // 0.0023
console.log(client.tokenfence.remaining); // 0.4977
```

## Works with Anthropic Too

```python
import anthropic
from tokenfence import guard

client = guard(
    anthropic.Anthropic(),
    budget="$1.00",
    fallback="claude-3-haiku-20240307",
    on_limit="stop",
)

response = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Summarize this document..."}],
)
```

## Real-World Example: Multi-Agent Orchestration

Here's where per-workflow budgets really shine. In a multi-agent system, each agent gets its own budget:

```python
from tokenfence import guard
import openai

# Cheap agent for simple tasks
support_agent = guard(openai.OpenAI(), budget="$0.10", on_limit="stop")

# Medium budget for analysis
analysis_agent = guard(openai.OpenAI(), budget="$1.00", fallback="gpt-4o-mini")

# Higher budget for complex research
research_agent = guard(openai.OpenAI(), budget="$5.00", fallback="gpt-4o-mini")
```

If the research agent goes haywire, it burns through $5 max. The support and analysis agents keep working. No blast radius.

## Comparison: TokenFence vs. Alternatives

| Feature | OpenAI Limits | LangSmith | TokenFence |
|---------|--------------|-----------|------------|
| Per-workflow budgets | ❌ | ❌ | ✅ |
| Auto model downgrade | ❌ | ❌ | ✅ |
| Kill switch | Account-wide | ❌ | Per-workflow |
| Setup time | N/A | Hours | 2 lines |
| Framework lock-in | N/A | LangChain | None |

## Getting Started

```bash
pip install tokenfence        # Python
npm install tokenfence         # Node.js / TypeScript
```

Two lines to protect your entire AI budget. No framework lock-in. No infrastructure to deploy.

---

*TokenFence is the cost circuit breaker for AI agents. Learn more at [tokenfence.dev](https://tokenfence.dev).*
