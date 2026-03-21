# Multi-Agent Cost Tracking: The Observability Layer You're Missing

*March 21, 2026 · 7 min read*

You've deployed a multi-agent system. A planner agent delegates to a researcher, a writer, and a reviewer. It works beautifully in dev. Then the bill arrives: $47 for a single workflow run. Which agent caused it? You have no idea.

This is the observability gap in multi-agent AI systems. We instrument latency, errors, and throughput — but **per-agent cost attribution** is almost always missing.

## Why Multi-Agent Cost Tracking Is Different

Single-agent systems are simple: one model, one request, one cost. Multi-agent architectures break this model in three ways:

1. **Agent fan-out:** A planner spawns 5 researcher agents. Each researcher makes 3–10 API calls. Cost grows exponentially, not linearly.
2. **Model mixing:** Your planner uses GPT-4, researchers use GPT-3.5, the writer uses Claude. Each has different per-token pricing. Aggregating cost across providers is manual math.
3. **Retry cascades:** When an agent fails, the orchestrator retries — sometimes with a more expensive model. A single retry loop can 10x the cost of a workflow.

## The 3 Metrics Every Multi-Agent System Needs

### 1. Cost per Agent Role

Not just total cost — cost *by role*. Which agent type is your biggest spender?

```python
from tokenfence import guard

# Give each agent role its own budget
planner_client = guard(openai.OpenAI(), budget=1.00, label="planner")
researcher_client = guard(openai.OpenAI(), budget=2.00, label="researcher")
writer_client = guard(anthropic.Anthropic(), budget=1.50, label="writer")
```

### 2. Cost per Workflow Run

Individual API calls are noise. What matters is the **total cost of a complete workflow execution**.

```python
workflow_client = guard(openai.OpenAI(), budget=5.00, label="doc-summary-v2")
```

### 3. Cost Anomaly Detection

Set thresholds. If a workflow that normally costs $0.15 suddenly costs $3.00, that's a bug. TokenFence's budget caps act as automatic anomaly detection.

## The Architecture Pattern

```python
import openai
from tokenfence import guard

class AgentOrchestrator:
    def __init__(self, total_budget: float):
        self.base_client = openai.OpenAI()
        self.total_budget = total_budget

    def create_agent(self, role: str, budget_pct: float):
        agent_budget = self.total_budget * budget_pct
        return guard(self.base_client, budget=agent_budget, label=role)

    def run_workflow(self, task: str):
        planner = self.create_agent("planner", budget_pct=0.15)
        researcher = self.create_agent("researcher", budget_pct=0.50)
        writer = self.create_agent("writer", budget_pct=0.25)
        reviewer = self.create_agent("reviewer", budget_pct=0.10)
```

## What NOT to Do

- **Don't rely on monthly billing dashboards.** By the time you see the bill, the damage is done.
- **Don't give all agents the same budget.** Allocate proportionally to expected usage.
- **Don't ignore retry costs.** Retries are the #1 source of cost surprises.
- **Don't build custom tracking infrastructure.** Ship your product, not your cost tracker.

## Real-World Numbers

| Workflow Type | Agents | Avg API Calls | Avg Cost | P99 Cost |
|---|---|---|---|---|
| Document summary | 2 | 4–6 | $0.08 | $0.35 |
| Research + report | 4 | 15–30 | $0.45 | $3.20 |
| Code review pipeline | 3 | 8–12 | $0.22 | $1.80 |
| Customer support triage | 5 | 20–40 | $0.60 | $5.50 |

## Getting Started

```bash
pip install tokenfence
npm install tokenfence
```

Wrap each agent's LLM client in a `guard()` call with a budget. That's it.

Docs: https://tokenfence.dev/docs
Examples: https://github.com/u4ma-kev/tokenfence-examples
