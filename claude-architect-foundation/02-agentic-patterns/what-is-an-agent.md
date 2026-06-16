# What Is an Agent? — The Agentic Loop

> **Exam:** Claude Architect Foundation
> **Chapter:** 02 — Agentic Patterns
> **Difficulty:** Beginner → Intermediate
> **Last Updated:** 2026-06

---

## 📖 What Is This Topic About?

A **Claude agent** is not just a chatbot that answers questions. It is a system where Claude can take actions in the real world — calling APIs, reading files, writing code, sending messages — in order to complete a goal over multiple steps. Understanding the agentic loop is fundamental to every other topic in this exam.

---

## 🧠 Key Concepts

- **Agent** — A Claude-powered system that uses tools to take actions and make decisions over multiple steps to achieve a goal
- **Agentic Loop** — The repeating cycle of Observe → Think → Act → Observe that an agent follows until a task is complete
- **Tool Use** — The mechanism by which Claude calls external functions (APIs, databases, file systems) during an agentic loop
- **Tool Call** — A structured request Claude sends to the application layer, asking it to execute a specific function with specific parameters
- **Tool Result** — The output the application sends back to Claude after executing a tool
- **Human-in-the-Loop** — A design pattern where human confirmation is required before certain agent actions are executed

---

## 📘 Detailed Explanation

### What Makes a System "Agentic"?

Not every Claude application is an agent. The distinction is:

| Type | What It Does | Example |
|---|---|---|
| **Single-turn Q&A** | User asks, Claude answers, done | "What is the capital of France?" |
| **Multi-turn Chat** | Conversation with memory across turns | Customer support chatbot |
| **Agentic System** | Claude takes actions, observes results, decides next step, repeats | Research assistant that searches, reads, and writes a report |

An agentic system is characterised by:
1. Access to **tools** (ability to take real-world actions)
2. **Multiple steps** — the goal requires a sequence of actions, not just one response
3. **Decision-making** — Claude decides which tool to call next based on what it observes

---

### The Agentic Loop — Step by Step

```
╔═══════════════════════════════════════════════════════════════╗
║                     THE AGENTIC LOOP                          ║
╠═══════════════════════════════════════════════════════════════╣
║                                                               ║
║  ┌──────────────────────────────────────────────────────┐    ║
║  │  USER / ORCHESTRATOR gives Claude a goal             │    ║
║  │  "Find all open invoices over ₹50,000 and send       │    ║
║  │   payment reminders to each client"                  │    ║
║  └─────────────────────┬────────────────────────────────┘    ║
║                         │                                     ║
║              ┌──────────▼──────────┐                         ║
║              │   1. OBSERVE        │                         ║
║              │   Claude reads the  │                         ║
║              │   current context   │                         ║
║              │   (goal + history   │                         ║
║              │   + tool results)   │                         ║
║              └──────────┬──────────┘                         ║
║                         │                                     ║
║              ┌──────────▼──────────┐                         ║
║              │   2. THINK          │                         ║
║              │   Claude reasons:   │                         ║
║              │   "What do I know?  │                         ║
║              │   What do I need?   │                         ║
║              │   What tool helps?" │                         ║
║              └──────────┬──────────┘                         ║
║                         │                                     ║
║              ┌──────────▼──────────┐                         ║
║              │   3. ACT            │                         ║
║              │   Claude calls a    │                         ║
║              │   tool OR generates │                         ║
║              │   final answer      │                         ║
║              └──────────┬──────────┘                         ║
║                         │  (if tool call)                    ║
║              ┌──────────▼──────────┐                         ║
║              │   4. RECEIVE RESULT │                         ║
║              │   Tool result added │                         ║
║              │   to context →      │                         ║
║              │   Loop back to      │                         ║
║              │   OBSERVE           │                         ║
║              └──────────┬──────────┘                         ║
║                         │  (if final answer)                 ║
║              ┌──────────▼──────────┐                         ║
║              │   5. COMPLETE       │                         ║
║              │   Claude delivers   │                         ║
║              │   the final result  │                         ║
║              └─────────────────────┘                         ║
╚═══════════════════════════════════════════════════════════════╝
```

---

### A Concrete Walk-Through

Goal: "Check the weather in Mumbai and send an alert to the team if it's raining."

| Loop Iteration | Claude Thinks | Claude Acts | Result |
|---|---|---|---|
| Turn 1 | "I need to check the weather. I have a `get_weather` tool." | Calls `get_weather(city="Mumbai")` | Returns: "Heavy rain, 28°C" |
| Turn 2 | "It is raining. I should send an alert. I have a `send_slack_message` tool." | Calls `send_slack_message(channel="#ops", message="Rain alert: Mumbai is experiencing heavy rain.")` | Returns: "Message sent successfully" |
| Turn 3 | "Both steps are complete. I can now confirm to the user." | Generates final response: "Done! Weather checked (heavy rain in Mumbai) and alert sent to #ops." | Loop ends |

---

### Human-in-the-Loop — When Agents Should Pause

Not all actions should proceed automatically. For irreversible or high-stakes actions, agents should **pause and ask for confirmation**:

```
┌────────────────────────────────────────────────────────┐
│           WHEN TO REQUIRE HUMAN CONFIRMATION           │
│                                                        │
│  ✅ Irreversible actions (deleting files, records)     │
│  ✅ Financial actions above a threshold (e.g., >₹5000) │
│  ✅ Sending communications to external parties         │
│  ✅ Actions on production systems                      │
│  ✅ When the agent is uncertain about the correct path │
│  ✅ When the user explicitly requests a human          │
└────────────────────────────────────────────────────────┘
```

