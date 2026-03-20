# Why Your AI Agents Need a Cost Kill Switch

*Your agent just burned through $400 in tokens. Here's how to make sure that never happens again.*

---

## The Problem Nobody Talks About

AI agents are incredible. They reason, plan, call tools, and execute multi-step workflows autonomously. But there's a dirty secret in every production AI deployment:

**Nobody knows what their agents will cost until they get the bill.**

A single runaway agent loop can burn through hundreds or thousands of dollars in minutes. A recursive tool-calling chain that hits an edge case. A retry loop that doesn't back off. A user prompt that triggers 47 sub-agent spawns.

It's not hypothetical. It's happening right now, every day, at companies running production AI agents.

## The Real Numbers

Let's do some quick math:

- GPT-4o: $2.50/1M input + $10/1M output tokens
- Claude Opus 4: $15/1M input + $75/1M output tokens  
- A complex agent workflow can easily consume 500K-2M tokens per run
- That's **$1.25–$150 per workflow execution**

Now multiply by:
- 100 concurrent users = $125–$15,000/hour
- One infinite loop = **unlimited spend** until you manually kill it

Most teams discover this the hard way — via an AWS bill that makes the CFO call an emergency meeting.

## Why Rate Limits Aren't Enough

"But I set rate limits on my API!" Great. Rate limits protect the *provider*. They don't protect *your budget*.

Rate limits tell the API: "Don't serve more than X requests per minute."  
A cost circuit breaker tells your agent: "Don't spend more than $5 on this task."

These are fundamentally different controls:

| Control | Protects | Granularity | Agent-Aware? |
|---------|----------|-------------|--------------|
| API Rate Limit | Provider | Per-account | No |
| Monthly Spend Cap | Your wallet | Per-month | No |
| **Cost Circuit Breaker** | **Your workflow** | **Per-task** | **Yes** |

## What a Cost Kill Switch Actually Does

A proper cost circuit breaker for AI agents needs three things:

### 1. Per-Workflow Budgets
Not per-month. Not per-account. Per *individual workflow execution*. Your summarization agent gets $0.50. Your code review agent gets $2.00. Your research agent gets $5.00.

```python
from tokenfence import TokenFence

fence = TokenFence(budget=2.00)  # $2 max for this workflow

# Every API call is tracked automatically
response = fence.guard(
    client.chat.completions.create,
    model="gpt-4o",
    messages=[{"role": "user", "content": prompt}]
)
```

### 2. Automatic Model Downgrade
When you hit 80% of your budget, automatically downgrade to a cheaper model. Your agent keeps working, just more efficiently.

```python
fence = TokenFence(
    budget=5.00,
    on_limit="downgrade",
    downgrade_map={"gpt-4o": "gpt-4o-mini"}
)
# At 80% spend: silently switches to gpt-4o-mini
# Agent doesn't even notice
```

### 3. Hard Kill Switch
At 100% budget, stop. No exceptions. No "just one more call." Done.

```python
fence = TokenFence(budget=1.00, on_limit="stop")
# At $1.00 spent: raises BudgetExceeded
# Your orchestrator catches it and handles gracefully
```

## The Subagent Problem

This gets worse with multi-agent systems. When GPT-5 mini/nano agents are spawning sub-agents, and those sub-agents spawn more sub-agents, cost compounds exponentially.

Without per-workflow budgets, a single user request can trigger a cascade:
- Agent A calls Agent B (3 times)
- Agent B calls Agent C (2 times each)
- Agent C makes 5 API calls each
- Total: **30 API calls** from one user request

TokenFence tracks cost across the entire workflow tree, not just individual calls.

## Getting Started

Install:
```bash
pip install tokenfence
```

Two lines to protect any workflow:
```python
from tokenfence import TokenFence

fence = TokenFence(budget=5.00)
response = fence.guard(client.chat.completions.create, model="gpt-4o", messages=messages)
```

That's it. Your agent now has a budget. When it's spent, it stops (or downgrades, your choice).

## What's Next

TokenFence is in early access. We're building:
- **Dashboard** for real-time spend visibility across all your agents
- **Team budgets** for multi-user environments
- **Alerts** via Slack/Discord/email when agents approach limits
- **Node.js SDK** (coming soon)

The future of AI agents is autonomous. The future of AI *cost control* is TokenFence.

---

*TokenFence is the cost circuit breaker for AI agents. Per-workflow budgets, automatic model downgrade, and a hard kill switch — in 2 lines of code.*

*[Get started →](https://tokenfence.dev)*
