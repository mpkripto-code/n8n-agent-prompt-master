# n8n Agent Prompt Master 🤖

> A research-backed framework for writing production-grade system prompts for n8n AI agents —
> covering context engineering, HITL gates, MCP security, Think Tool patterns, and multi-agent orchestration.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![n8n](https://img.shields.io/badge/n8n-workflow-EA4B71)](https://n8n.io)
[![2026 Edition](https://img.shields.io/badge/edition-2026-4A90D9)]()
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CHANGELOG.md)

---

## Why this exists

Generic prompt engineering advice **breaks in n8n agents**.

The ReAct loop, stateless execution, output parsers, tool schemas, and HITL gates create failure modes that no ChatGPT-style prompt guide addresses. Most tutorials show you how to write a prompt. None of them explain why your agent loops forever, calls the wrong tool, returns malformed JSON, or gets hijacked by data it retrieved.

This framework is built from:

- **Anthropic's Context Engineering research** (2025) — the shift from prompt to context
- **n8n official release notes** — including the Jan 2026 native HITL release
- **OWASP MCP Top 10** security framework (2025)
- **Real n8n production patterns** — what actually breaks at scale
- **Chroma & Databricks research** on context rot and attention degradation

---

## What's inside

```
n8n-agent-prompt-master/
├── SKILL.md                           ← Framework core: pipeline, 10 techniques, routing rules
├── references/
│   ├── n8n-agent-profiles.md          ← 14 agent configurations with exact prompt fixes
│   ├── n8n-agent-templates.md         ← 10 fill-in-the-blank templates
│   └── n8n-agent-patterns.md          ← 38 anti-patterns with before/after examples
└── examples/
    ├── rag-agent-prompt.md            ← RAG agent with grounded retrieval
    ├── orchestrator-prompt.md         ← Multi-agent orchestrator with routing map
    └── prompt-tuning-agent-prompt.md  ← Agent that rewrites other agents' prompts
```

---

## Quick diagnosis

**Agent calling the wrong tool?**
→ Anti-pattern #3: Ambiguous Tool Selection + Template: `ReAct-Bounded`

**Agent stuck in an infinite loop?**
→ Anti-pattern #1: No Loop Budget + add Loop Budget technique

**Returns prose when JSON is expected?**
→ Anti-patterns #11–#13: Output Contract failures

**Accuracy drops after 15+ turns?**
→ Anti-pattern #32: No Context Compaction + Context Engineering section in `SKILL.md`

**Agent skips a required step sometimes?**
→ That's not a prompt problem. Move the step to a deterministic n8n node.

**HITL denial triggers a retry?**
→ Anti-pattern #38 + Profile #14 + HITL Gate Declaration technique

**Tool output hijacks the agent?**
→ Anti-patterns #36–#37: Prompt Injection + Security-First template

---

## Key concepts (2025–2026)

### 1. Context Engineering > Prompt Engineering

The system prompt is one layer. You also control:
- What tool outputs return (pre-filter before they hit the agent)
- How the memory window is scoped (bigger ≠ better)
- What data passes between agents (identifiers, not full payloads)
- Where compaction triggers (before context rot sets in)

> *"Context engineering is the delicate art and science of filling the context window with just the right information for the next step."* — Andrej Karpathy

### 2. Deterministic Hybrid Architecture

The dominant production pattern in 2026: **deterministic n8n workflow nodes + narrow-scope AI agent**.

| If the behavior must... | Don't use | Use instead |
|---|---|---|
| Happen 100% of the time | `"always do X"` prompt rule | Deterministic n8n node |
| Happen with human judgment | `"ask user"` prompt rule | HITL gate on that tool |
| Route on a condition | Agent routing decision | If/Switch node before the agent |

**If your agent is skipping a required step: the fix is not a stronger prompt. Move that step to the workflow.**

### 3. Injection-Resistant by Default

OWASP ranks prompt injection as the #1 LLM vulnerability. Tool outputs, retrieved documents, and MCP server responses can contain hidden instructions that hijack agent behavior.

Every template in this framework includes a data security section. Use it.

### 4. HITL is Architecture, Not a Prompt Safeguard

n8n added native per-tool HITL in January 2026. When a tool has human review enabled, the workflow pauses until a human approves or denies the specific call. This is **deterministic** — not probabilistic like a `"please confirm before acting"` prompt rule.

When HITL gates are active, your system prompt **must** declare which tools are gated and how to handle denials. The agent doesn't know otherwise.

---

## The 10 techniques

| # | Technique | When to use |
|---|---|---|
| 1 | Role + Scope Declaration | Always — first line of every prompt |
| 2 | Minimal Viable Toolset | When 2+ tools have overlapping purpose |
| 3 | Tool Intent Anchoring | When the agent calls the wrong tool |
| 4 | Output Contract | Always — define the JSON schema |
| 5 | Loop Budget | Always — set a max tool call count |
| 6 | Escalation Path | Always — define what happens when stuck |
| 7 | HITL Gate Declaration | When any tool has human review enabled |
| 8 | Injection-Resistant Data Handling | When agent processes external data |
| 9 | Context Pruning Instructions | For agents that run more than 10 turns |
| 10 | Few-Shot Tool Examples | When tool selection is ambiguous |

---

## The templates at a glance

| Template | Use for |
|---|---|
| `ReAct-Bounded` | Standard AI Agent node with tools |
| `LLM-Chain-Compact` | Basic LLM Chain (no tools) |
| `Memory-Aware` | Agent with Window Buffer Memory |
| `Sub-Agent-Strict` | Called by an orchestrator |
| `Orchestrator-Router` | Routes tasks to sub-agents |
| `Tool-Annotated-ReAct` | Custom HTTP/Code tools with no auto-schema |
| `Prompt-Tuner` | Agent that rewrites other agents' prompts |
| `RAG-Grounded` | Agent with vector store retrieval |
| `Context-Engineered Multi-Agent` | Multi-agent with token cost concern |
| `Security-First` | Agent processing untrusted external data |

Full templates with fill-in-the-blank structure: [`references/n8n-agent-templates.md`](references/n8n-agent-templates.md)

---

## Ready-to-use examples

| Example | What it shows |
|---|---|
| [`examples/rag-agent-prompt.md`](examples/rag-agent-prompt.md) | RAG agent with grounding mandate, citation rules, single-retrieval rule |
| [`examples/orchestrator-prompt.md`](examples/orchestrator-prompt.md) | Orchestrator with explicit routing map, error forwarding, aggregation contract |
| [`examples/prompt-tuning-agent-prompt.md`](examples/prompt-tuning-agent-prompt.md) | Agent that receives a performance report and outputs an improved system prompt as JSON |

---

## How to use this as a Claude Skill

If you're using Claude (claude.ai or API), you can install this as a skill:

1. Copy the entire repo to `/mnt/skills/user/n8n-agent-prompt-master/`
2. Claude will automatically detect and load it when you ask about n8n agent prompts
3. The skill triggers on: `"write a system prompt for my n8n agent"`, `"my agent calls the wrong tool"`, `"fix my n8n agent prompt"`, and similar

---

## Contributing

Found a new anti-pattern in production? Have a template for a node type not covered? PRs welcome.

Format for new anti-patterns:
```
### N. Pattern Name
**Symptom:** What goes wrong and how you'd notice it.
**Before:** The prompt instruction that causes it (or the absence of one).
**After:** The fix.
```

---

## Author

**Never Navarro** — AI Engineer & n8n specialist from Montevideo 🇺🇾  
Co-founder [@Suelio](https://github.com/Suelio)  
→ [github.com/mpkripto-code](https://github.com/mpkripto-code)

---

## License

MIT — use it, fork it, improve it. See [LICENSE](LICENSE).
