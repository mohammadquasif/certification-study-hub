# 🤖 Claude Architect Foundation — Study Hub

Welcome to the Claude Architect Foundation section of certification-study-hub!

This section covers everything you need to clear the **Claude Architect Foundation** certificate offered by Anthropic.

---

## 📂 Folder Structure

```
claude-architect-foundation/
├── README.md                              ← You are here
├── 01-introduction-to-claude/
│   ├── what-is-claude.md
│   ├── how-claude-thinks.md
│   └── constitutional-ai-and-safety.md
├── 02-agentic-patterns/
│   ├── what-is-an-agent.md
│   ├── orchestrator-and-workers.md
│   ├── prompt-chaining-and-routing.md
│   └── session-and-context-management.md
├── 03-tool-use-and-mcp/
│   ├── introduction-to-tool-use.md
│   ├── designing-good-tools.md
│   ├── error-handling-in-tools.md
│   ├── mcp-architecture.md
│   └── confirmation-and-safety-gates.md
├── 04-structured-output-and-extraction/
│   ├── json-schema-for-extraction.md
│   ├── handling-ambiguity-and-nulls.md
│   ├── confidence-and-human-review.md
│   ├── batch-processing.md
│   └── long-document-extraction.md
├── 05-claude-code-and-workflows/
│   ├── introduction-to-claude-code.md
│   ├── claude-md-and-skills.md
│   ├── session-management.md
│   ├── code-review-workflows.md
│   └── exploration-patterns.md
├── practice-questions/
│   └── chapter-wise-questions.md
├── mock-tests/
│   ├── caf-mock-test-60q.html            ← 🔥 Interactive 60-Question Test
│   └── mock-test-01.md
└── revision-notes/
    └── quick-revision.md
```

---

## 📋 Exam Overview

| Item | Detail |
|---|---|
| Certificate | Claude Architect Foundation |
| Provider | Anthropic |
| Format | Online assessment |
| Number of Questions | ~60 questions |
| Duration | ~60 minutes |
| Focus | Responsible AI, Claude architecture, tool use, agentic patterns, structured output |
| Level | Beginner / Foundation |

> Check the official Anthropic website for the latest exam details, as this information may change.

---

## 🗂️ Exam Domain Breakdown

| Domain | Estimated Weight | What It Tests |
|---|---|---|
| Claude Fundamentals & Architecture | ~15% | What Claude is, how the agentic loop works, context windows |
| Agentic Patterns & Multi-Agent Systems | ~25% | Orchestrators, workers, prompt chaining, routing, session management |
| Tool Use & MCP Design | ~25% | Tool definitions, error handling, MCP architecture, safety gates |
| Structured Output & Extraction | ~20% | JSON schemas, ambiguity, batch processing, human review |
| Claude Code & Developer Workflows | ~15% | CLAUDE.md, Skills, session resumption, code review patterns |

**Tool Use and Agentic Patterns together = ~50% of the exam. Focus here first.**

---

## ✅ Recommended Chapter Order

Follow the chapters in this order for best results:

1. `01-introduction-to-claude/` — Understand what Claude is and how it works
2. `02-agentic-patterns/` — Learn how agents think, plan, and act
3. `03-tool-use-and-mcp/` — Master tool use and MCP design — tested heavily
4. `04-structured-output-and-extraction/` — JSON schemas, extraction patterns, batch processing
5. `05-claude-code-and-workflows/` — Practical developer workflows with Claude Code

Then: **Practice Questions → Revision Notes → Mock Test**

---

## 🔥 Interactive Mock Test

Take the full 60-question mock test with a timer, auto-scoring, and detailed results:

👉 **[Open the Mock Test](./mock-tests/caf-mock-test-60q.html)** — Download and open in your browser.

---

## 💡 Top Exam Tips

- **The Agentic Loop** — Know the observe → think → act → observe cycle deeply
- **Tool Error Handling** — `isError: true` + structured error metadata (errorCategory, isRetryable) is the pattern
- **MCP vs Tool Use** — MCP is a protocol for connecting Claude to external servers; tool use is how Claude calls any tool
- **Batches API vs Real-Time** — Batch = cost savings + up to 24h window; Real-time = immediate response
- **CLAUDE.md vs Skills** — CLAUDE.md = always-on instructions; Skills = activated on demand for specific tasks
- **Null vs Fabrication** — When source data is missing, return null. Never fabricate field values.
- **Session Forking** — Use `fork_session` to explore two approaches from the same starting point
- **Independent Review** — A fresh agent reviewing code avoids confirmation bias from the generator

---

Start with `01-introduction-to-claude/what-is-claude.md` 👇
