# n8n Agent Profiles

Profiles for 12 n8n AI agent configurations. Each profile covers: what it does, what breaks it,
and the specific prompt fixes required.

---

## 1. AI Agent Node (LangChain / ReAct)

**What it does:** Autonomous reasoning loop with tool calls. Repeats Thought→Action→Observation
until it reaches a final answer or hits the iteration cap.

**What breaks it:**
- No tool call budget → runaway loops burning tokens
- Ambiguous tool descriptions → wrong tool called
- Missing output schema → freeform text when JSON is expected
- Instructions written for single-shot LLMs (ChatGPT style)

**Prompt fixes:**
- Open with role + explicit tool inventory: *"You have access to: [tool1], [tool2]. Use no other tools."*
- Set loop budget: *"Complete this task in at most 5 tool calls. If not resolved by then, return the error JSON."*
- Add tool-specific rules: *"Call search_web ONLY when the user's query requires information beyond your training data."*
- Close with output contract matching the Structured Output Parser schema

**Template:** `ReAct-Bounded`

---

## 2. Basic LLM Chain Node

**What it does:** Single prompt-in, text-out. No tools. No loop. Simplest n8n AI node.

**What breaks it:**
- Instructions written as if tools are available (they're not)
- No output format spec → unpredictable structure for downstream nodes
- Padding inflates token cost with no benefit

**Prompt fixes:**
- No tool instructions whatsoever
- Explicit output format at the end: *"Return ONLY a JSON object. No markdown. No commentary."*
- Use CO-STAR or RTF template — keep it under 200 tokens

**Template:** `LLM-Chain-Compact`

---

## 3. Agent with Window Buffer Memory

**What it does:** AI Agent that remembers recent conversation turns via a Window Buffer Memory node.

**What breaks it:**
- Prompt assumes unlimited memory (only last N turns are visible)
- No contradiction handling when memory conflicts with current input
- Agent re-asks for info already in memory window

**Prompt fixes:**
- Declare memory scope: *"You have access to the last {windowSize} conversation turns. Do not ask for information already present in your memory."*
- Add contradiction rule: *"If current input contradicts memory, prioritize current input and note the discrepancy."*
- Never instruct the agent to 'remember' things beyond the window

**Template:** `Memory-Aware`

---

## 4. Sub-Agent (Orchestrated)

**What it does:** A specialized agent called by an orchestrator. Receives structured input,
performs one job, returns structured output. Should never do anything outside its scope.

**What breaks it:**
- Scope creep: sub-agent tries to solve the full problem
- Ambiguous output schema: orchestrator can't parse the response
- Sub-agent asks clarifying questions (should return error JSON instead)

**Prompt fixes:**
- Hard scope declaration: *"Your ONLY task is [X]. Do not attempt to do [Y] or [Z]."*
- Strict output contract with no optional fields
- Replace clarifying questions with error output: *"If required input is missing, return: {\"error\": \"missing_field\", \"field\": \"<field_name>\"}"*

**Template:** `Sub-Agent-Strict`

---

## 5. Orchestrator Agent

**What it does:** Receives a high-level task, decomposes it, routes subtasks to sub-agents
or tools, and aggregates results into a final output.

**What breaks it:**
- No routing rules → arbitrary sub-agent selection
- No aggregation contract → output varies per run
- Infinite routing loops when sub-agents return errors

**Prompt fixes:**
- Explicit routing map: *"For tasks involving [X], call sub-agent-A. For tasks involving [Y], call sub-agent-B."*
- Aggregation contract: define how to combine sub-agent outputs
- Error handling: *"If a sub-agent returns an error field, log it and continue with remaining subtasks. Include errors in the final summary."*
- Max routing depth: *"Do not route the same subtask to a sub-agent more than 2 times."*

**Template:** `Orchestrator-Router`

---

## 6. Tool-Calling Agent (Custom Tools via HTTP/Code Nodes)

**What it does:** Agent with tools defined via HTTP Request nodes or Code nodes. Tool schemas
are custom — the agent must infer argument requirements from the system prompt.

**What breaks it:**
- Agent guesses argument names (no schema visible)
- Missing required arguments → tool errors → retry loop
- Agent calls tool with null/undefined fields

**Prompt fixes:**
- Embed tool signatures directly in the system prompt:
  ```
  Tool: search_crm
  Required args: { "query": string, "limit": number (max 10) }
  Returns: { "results": array of { "id", "name", "email" } }
  Call ONLY when: user asks about a specific contact or company
  ```
- Add argument validation rule: *"Never call a tool with a null or undefined argument. If a required argument is unavailable, use the escalation path instead."*

**Template:** `Tool-Annotated-ReAct`

---

## 7. Self-Refinement Loop Agent

**What it does:** Agent generates output → evaluates it against criteria → refines → repeats
until quality threshold met or iteration cap reached.

**What breaks it:**
- No evaluation criteria → agent always accepts first output
- No iteration cap → infinite loop
- Evaluation prompt same as generation prompt → no real refinement

**Prompt fixes:**
- Separate evaluation persona: *"Switch to critic mode. Evaluate the previous output against these criteria: [list]. Score 1–5 on each."*
- Hard cap: *"Stop after 3 refinement cycles regardless of score."*
- Acceptance threshold: *"If all scores are ≥ 4, accept the output and stop refining."*

**Template:** `Self-Refinement`

---

## 8. Prompt Tuning Agent (danielrosehill Pattern)

**What it does:** Receives a structured report about an agent's current behavior (working/not
working + current prompt) and outputs an improved system prompt + change summary as JSON.

