# LlamaIndex Cost Control: How to Budget Your RAG Pipelines and Data Agents Before Retrieval Bills Spiral

*Published: March 22, 2026 · 9 min read*

LlamaIndex is the leading framework for retrieval-augmented generation (RAG) and data agents in 2026. Over 40,000 GitHub stars, deep integrations with every major vector store and LLM provider, and it's the default starting point for any project that needs to connect AI to your data.

The cost problem? LlamaIndex pipelines call LLMs in places you don't expect — and each call compounds with retrieved context.

## What a Typical LlamaIndex RAG Query Actually Costs

- Query embedding: ~100 tokens → $0.0001
- Vector retrieval: free (database lookup)
- Retrieved chunks stuffed into prompt: 4-8 chunks × 500-1,000 tokens = 2,000-8,000 tokens
- Synthesis LLM call: 8,000+ input tokens + 500-1,500 output tokens
- **Total per query with GPT-4o: ~$0.03-$0.06**

500 queries/day across your team? **$15-$30/day. $450-$900/month.** Before sub-question decomposition, reranking, or agent loops.

## The Five Cost Traps in LlamaIndex

### Trap 1: Sub-Question Query Engine Multiplication

`SubQuestionQueryEngine` decomposes complex queries into 3-5 sub-questions. Each hits a different index. One user query becomes 4-6 LLM calls. That $0.04 query becomes $0.20.

### Trap 2: Tree Summarize Response Mode

`tree_summarize` recursively summarizes retrieved chunks. With 20 chunks, that's 4-5 LLM calls just for summarization. Switch from `compact` to `tree_summarize` and costs jump 4x.

### Trap 3: Data Agent Tool Loops

LlamaIndex Data Agents autonomously decide which tools to call. Ambiguous results trigger retries. Each iteration includes full conversation history plus all tool outputs. One complex question can trigger 8-12 LLM calls.

### Trap 4: Embedding Costs at Scale

Every document needs embeddings. Every query needs an embedding. With OpenAI's `text-embedding-3-large` at $0.13/1M tokens, 100K documents cost ~$2.60 to embed. Re-indexing weekly? $10.40/month on embeddings alone.

### Trap 5: Chat Engine Memory Growth

Chat engines maintain conversation history. After 15 turns, you're sending 20,000+ tokens of history with every message. A Q&A bot can accumulate $5-$10/user/month from history bloat.

## Adding Budget Limits with TokenFence

```python
from tokenfence import guard
import openai

# Budget-protected LLM client
guarded_client = guard(openai.OpenAI(), budget=2.00)

# LlamaIndex uses this for ALL LLM calls
from llama_index.core import VectorStoreIndex, SimpleDirectoryReader

documents = SimpleDirectoryReader("./data").load_data()
index = VectorStoreIndex.from_documents(documents)
query_engine = index.as_query_engine()

response = query_engine.query("What were Q1 revenue trends?")
# TokenFence tracks every call, enforces the $2.00 cap
```

## Response Mode Cost Comparison

| Response Mode | LLM Calls | Cost per Query (GPT-4o) | When to Use |
|---|---|---|---|
| `compact` | 1-2 | $0.03-$0.06 | Default. Good enough for most queries. |
| `refine` | N (one per chunk) | $0.10-$0.40 | High-accuracy needs. Budget carefully. |
| `tree_summarize` | log(N) | $0.08-$0.20 | Large document sets. Watch for recursion. |
| `simple_summarize` | 1 | $0.02-$0.04 | Cheapest. Truncates to fit context. |
| `accumulate` | N | $0.08-$0.30 | Per-chunk answers. Expensive. |

**Rule of thumb:** Start with `compact`. Use `tree_summarize` only when quality requires it. Never use `refine` without a budget cap.

## The LlamaIndex Cost Control Checklist

1. **Wrap every LLM client with TokenFence** — per-pipeline budget caps are non-negotiable
2. **Audit your response mode** — `compact` is 2-10x cheaper than `refine`
3. **Count your sub-questions** — `SubQuestionQueryEngine` multiplies costs 3-5x
4. **Cap retrieved chunks** — `similarity_top_k=4` instead of 8 cuts context costs in half
5. **Cache embeddings** — never re-embed the same document twice
6. **Set conversation history limits** — truncate after 10 turns
7. **Enable model downgrade** — auto-switch to cheaper models before hitting limits
8. **Monitor per-user spend** — one power user can consume 80% of your budget

---

*TokenFence adds per-workflow budget caps, automatic model downgrade, and kill switches to any LLM client — including LlamaIndex RAG pipelines and data agents. Three lines of Python. Open source core. `pip install tokenfence`*

*Blog #41 in the TokenFence series. Sixth in the framework-specific series (CrewAI → AutoGen → LangChain → Semantic Kernel → Haystack → LlamaIndex).*
