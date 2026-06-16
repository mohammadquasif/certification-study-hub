# Session & Context Management — Memory Across Long Conversations

> **Exam:** Claude Architect Foundation
> **Chapter:** 02 — Agentic Patterns
> **Difficulty:** Intermediate
> **Last Updated:** 2026-06

---

## 📖 What Is This Topic About?

As conversations grow long — spanning many tool calls, many turns, and sometimes hours of work — two challenges emerge: the context window fills up, and the conversation becomes expensive to process. This chapter covers strategies for managing context over long sessions: session resumption, context summarisation, forking sessions for parallel exploration, and external memory.

---

## 🧠 Key Concepts

- **Session** — A single conversation thread with Claude, containing all turns from start to current
- **Session Resumption** — Continuing a previous session by loading its transcript back into a new API call
- **Context Pressure** — The state where a long conversation has consumed most of the context window, reducing performance
- **Context Summarisation** — Compressing earlier turns into a shorter summary that preserves key facts without keeping the full raw text
- **Session Forking** — Creating two independent branches from the same conversation checkpoint, for exploring alternative approaches
- **External Memory** — Storing structured facts (entity IDs, statuses, decisions) outside the context window and retrieving them via tools

---

## 📘 Detailed Explanation

### The Context Window Filling Problem

Every message, tool call, and tool result adds tokens to the context window. Over a long investigation or multi-issue support session, the window fills:

```
┌──────────────────────────────────── CONTEXT WINDOW ────────────────────────────────┐
│                                                                                     │
│  Turn 1–10   Turn 11–20   Turn 21–30   Turn 31–40   Turn 41–50   ???               │
│  [▓▓▓▓▓▓▓]  [▓▓▓▓▓▓▓▓]  [▓▓▓▓▓▓▓▓]  [▓▓▓▓▓▓▓▓]  [▓▓▓▓▓▓▓▓]  [NO ROOM!]         │
│                                                                                     │
│  As the window fills:                                                               │
│  • Early turns get less "attention" from the model                                  │
│  • Accuracy on information from early turns degrades                                │
│  • Eventually: context_length_exceeded error                                        │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

### Strategy 1: Context Summarisation

Instead of keeping every raw turn, compress earlier turns into a concise summary and discard the verbose originals:

```
BEFORE SUMMARISATION (raw, verbose):
Turn 1: "Hi I need help with my order"
Turn 2: "Sure, what's your order ID?" 
Turn 3: "It's ORD-4921"
Turn 4: [tool call: lookup_order("ORD-4921")]
Turn 5: [tool result: {id: "ORD-4921", item: "Laptop", price: 85000, status: "shipped", tracking: "TR7892"}]
Turn 6: "Your laptop was shipped 3 days ago, tracking number TR7892"
... (6 turns = ~800 tokens)