---

### Escalation to a Human

A well-designed agent always has an **escalation path**. Claude should escalate when:
- The customer explicitly requests a human
- The issue requires a policy exception beyond the agent's authority
- The agent cannot make meaningful progress after reasonable attempts
- The situation involves risk (fraud, safety, legal)

When escalating, the agent should provide a **structured handoff** — not just the raw conversation, but a summary of: what the user needed, what actions were taken, what was discovered, and what the recommended next step is.

---

## 🇮🇳 Indian Real-Life Example

**Think of an agent like a junior CA (Chartered Accountant) doing your tax filing:**

1. **Observe:** Reads your Form 16, bank statements, and investment proofs
2. **Think:** "I need to check Section 80C deductions and compute tax liability"
3. **Act:** Searches the tax rules database, runs the calculation
4. **Observe Result:** Sees that you may owe ₹12,000 in taxes
5. **Think:** "This is above the threshold — I should confirm with the senior CA before filing"
6. **Act:** Escalates to the senior for confirmation (Human-in-the-Loop)

The junior CA does not just do one thing — they iterate, check, verify, and escalate when needed. That is exactly how a Claude agent works.

---

## 🔑 Exam-Focused Points

- ✅ The agentic loop is: **Observe → Think → Act → Observe** (repeating until goal is achieved)
- ✅ Claude decides which tool to call next **based on the tool results it observes** — not from a pre-planned sequence
- ✅ Tool results are added to the context window and Claude reasons about the next step
- ✅ **Human-in-the-loop** is required for irreversible, high-stakes, or uncertain situations
- ✅ Escalation should include a **structured handoff** — not just the transcript
- ✅ Escalate when: customer requests a human, policy exception required, agent cannot progress, safety risk

---

## 🧩 Scenario-Based Thinking

**Scenario:** An e-commerce agent calls `lookup_order` and receives the order details. The order was placed 45 days ago. The agent must decide whether to call `process_refund` or `escalate_to_human`.

**How does the agent make this decision?**

- A) A pre-configured decision tree maps order age to action
- B) The orchestration layer routes to the next tool automatically
- C) The order details are added to the context and Claude reasons about the next step based on policy guidelines in the system prompt
- D) Claude calls `process_refund` by default unless it fails

**Answer:** C — Claude reasons based on the information in the context window (order details + system prompt policy). This is the core of the agentic loop — Claude *thinks* about what to do next based on what it observes.

---

## 💡 Memory Tricks

**OTAO Loop:** Observe, Think, Act, Observe. **"Only Tigers Attract Attention"** — O, T, A, O.

**Agent vs Chatbot:** A chatbot *answers*. An agent *acts*. The agent keeps going until the job is done.

---

## ❓ Chapter Practice Questions

**Q1.** What is the key characteristic that distinguishes a Claude *agent* from a simple question-answering Claude application?

- A) Agents use a different version of the Claude model
- B) Agents have access to tools and take multiple actions over several steps to achieve a goal, reasoning about what to do next based on results
- C) Agents do not require a system prompt
- D) Agents store memory permanently in the model's weights

**Answer:** B

**Explanation:** An agent is characterised by: tools (to take real-world actions), multiple steps, and dynamic decision-making at each step based on observed results. Simple Q&A applications have none of these — they send a message and receive a text response.

---

**Q2.** In an agentic loop, when the model calls `get_customer_profile` and receives the result, what determines what the model does next?

- A) A hardcoded decision tree in the application layer
- B) The result is added to the context window, and Claude reasons about the next appropriate action
- C) The tool result contains a `next_action` field that Claude reads
- D) The model returns to the beginning of the loop and re-reads the original goal only

**Answer:** B

**Explanation:** Tool results are added to the conversation context. Claude then goes through the Observe → Think → Act cycle again, now with the new information. Claude decides the next action based on all the context — original goal, prior tool results, and system prompt guidance.

---

**Q3.** A billing agent has successfully verified that a customer is entitled to a refund of ₹8,500. The system's policy allows the agent to process refunds up to ₹5,000 autonomously. What should the agent do?

- A) Process the refund anyway, since the customer is clearly entitled
- B) Decline the refund because it exceeds the automated limit
- C) Ask the customer to call back during business hours
- D) Compile a structured handoff with refund details and escalate to a human agent

**Answer:** D

**Explanation:** The refund exceeds the agent's authorisation limit. The agent should not exceed its authority (even if the refund is legitimate) nor simply refuse or deflect. The correct design is to escalate with a structured handoff — customer ID, verification status, refund amount, policy reference — so the human agent can resolve it efficiently.

---

## 📌 Quick Revision Summary

- Agent = Claude + tools + multi-step reasoning toward a goal
- Agentic loop: Observe → Think → Act → Observe (repeats until complete)
- Tool results are added to context; Claude reasons about the next step each time
- Human-in-the-loop: required for irreversible, high-stakes, or uncertain actions
- Escalation = structured handoff (not just transcript) with diagnosis + recommended action
- Escalate when: user requests human, policy exception needed, agent cannot progress, safety risk

---

## 📎 References

- [Anthropic Documentation — Agents Overview](https://docs.anthropic.com/en/docs/agents)
- [Anthropic Documentation — Tool Use](https://docs.anthropic.com/en/docs/tool-use)

---

*Notes by certification-study-hub. Chapter 02 — What Is an Agent.*
