# Introduction to Tool Use — How Claude Calls External Functions

> **Exam:** Claude Architect Foundation
> **Chapter:** 03 — Tool Use & MCP
> **Difficulty:** Beginner → Intermediate
> **Last Updated:** 2026-06

---

## 📖 What Is This Topic About?

Claude can do more than generate text — it can call external functions. **Tool use** is the mechanism that enables Claude to interact with APIs, databases, file systems, and any other external service. It is the foundation of all agentic applications and the most heavily tested topic on this exam.

---

## 🧠 Key Concepts

- **Tool** — A function that the application exposes to Claude; Claude can request its execution with specific parameters
- **Tool Definition** — A JSON description of a tool: its name, what it does, and what parameters it accepts
- **Tool Call** — When Claude decides to use a tool, it generates a `tool_use` block containing the tool name and parameters
- **Tool Result** — The output the application sends back to Claude after executing the tool
- **Tool Use Loop** — The cycle of: Claude calls tool → app executes → result returned to Claude → Claude continues
- **Structured Output** — Guaranteed JSON output from Claude when using tool use, rather than free-form text

---

## 📘 Detailed Explanation

### Why Tool Use Exists

Without tools, Claude can only generate text. It cannot:
- Look up live data (stock prices, weather, order status)
- Write to a database or file
- Call an API
- Execute code

Tool use bridges this gap. The application defines what tools are available, and Claude decides when and how to use them.

---

### How Tool Use Works — The Full Cycle