AFTER SUMMARISATION (compressed):
"CONTEXT SUMMARY: Customer's order ORD-4921 (Laptop, ₹85,000) was shipped. Tracking: TR7892. 
Customer is now asking about delivery date."
(1 turn = ~60 tokens — 93% reduction)
```

**Key rule:** Extract and preserve *structured facts* (IDs, amounts, statuses) precisely. Do not summarise these into vague natural language — you may lose precision.

---

### Strategy 2: External Persistent Memory

For long-running workflows or returning customers, the best approach is to persist structured data outside the context window entirely — in a database, file, or structured store:

```
┌────────────────────────────────────────────────────────────────────┐
│                    EXTERNAL MEMORY PATTERN                         │
│                                                                    │
│  AGENT                         EXTERNAL STORE (DB/file)           │
│   │                                   │                           │
│   │  extract_key_facts()               │                           │
│   │ ─────────────────────────────────►│                           │
│   │                                   │ Stores:                   │
│   │                                   │ {customer_id, order_ids,  │
│   │                                   │  open_issues, decisions}  │
│   │                                   │                           │
│   │ ◄─────────── stored ──────────────│                           │
│   │                                   │                           │
│   [new session starts]                │                           │
│   │                                   │                           │
│   │  retrieve_context(customer_id)    │                           │
│   │ ─────────────────────────────────►│                           │
│   │ ◄───────── structured facts ──────│                           │
│   │                                   │                           │
│   [agent continues with rich context] │                           │
└────────────────────────────────────────────────────────────────────┘
```

---

### Strategy 3: Session Forking

Sometimes you want to explore two different approaches to the same problem, without letting one approach's reasoning contaminate the other. **Forking** creates two independent branches from the same checkpoint:

```
┌──────────────────────────────────────────────────────────────┐
│                     SESSION FORKING                          │
│                                                              │
│  ORIGINAL SESSION (30 turns of analysis)                     │
│  "We've identified two approaches to refactor this module:   │
│   (A) Extract as microservice  (B) Refactor in place"        │
│                    │                                         │
│          ┌─────────┴─────────┐                               │
│          │  FORK HERE         │                               │
│          ▼                   ▼                               │
│  BRANCH A                BRANCH B                            │
│  Explore microservice    Explore in-place                    │
│  extraction approach     refactoring approach                │
│  (independent context)   (independent context)               │
│          │                   │                               │
│          ▼                   ▼                               │
│  Developer compares results and chooses one approach         │
└──────────────────────────────────────────────────────────────┘
```

**Why forking beats sequential exploration:**
- Branch A's reasoning does not contaminate Branch B's
- Both explorations start from the same knowledge base
- Truly independent, comparable results

---

### Session Resumption

When a session is interrupted (connection drops, end of workday, system restart), you can resume it:

- **By name** — If the session was given a name (`--resume session-name`), resume by that name
- **By ID** — If you have the session UUID from the transcript, use `--session-id UUID`
- **With a summary** — Inject a summary of prior findings as context for a fresh session

**When resuming, always inform the agent of any changes** that happened while the session was paused (e.g., files changed, new commits merged). The agent's knowledge is a snapshot from when it was last active.

---

### Sliding Window Context (Anti-Pattern Warning)

Sliding window context keeps only the most recent N turns. While simple to implement, it has a dangerous flaw:

```
PROBLEM WITH SLIDING WINDOW:
Turn 1:  Customer gives order ID: ORD-4921
Turn 2:  Agent looks up order details
...
Turn 35: Window slides, Turn 1 and Turn 2 are DROPPED
Turn 36: Agent tries to reference order details → no longer in context → error
```

Sliding windows can silently lose critical information. **Prefer summarisation + external storage** over sliding windows.

---

## 🇮🇳 Indian Real-Life Example

**Context Management = A Court Case File:**

Imagine a lawyer handling a complex case that spans months of hearings. The lawyer cannot memorise every word of every hearing. Instead:
- After each hearing, they summarise key decisions in a case file (context summarisation)
- The original hearing transcripts go to the archive (external memory)
- For each new hearing, the lawyer reads the case file summary (retrieving external memory)
- If two lawyers want to argue alternative legal strategies, they work from separate copies of the case file (session forking)

---

## 🔑 Exam-Focused Points

- ✅ Context window fills over long sessions → accuracy degrades → eventually `context_length_exceeded`
- ✅ **Summarisation** = compress early turns into concise facts; preserve structured data (IDs, amounts) precisely
- ✅ **External memory** = persist structured facts in a DB or file; retrieve via tools in new sessions
- ✅ **Session forking** = creates two independent branches from the same checkpoint; prevents cross-contamination between alternative approaches
- ✅ When resuming a paused session, **inform the agent of any changes** that occurred during the pause
- ✅ Sliding window = simple but risky; silently drops potentially critical earlier information

---

## 🧩 Scenario-Based Thinking

**Scenario:** A developer was investigating a complex payment module for 2 hours, building significant context. The connection dropped overnight. A teammate merged a PR that renamed two utility functions. The developer wants to continue the same investigation.

**What is the most effective approach?**

- A) Start a completely fresh session
- B) Resume the paused session and inform the agent about the renamed functions
- C) Resume the session without mentioning the changes — architecture knowledge is still valid
- D) Start a fresh session with a manually written summary of findings

**Answer:** B — Resume the session (preserving 2 hours of accumulated context) AND inform the agent about the renamed functions. The architecture knowledge is valid; only the specific function names changed. Silently resuming without mentioning the change (C) risks the agent making incorrect references to the old function names.

---

## 💡 Memory Tricks

**Context Window = Laptop RAM:** If you open too many tabs, performance suffers. Close (summarise) older tabs to free up RAM. Important data goes to the hard drive (external storage).

**Forking = Git Branch:** Just like `git checkout -b new-feature` creates an independent branch from the current commit, session forking creates an independent conversation branch from the current turn.

---

## ❓ Chapter Practice Questions

**Q1.** A customer support agent handles a long conversation covering three separate issues across 47 turns. Approaching turn 48, the agent is asked to recap the first issue. What is the most effective context management strategy to ensure accuracy on earlier information?

- A) Use a sliding window that always keeps the last 30 turns
- B) Rely on the model to recall earlier turns from its in-weights memory
- C) Extract and persist structured issue data (issue type, order IDs, resolution status) into a separate context layer, retrievable on demand
- D) Restart the conversation from turn 30 to reduce context size

**Answer:** C

**Explanation:** Structured external memory is more reliable than relying on the raw context or sliding windows. Extracting key facts (issue type, IDs, statuses) into a separate structured store and retrieving them when needed is the recommended approach for long, multi-issue conversations. Sliding windows risk silently dropping critical earlier information.

---

**Q2.** An engineer spent 30 minutes analysing a legacy authentication module, identifying two possible refactoring approaches. They want to have the agent explore specific code changes for each approach before choosing one to implement.

**What is the most effective way to structure this exploration?**

- A) Start two fresh sessions, manually providing a summary of findings to each
- B) Continue in the original session, exploring Approach A then Approach B sequentially
- C) Resume the session and explore Approach A, then start a new session for Approach B
- D) Use session forking to create two independent branches from the current analysis checkpoint

**Answer:** D

**Explanation:** Session forking creates two branches from the same knowledge base, allowing independent exploration of each approach. This prevents Approach A's code-level reasoning from contaminating Approach B's, and both branches start with the full 30-minute analysis context. Sequential exploration (B) allows contamination; fresh sessions (A) lose the accumulated context.

---

**Q3.** A team stores an entire 400-turn conversation history and sends it with every API call to ensure the agent has full context. What is the primary risk of this approach?

- A) Sending historical turns violates Anthropic's usage policy
- B) Claude ignores turns older than 100 turns
- C) Token costs grow linearly with conversation length, and performance degrades as the context window fills
- D) External APIs cannot receive requests larger than 200K tokens

**Answer:** C

**Explanation:** Sending the full raw history with every API call has two compounding problems: rapidly increasing token costs (you pay for all input tokens every call) and degrading model performance as the context window fills. The recommended approach is to summarise earlier turns and persist structured facts in external storage, keeping the active context window lean.

---

## 📌 Quick Revision Summary

- Context window fills during long sessions → performance degrades → context_length_exceeded error
- Summarisation: compress early turns into brief structured summaries; preserve IDs/amounts precisely
- External memory: persist facts in DB/file; retrieve via tools in new or resumed sessions
- Session forking: two independent branches from same checkpoint; prevents cross-contamination
- Session resumption: by name or session ID; always notify agent of changes during the pause
- Sliding window: risky — silently drops critical earlier information; prefer summarisation

---

## 📎 References

- [Anthropic Documentation — Long Context Best Practices](https://docs.anthropic.com/en/docs/long-context)
- [Anthropic Documentation — Claude Code Session Management](https://docs.anthropic.com/en/docs/claude-code)

---

*Notes by certification-study-hub. Chapter 02 — Session & Context Management.*
