# Changelog

All notable changes to this project will be documented in this file.

---

## [1.0.0] — 2026-05-10

### Added

- `SKILL.md` — Core framework: 10-technique pipeline, template routing rules, context engineering section, security-first defaults
- `references/n8n-agent-profiles.md` — 14 agent configurations with exact prompt fixes (AI Agent, Basic LLM Chain, Memory-Aware, Sub-Agent, Orchestrator, Tool-Calling, Self-Refinement, Prompt Tuning, RAG, Email Delivery, Data Transformation, Think Tool, HITL)
- `references/n8n-agent-templates.md` — 10 fill-in-the-blank templates: ReAct-Bounded, LLM-Chain-Compact, Memory-Aware, Sub-Agent-Strict, Orchestrator-Router, Tool-Annotated-ReAct, Prompt-Tuner, RAG-Grounded, Context-Engineered Multi-Agent, Security-First
- `references/n8n-agent-patterns.md` — 38 anti-patterns with before/after fixes across 7 categories: Loop & Tool Misuse, Output Contract Failures, Scope & Role Failures, Memory & State Failures, Token & Performance Failures, Context Engineering Failures, Security Failures
- `examples/rag-agent-prompt.md` — Ready-to-use RAG agent prompt with grounding mandate, citation rules, single-retrieval rule, and common issue fixes
- `examples/orchestrator-prompt.md` — Ready-to-use multi-agent orchestrator prompt with 3-sub-agent routing map, aggregation contract, and error handling
- `examples/prompt-tuning-agent-prompt.md` — Ready-to-use prompt tuning agent with input/output schema, n8n wiring guide, and self-improvement loop pattern

### Framework coverage

- Anthropic Context Engineering research (2025)
- n8n native HITL gates (January 2026 release)
- OWASP MCP Top 10 security framework (2025)
- Think Tool patterns and known n8n bugs (early 2026)
- Deterministic Hybrid Architecture pattern

---

## Versioning

This project follows [Semantic Versioning](https://semver.org/).

- **MAJOR** version — breaking changes to template structure or SKILL.md pipeline
- **MINOR** version — new templates, profiles, or anti-patterns added
- **PATCH** version — fixes to existing content, typos, clarifications