```
╔══════════════════════════════════════════════════════════════════════╗
║                     TOOL USE CYCLE                                   ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  1. APPLICATION DEFINES TOOLS                                        ║
║  ┌──────────────────────────────────────────────────────────────┐   ║
║  │  {                                                           │   ║
║  │    "name": "get_order_status",                               │   ║
║  │    "description": "Retrieves the current status of an        │   ║
║  │     order given its ID. Returns status, tracking number,     │   ║
║  │     and estimated delivery date.",                           │   ║
║  │    "input_schema": {                                         │   ║
║  │      "type": "object",                                       │   ║
║  │      "properties": {                                         │   ║
║  │        "order_id": {                                         │   ║
║  │          "type": "string",                                   │   ║
║  │          "description": "The order ID, e.g. ORD-4921"        │   ║
║  │        }                                                     │   ║
║  │      },                                                      │   ║
║  │      "required": ["order_id"]                                │   ║
║  │    }                                                         │   ║
║  │  }                                                           │   ║
║  └──────────────────────────────────────────────────────────────┘   ║
║                           │                                          ║
║                           │ Sent with every API request              ║
║                           ▼                                          ║
║  2. CLAUDE DECIDES TO USE A TOOL                                     ║
║  ┌──────────────────────────────────────────────────────────────┐   ║
║  │  Claude generates a tool_use block:                          │   ║
║  │  {                                                           │   ║
║  │    "type": "tool_use",                                       │   ║
║  │    "id": "toolu_abc123",                                     │   ║
║  │    "name": "get_order_status",                               │   ║
║  │    "input": { "order_id": "ORD-4921" }                       │   ║
║  │  }                                                           │   ║
║  └──────────────────────────────────────────────────────────────┘   ║
║                           │                                          ║
║                           │ Application receives this                ║
║                           ▼                                          ║
║  3. APPLICATION EXECUTES THE TOOL                                    ║
║  ┌──────────────────────────────────────────────────────────────┐   ║
║  │  Application code:                                           │   ║
║  │  def get_order_status(order_id):                             │   ║
║  │      return db.query("SELECT * FROM orders WHERE id=?",      │   ║
║  │                       order_id)                              │   ║
║  └──────────────────────────────────────────────────────────────┘   ║
║                           │                                          ║
║                           ▼                                          ║
║  4. APPLICATION SENDS TOOL RESULT BACK TO CLAUDE                     ║
║  ┌──────────────────────────────────────────────────────────────┐   ║
║  │  {                                                           │   ║
║  │    "type": "tool_result",                                    │   ║
║  │    "tool_use_id": "toolu_abc123",                            │   ║
║  │    "content": "Order ORD-4921: Shipped. Tracking: TR7892.    │   ║
║  │                Estimated delivery: 18 June 2026."            │   ║
║  │  }                                                           │   ║
║  └──────────────────────────────────────────────────────────────┘   ║
║                           │                                          ║
║                           ▼                                          ║
║  5. CLAUDE CONTINUES WITH THE RESULT IN CONTEXT                      ║
║  Claude reads the tool result and generates its next response        ║
║  (either another tool call OR the final answer)                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

### Tool Use vs Asking Claude to Return JSON

A common question is: why use tool use instead of just asking Claude to "return a JSON object"?

| Approach | Reliability | Guaranteed Valid JSON? | Schema Enforced? |
|---|---|---|---|
| "Please return JSON in your response" | Medium | ❌ No | ❌ No |
| Tool use with defined schema | High | ✅ Yes | ✅ Yes |

When you define a tool with a JSON schema, Claude must fill in the schema's fields — it cannot return free-form text or malformed JSON. This is why **tool use is the recommended approach for structured data extraction**.

---

### What the Application Is Responsible For

Claude does NOT execute the tool itself. Claude only requests execution. The application is responsible for:

```
┌──────────────────────────────────────────────────────────┐
│         APPLICATION'S TOOL EXECUTION RESPONSIBILITIES     │
│                                                          │
│  ✅ Actually running the function (calling the API)      │
│  ✅ Handling authentication (API keys, tokens)           │
│  ✅ Validating Claude's parameters before execution      │
│  ✅ Handling errors from the external service            │
│  ✅ Formatting the result and returning it to Claude     │
│  ✅ Enforcing business rules (e.g., max refund amount)   │
└──────────────────────────────────────────────────────────┘
```

---

## 🇮🇳 Indian Real-Life Example

**Think of tool use like a bank officer with access to multiple counters:**

When you visit a bank and ask the officer a question about your account:
1. The officer (Claude) decides they need to check your balance — they press a button on their terminal (tool call)
2. The bank's core banking system (the tool execution) looks up your account
3. The system returns your balance to the officer's screen (tool result)
4. The officer (Claude) reads the balance and answers your question

The officer cannot access the core banking system themselves — they can only request data and act on it. That is exactly how Claude's tool use works: Claude requests, the application executes.

---

## 🔑 Exam-Focused Points

- ✅ Tool use = Claude *requests* execution; the **application actually runs the function**
- ✅ Tool definitions consume tokens from the context window — complex schemas with many fields reduce document space
- ✅ Tool use guarantees **schema-compliant JSON output** — more reliable than asking Claude to "return JSON in text"
- ✅ Each tool definition includes: name, description, and input_schema (JSON Schema format)
- ✅ Claude matches tool to need based primarily on the **tool description** — write clear, specific descriptions
- ✅ The tool result is added to the conversation context; Claude continues from there

---

## 🧩 Scenario-Based Thinking

**Scenario:** A developer's pipeline occasionally receives responses that don't parse as valid JSON, causing downstream failures. The current implementation instructs Claude via prompt to "always return your answer as valid JSON."

**What is the most reliable solution?**

- A) Add stricter prompt instructions and more examples
- B) Post-process Claude's response and attempt to fix malformed JSON
- C) Switch to using tool use with a defined JSON schema — this guarantees schema-compliant output
- D) Use a lower temperature setting to reduce output variability

**Answer:** C — Tool use with a defined schema is the most reliable approach. Claude cannot return free-form text when tool use is active — it must fill in the defined schema. Prompt instructions (A) and post-processing (B) are workarounds that still fail unpredictably.

---

## 💡 Memory Tricks

**Claude = Detective, Tools = Witnesses:** Claude (the detective) cannot go out and gather evidence directly. It can only call witnesses (tools) and ask them questions. The witnesses report back, and the detective reasons about what to do next.

**Tool Definition = Job Posting:** Just as a job posting describes what the role does, who is eligible (parameters), and what's required, a tool definition describes what the function does, what parameters it accepts, and which are required.

---

## ❓ Chapter Practice Questions

**Q1.** A developer defines three tools for an e-commerce agent: `lookup_product`, `check_inventory`, and `process_order`. After testing, they find Claude frequently calls `check_inventory` when the user is asking about a product description. What is the most likely root cause?

- A) The tool input schemas are incorrectly defined
- B) `check_inventory` has a description that overlaps with product-related queries, causing Claude to select it inappropriately
- C) The developer needs to reduce the number of tools available
- D) Claude cannot distinguish between product lookup and inventory queries

**Answer:** B

**Explanation:** Claude selects tools based primarily on their descriptions. If `check_inventory`'s description is vague (e.g., "gets product information") rather than specific (e.g., "checks current stock quantity for a given product SKU"), Claude may select it for product description queries. The fix is to improve tool descriptions to be clear and distinct.

---

**Q2.** Why is tool use the recommended approach for extracting structured data from documents, compared to instructing Claude to "return your answer as a JSON object"?

- A) Tool use is faster and reduces latency
- B) Tool use with a defined schema guarantees schema-compliant JSON; free-form JSON instructions can produce malformed or inconsistent output
- C) Tool use prevents Claude from hallucinating field values
- D) Tool use allows Claude to execute the extraction in parallel

**Answer:** B

**Explanation:** When tool use is active with a defined JSON schema, Claude must conform to that schema — it cannot return free-form text or skip required fields. Prompt instructions to "return JSON" are unreliable because Claude can still produce text surrounding the JSON or malformed JSON. Tool use provides structural guarantees.

---

**Q3.** An extraction pipeline processes 10,000 documents using tool use. The tool definition has 15 fields with detailed descriptions, totalling approximately 3,000 tokens. Large documents (180,000+ tokens) show significantly lower accuracy. What is the most likely explanation?

- A) The tool definition prevents Claude from reading documents larger than 150,000 tokens
- B) Tool definitions consume input context tokens; combined with system prompts and large document content, the total approaches the 200K context limit, degrading accuracy for content in later document sections
- C) Claude processes the tool definition after the document, so large documents push the tool definition out of context
- D) 15-field schemas exceed Claude's maximum supported tool complexity

**Answer:** B

**Explanation:** Tool definitions count toward the context window. A 3,000-token tool definition + system prompt + 180,000-token document = very close to the 200K limit. As the context fills, Claude's effective attention to document content — especially toward the end — degrades. Solution: reduce tool schema verbosity or chunk large documents.

---

## 📌 Quick Revision Summary

- Tool use = Claude *requests* execution; application *executes* and returns result
- Tool definitions consume context window tokens — complex schemas reduce document space
- Tool use guarantees schema-compliant JSON output; prompt instructions do not
- Claude selects tools based on their descriptions — write clear, specific, distinct descriptions
- Tool result is added to conversation context; Claude continues the agentic loop from there
- Application is responsible for: executing the function, authentication, error handling, business rules

---

## 📎 References

- [Anthropic Documentation — Tool Use](https://docs.anthropic.com/en/docs/tool-use)
- [Anthropic Documentation — Tool Use Examples](https://docs.anthropic.com/en/docs/tool-use-examples)

---

*Notes by certification-study-hub. Chapter 03 — Introduction to Tool Use.*
