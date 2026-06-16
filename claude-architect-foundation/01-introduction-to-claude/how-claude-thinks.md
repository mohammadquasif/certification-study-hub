# How Claude Thinks — Context, Memory & The Request Loop

> **Exam:** Claude Architect Foundation
> **Chapter:** 01 — Introduction to Claude
> **Difficulty:** Beginner
> **Last Updated:** 2026-06

---

## 📖 What Is This Topic About?

To build well with Claude, you need to understand *how* it processes information — what it can see, what it remembers, and how it decides what to do next. This chapter covers the request-response loop, the context window, the different types of memory available to Claude, and what "stateless" means for developers.

---

## 🧠 Key Concepts

- **Context Window** — The total amount of text Claude can process in one request (input + output combined)
- **Tokens** — The unit used to measure context window size; roughly 1 token ≈ 0.75 English words
- **Stateless** — Claude does not automatically remember previous conversations; each API call starts fresh unless history is explicitly included
- **In-Context Memory** — Information available because it is in the current conversation window
- **External Memory** — Information stored outside Claude (databases, files) and retrieved via tools
- **System Prompt** — Instructions sent to Claude at the start of every conversation; sets behaviour, persona, and constraints

---

## 📘 Detailed Explanation

### The Request-Response Loop

Every interaction with Claude follows the same loop:

```
┌─────────────────────────────────────────────────────────┐
│                   APPLICATION / USER                    │
│                                                         │
│  ┌───────────────────────────────────────────────┐     │
│  │  1. BUILD REQUEST                             │     │
│  │     • System prompt (instructions)            │     │
│  │     • Conversation history (all prior turns)  │     │
│  │     • Current user message                    │     │
│  │     • Tool definitions (if using tools)       │     │
│  └───────────────┬───────────────────────────────┘     │
│                  │  API Call                            │
│  ┌───────────────▼───────────────────────────────┐     │
│  │  2. CLAUDE PROCESSES (reads everything above) │     │
│  │     • Understands context                     │     │
│  │     • Reasons about the best response         │     │
│  │     • Generates a reply (or tool call)        │     │
│  └───────────────┬───────────────────────────────┘     │
│                  │  Response                            │
│  ┌───────────────▼───────────────────────────────┐     │
│  │  3. APPLICATION RECEIVES RESPONSE             │     │
│  │     • Displays to user, OR                    │     │
│  │     • Executes tool call, OR                  │     │
│  │     • Stores in DB and loops back to Step 1   │     │
│  └───────────────────────────────────────────────┘     │
└─────────────────────────────────────────────────────────┘
```

> **Key insight for the exam:** Claude does NOT automatically remember previous messages. Your application must send the full conversation history with every API call.

---

### The Context Window — What Counts as "Space"?

The context window is like a fixed-size whiteboard. Everything on the whiteboard counts toward the limit:

```
┌──────────────────── CONTEXT WINDOW (200K tokens) ──────────────────┐
│                                                                     │
│  ┌─────────────────┐  ┌──────────────────┐  ┌──────────────────┐  │
│  │  System Prompt  │  │   Tool           │  │  Conversation    │  │
│  │  (instructions, │  │   Definitions    │  │  History         │  │
│  │   persona,      │  │   (JSON schema   │  │  (all prior      │  │
│  │   constraints)  │  │   of each tool)  │  │  turns)          │  │
│  │   ~500–2000     │  │   ~200–3000      │  │   varies         │  │
│  │   tokens        │  │   tokens         │  │                  │  │
│  └─────────────────┘  └──────────────────┘  └──────────────────┘  │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │  Documents / Pasted Content (can be very large)              │  │
│  │  e.g., a 190,000-token contract + 2,500 token tool schema    │  │
│  │  = very close to the 200K limit → performance degrades      │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │  Claude's Response (output also counts toward total)         │  │
│  └──────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

**Why this matters:** If your document is 190,000 tokens and your system prompt + tool definitions use 5,000 tokens, you are at 195,000/200,000 — leaving only 5,000 tokens for Claude's response. This can cause accuracy problems, especially for content toward the end of the document.

---

### Memory Types

Claude can "remember" things in four different ways, depending on how the system is designed:

| Memory Type | Where it Lives | Who Manages It | Lifespan | Example |
|---|---|---|---|---|
| **In-Context** | Inside the context window | Application sends it | Current request only | Previous messages in a chat |
| **External (retrieved)** | Database, files, vector store | Application + tools | Persistent | Customer history in a CRM |
| **In-Cache** (prompt caching) | Anthropic's servers | Anthropic API feature | Minutes to hours | Reusing a large system prompt |
| **In-Weights** | Claude's model parameters | Anthropic (at training) | Permanent | General knowledge, language |

---

### What "Stateless" Means for Developers

Claude is **stateless** by default. This means:

```
Conversation Turn 1:
  You: "My name is Priya."
  Claude: "Hello Priya, how can I help you?"

Conversation Turn 2 — WITHOUT sending Turn 1 history:
  You: "What is my name?"
  Claude: "I'm sorry, I don't have access to that information."
               ↑ Claude genuinely has no memory of Turn 1
