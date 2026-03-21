# How to Add Budget Limits to LangChain, CrewAI, and AutoGen Agents

*Published: March 21, 2026 · 8 min read*

LangChain, CrewAI, and AutoGen make it easy to build multi-agent systems. But none of them ship with per-workflow budget controls. Here's how to fix that.

## The Multi-Agent Cost Problem

Multi-agent frameworks are exploding in popularity. LangChain has 100K+ GitHub stars. CrewAI crossed 50K. AutoGen is Microsoft's flagship agent framework. They all share one critical gap:

**No built-in way to set a dollar budget on a workflow.**

When you run a CrewAI crew with 4 agents, each agent makes independent LLM calls. A "research agent" might call GPT-4o 30 times. A "writer agent" might call Claude 3.7 Sonnet 15 times. The orchestrator has no idea what the total cost is until after the fact.

## Adding Budget Caps to LangChain

```python
from tokenfence import guard
import openai

guarded_client = guard(
    openai.OpenAI(),
    budget="$2.00",
    fallback="gpt-4o-mini",
    on_limit="stop"
)
```

## Adding Budget Caps to CrewAI

```python
from crewai import Agent, Crew
from tokenfence import guard
import openai

client = guard(openai.OpenAI(), budget="$5.00", fallback="gpt-4o-mini", on_limit="stop")
```

## Async Support (New in 0.2.0)

```python
from tokenfence import async_guard
import openai

client = async_guard(openai.AsyncOpenAI(), budget="$1.00", fallback="gpt-4o-mini")
```

## Get Started

```bash
pip install tokenfence
```

Read the [full documentation](https://tokenfence.dev/docs) or browse [examples on GitHub](https://github.com/u4ma-kev/tokenfence-examples).