**What breaks it:**
- Output parser not wired → freeform text response breaks downstream nodes
- Agent invents new agent capabilities not present in the original
- Agent rewrites working sections unnecessarily
- Markdown fences inside JSON string values break parsing

**Prompt fixes:**
- Strict preservation rule: *"Do not modify behaviors listed under 'working well'."*
- Capability boundary: *"Do not add tools, integrations, or behaviors not described in the original prompt."*
- JSON purity rule: *"Return ONLY the JSON object. No markdown. No commentary. No code fences."*
- Wire a Structured Output Parser node with exact schema

**Template:** `Prompt-Tuner`

---

## 9. RAG Agent (Retrieval-Augmented Generation)

**What it does:** Agent retrieves context from a vector store (Pinecone, Supabase pgvector, etc.)
before answering. Tool is the retrieval node.

**What breaks it:**
- Agent answers from training data instead of retrieved context
- Agent calls retrieval tool too many times for the same query
- Hallucinated citations not caught

**Prompt fixes:**
- Grounding mandate: *"Answer ONLY using information returned by the retrieve_context tool. If context is empty or irrelevant, say: 'I could not find relevant information.'"*
- Single retrieval rule: *"Call retrieve_context once per user question. Do not call it multiple times for the same question."*
- Citation rule: *"Always reference the source returned in the context metadata."*

**Template:** `RAG-Grounded`

---

## 10. Email/Notification Delivery Agent

**What it does:** Processes data, formats a message, sends via Gmail/Outlook/Slack node.
Common final step in many n8n workflows.

**What breaks it:**
- Agent formats HTML inside JSON string (escape issues)
- Agent adds commentary after the JSON output → breaks parser
- Template variables not substituted (agent outputs literal `{{ }}` syntax)

**Prompt fixes:**
- Output the message body as a plain string, not HTML, unless explicitly required
- If HTML is required: *"Escape all double quotes inside HTML strings using \". Do not include newlines inside JSON string values."*
- Variable substitution rule: *"Replace all placeholder variables with actual values from the input. Never output literal template syntax."*

**Template:** `Delivery-Agent`

---

## 11. Data Transformation Agent

**What it does:** Receives structured data (JSON, CSV, table), transforms or enriches it,
returns transformed data for downstream nodes.

**What breaks it:**
- Agent changes field names unprompted → breaks downstream node mappings
- Agent drops fields not explicitly mentioned
- Agent adds commentary before/after the JSON

**Prompt fixes:**
- Schema preservation: *"Return all input fields unless explicitly instructed to remove them. Do not rename fields."*
- Transformation-only rule: *"Add the following computed fields: [list]. Do not modify any other fields."*
- Output purity: *"Return ONLY the transformed JSON array/object. No commentary."*

**Template:** `Data-Transform`

---

## 12. Unknown n8n Agent Type (Fingerprint)

When the user's agent type is not in this list, ask these 4 questions:

1. Which n8n node is the agent running in? (AI Agent / Basic LLM Chain / Custom Code)
2. What tools or sub-nodes are connected to it?
3. What does the output feed into next? (Email / Database / Another agent / Webhook response)
4. Does it need to handle errors gracefully or can it fail loudly?

Use answers to select the closest template and adapt.

---

## 13. Agent with Think Tool

**What it does:** AI Agent node with the Think Tool sub-node connected. The agent uses the
think tool as a scratchpad to reason before calling external tools. Prevents impulsive wrong
tool calls on ambiguous inputs.

**What breaks it:**
- Think tool used for every single decision, including trivial ones → unnecessary latency
- Extended Thinking enabled on the model simultaneously → agent fails to determine which item to use (known n8n bug, early 2026)
- No Reasoning Protocol in system prompt → agent ignores the think tool entirely

**Prompt fixes:**
- Add Reasoning Protocol section: when to use think, what to think about
- Specify: "Do NOT use the think tool for simple, single-condition decisions"
- Disable Extended Thinking at the model level when Think Tool is connected

**Template:** `ReAct-Bounded` + Reasoning Protocol section

---

## 14. Agent with HITL Gates (Human-in-the-Loop Tools)

**What it does:** AI Agent with specific tools configured to require human approval before
execution (n8n native HITL, released Jan 2026). Workflow pauses; human approves or denies;
agent continues based on decision.

**What breaks it:**
- System prompt doesn't mention HITL → agent doesn't know which tools have gates or what to do on denial
- Agent retries denied tool call → violates the reviewer's decision; creates confusion
- Agent attempts alternative tool to achieve the same denied action → circumvents the gate
- Agent asks user for clarification instead of returning denial contract

**Prompt fixes:**
- Mandatory HITL Gate Declaration section listing gated tools
- Explicit denial behavior: return denial contract, do not retry, do not use alternatives
- Inform the user what happened without asking follow-up questions

**Template:** `ReAct-Bounded` + HITL Gate Declaration section

**Note:** HITL is not a prompt safeguard — it's a workflow architecture decision. If a step
MUST require human approval, add a HITL gate. Don't rely on "ask the user for confirmation"
in the system prompt — agents sometimes skip it under context pressure.