```

```
Conversation Turn 2 — WITH Turn 1 history sent again:
  You: "What is my name?"
  Claude: "Your name is Priya."
               ↑ Claude can "remember" because history was included
```

**Your application is responsible for:
1. Storing the conversation history
2. Including it in every subsequent API call
3. Deciding when to summarise or truncate to manage context window limits**

---

## 🇮🇳 Indian Real-Life Example

**Think of Claude like a highly skilled consultant who has no memory between meetings:**

Imagine you hire a top consultant. Each meeting, they are brilliant — but they have a unique rule: **they forget everything after the meeting ends**. Every new meeting, you must bring the notes from previous meetings yourself, and hand them over at the start.

This is how Claude works:
- No memory between API calls
- Your system must store and re-send conversation history
- The "notes you bring" = conversation history in your API request
- The consultant's desk has limited space = context window

---

## 🔑 Exam-Focused Points

- ✅ Claude is **stateless** — it does not retain memory between API calls unless the application explicitly sends previous history
- ✅ The context window includes: system prompt + tool definitions + conversation history + documents + Claude's response
- ✅ **Tool definitions consume tokens** — complex tool schemas with many fields reduce the space available for documents
- ✅ As a conversation approaches the context limit, **accuracy on late-appearing content degrades**
- ✅ Four memory types: in-context, external (retrieved via tools), cached, in-weights (training knowledge)
- ✅ Developers must manage context by **summarising earlier turns** or using **external storage** for long conversations

---

## 🧩 Scenario-Based Thinking

**Scenario:** A support agent built on Claude handles long customer conversations. After 30+ turns, the agent starts forgetting things the customer said in the first 10 turns — even though those turns are still in the conversation history.

**What is the most likely cause?**

1. The combined conversation history has grown very large
2. Claude is selective about which parts of history it uses
3. The context window is approaching its limit, so earlier content gets less attention
4. The system prompt is overriding the conversation history

**Answer:** The context window is approaching its limit. As the total tokens approach 200K, Claude's ability to attend to content from early in the conversation degrades. The solution is to **summarise earlier turns** and store only the essential facts, rather than keeping the full raw history.

---

## 💡 Memory Tricks

**The Whiteboard Analogy:** The context window is a whiteboard with limited space. You, system prompts, tools, docs, and responses all share it. When it's full, you must erase some earlier notes to make room.

**Stateless = Each Phone Call Starts Fresh:** Imagine calling customer support. Each time you call, you get a fresh agent who knows nothing about your previous calls. You have to briefly re-explain your history each time. That's Claude without application-managed history.

---

## ❓ Chapter Practice Questions

**Q1.** A developer builds a chatbot using Claude. Users report that the bot "forgets" their name and preferences after a few messages. What is the most likely root cause?

- A) Claude's context window is too small to hold user preferences
- B) The conversation history is not being included in subsequent API requests
- C) The system prompt is overriding user-provided information
- D) Claude's in-weights memory was not trained on the user's data

**Answer:** B

**Explanation:** Claude is stateless — it does not remember previous API calls. The developer's application must store and re-send the conversation history with every API request. Without this, Claude starts each turn with no knowledge of prior messages.

---

**Q2.** A document extraction system processes 180,000-token contracts using a 12-field tool schema (approximately 2,500 tokens). Claude's context window is 200,000 tokens. Which statement best explains why extraction accuracy drops for content near the end of the contract?

- A) Tool schemas prevent Claude from reading document content
- B) Claude processes documents in chunks, missing later sections
- C) The combined size of the document, tool schema, and system prompt approaches the context limit, leaving little room for the model to process late-appearing content effectively
- D) Claude prioritises tool definitions over document content when context is tight

**Answer:** C

**Explanation:** All elements — system prompt, tool definitions, and document content — share the same context window. When the total approaches 200K, the model's attention is spread very thin, and content toward the end of the document receives less effective processing.

---

**Q3.** Which type of memory persists across sessions and can be retrieved on demand using tools, without being part of the context window?

- A) In-context memory
- B) In-weights memory
- C) Cached memory
- D) External memory

**Answer:** D

**Explanation:** External memory lives in databases, files, or vector stores outside Claude. The application retrieves relevant records using tool calls and injects them into the context window when needed. This is the standard approach for persistent, long-term memory in Claude-based systems.

---

## 📌 Quick Revision Summary

- Claude is stateless — your application must send conversation history with every API call
- Context window = up to 200K tokens — shared by system prompt, tool definitions, history, documents, and response
- Tool definitions consume tokens; large schemas reduce space available for documents
- Four memory types: in-context, external (retrieved), cached, in-weights (training)
- Approaching context limits → accuracy degrades for content at the end of long documents
- Solution for long conversations: summarise early turns; use external storage for persistent facts

---

## 📎 References

- [Anthropic Documentation — Context Windows](https://docs.anthropic.com)
- [Claude API Reference — Messages](https://docs.anthropic.com/en/api/messages)

---

*Notes by certification-study-hub. Chapter 01 — How Claude Thinks.*
