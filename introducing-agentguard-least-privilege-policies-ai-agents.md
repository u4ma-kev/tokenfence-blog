# Introducing AgentGuard: Least-Privilege Policies for AI Agents — Because Prompts Are Not Permissions

*Published: 2026-03-22 | 10 min read*

**Tags:** AgentGuard, AI Safety, Least Privilege, Runtime Enforcement, Policy Engine, TokenFence

---

You wouldn't give a new hire admin access to your production database on day one. So why are we giving AI agents unrestricted access to every tool in the system and crossing our fingers that a prompt instruction will keep them in line?

After the Meta AI agent SEV1 incident and Grigorev's production database wipe, the industry has a painfully obvious lesson staring it in the face: **prompts are suggestions. Policies are enforcement.**

Today, we're shipping the **AgentGuard Policy Engine** — a new module in TokenFence that brings least-privilege enforcement to AI agents. Define what your agents can do, deny what they can't, require human approval for dangerous operations, and audit every decision.

## The Problem: Agents With Root Access

Most AI agent frameworks today give agents access to tools through function calling. The agent gets a list of available tools, picks which ones to call, and the framework executes them. The security model? A system prompt that says "be careful."

```python
# The "security model" most AI agents use today
system_prompt = """
You are a helpful database assistant.
You can read data and generate reports.
IMPORTANT: Never delete, drop, or modify any data.
"""

# But the agent has access to ALL tools...
tools = [
    read_database,
    write_database,
    delete_record,    # "But I told it not to!"
    drop_table,       # "The prompt said don't!"
    truncate_logs,    # "It shouldn't call this..."
]
```

This is the equivalent of giving someone the keys to your house and a sticky note that says "please don't go in the bedroom." It might work most of the time. But when it fails, it fails catastrophically.

## What Went Wrong: The March 2026 Incidents

Two high-profile incidents in March 2026 proved this isn't theoretical:

- **Meta's AI Agent SEV1** — An autonomous AI agent in Meta's internal systems triggered a cascade of unintended actions that required an emergency response.
- **Grigorev Database Wipe** — A developer's AI coding agent, given broad tool access for a refactoring task, executed destructive database commands on a production system.

In both cases, the pattern is identical: the agent had the capability to do dangerous things, and a prompt-level instruction was the only "guardrail."

## The Solution: Runtime Policy Enforcement

AgentGuard brings the principle of least privilege to AI agents. Instead of trusting prompt instructions, you declare a policy — a machine-enforced contract.

```python
from tokenfence import Policy

policy = Policy(
    allow=["read_database", "generate_report", "list_tables"],
    deny=["delete_*", "drop_*", "truncate_*", "alter_*"],
    require_approval=["write_database", "create_table"],
    name="database-readonly-agent",
)

result = policy.check("read_database")
assert result.allowed  # ✅ Explicitly permitted

result = policy.check("drop_table")
assert result.denied   # 🚫 Blocked — no prompt can override this

result = policy.check("write_database")
assert result.needs_approval  # ⏸️ Requires human confirmation
```

## Three Layers of Protection

### 1. Allow/Deny Lists with Wildcards

```python
policy = Policy(
    allow=["read_*", "list_*", "search_*"],
    deny=["delete_*", "drop_*", "rm_*"],
    default="deny",  # Deny everything not explicitly allowed
)
```

### 2. Approval Gates

```python
policy = Policy(
    allow=["*"],
    deny=["drop_*", "truncate_*"],
    require_approval=["send_email", "publish_post", "transfer_funds"],
    on_approval=lambda result: ask_human(f"Allow {result.tool}?"),
)
```

### 3. Full Audit Trail

```python
for entry in policy.audit_log:
    print(f"{entry.tool} → {entry.decision} ({entry.reason})")
```

## Real-World Policy Templates

### Database Agent (Post-Grigorev)

```python
db_policy = Policy(
    allow=["read_database", "generate_report", "list_tables"],
    deny=["delete_*", "drop_*", "truncate_*", "alter_*"],
    require_approval=["write_database", "create_table"],
)
```

### Email Agent

```python
email_policy = Policy(
    allow=["read_email", "search_email", "draft_email"],
    deny=["delete_email_*", "forward_to_external"],
    require_approval=["send_email"],
)
```

### Code Review Agent

```python
review_policy = Policy(
    allow=["read_file", "list_files", "git_diff", "run_tests"],
    deny=["git_push", "git_force_*", "rm_*", "deploy_*"],
    require_approval=["git_commit", "create_pr"],
)
```

## Policy-as-Code

```python
import json

# Save
config = policy.to_dict()
with open("policies/database-agent.json", "w") as f:
    json.dump(config, f, indent=2)

# Load
policy = Policy.from_dict(json.load(open("policies/database-agent.json")))
```

## Combined with Budget Caps: Defense in Depth

```python
from tokenfence import guard, Policy

# Layer 1: Budget cap
client = guard(openai.OpenAI(), budget=5.00, fallback="gpt-4o-mini")

# Layer 2: Policy enforcement
policy = Policy(
    allow=["read_database", "generate_report"],
    deny=["delete_*", "drop_*"],
)

policy.enforce(tool_name)  # Blocks unauthorized tools
response = client.chat.completions.create(...)  # Caps spending
```

## Get Started

```bash
pip install tokenfence
```

96 tests passing. Zero dependencies. MIT licensed. Built for developers who learned from March 2026 that prompts are suggestions — but policies are law.

---

*Read more at [tokenfence.dev/blog](https://tokenfence.dev/blog)*
