# Orchestrator & Workers — Multi-Agent Architecture

> **Exam:** Claude Architect Foundation
> **Chapter:** 02 — Agentic Patterns
> **Difficulty:** Intermediate
> **Last Updated:** 2026-06

---

## 📖 What Is This Topic About?

Not every task can be done by a single agent in a single context window. Some tasks are too large, too complex, or involve parallel workstreams that need to happen simultaneously. The **Orchestrator-Workers pattern** solves this by having one Claude instance act as a manager (orchestrator) that delegates subtasks to specialised Claude instances (workers / subagents).

---

## 🧠 Key Concepts

- **Orchestrator** — A Claude instance that breaks a high-level goal into subtasks, delegates them to workers, and synthesises the results
- **Worker / Subagent** — A Claude instance (or other model) that receives a specific, bounded subtask from the orchestrator and returns a focused result
- **Parallelisation** — Running multiple subagents simultaneously so independent subtasks complete at the same time
- **Task Decomposition** — Breaking a complex goal into smaller, independently solvable pieces
- **Context Isolation** — Each subagent has its own context window, so they don't pollute each other's reasoning

---

## 📘 Detailed Explanation

### Single Agent vs Orchestrator-Workers

| Dimension | Single Agent | Orchestrator-Workers |
|---|---|---|
| Context window | One shared context | Each subagent has its own isolated context |
| Complexity | Good for focused tasks | Good for broad, multi-domain tasks |
| Parallelisation | Sequential only | Multiple subagents can run simultaneously |
| Error isolation | One failure = whole task fails | One subagent failure doesn't kill the whole job |
| Overhead | Low | Higher (coordination cost) |

---

### The Orchestrator-Workers Architecture

```
╔═══════════════════════════════════════════════════════════════════╗
║               ORCHESTRATOR-WORKERS PATTERN                        ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                   ║
║    USER / APPLICATION                                             ║
║         │                                                         ║
║         ▼                                                         ║
║  ┌─────────────────────────────────────────────────────────┐     ║
║  │                    ORCHESTRATOR (Claude)                 │     ║
║  │  • Receives the high-level goal                         │     ║
║  │  • Analyses what subtasks are needed                    │     ║
║  │  • Delegates to appropriate workers                     │     ║
║  │  • Synthesises all worker results into final output     │     ║
║  └─────────────┬─────────────┬──────────────┬─────────────┘     ║
║                │             │              │                     ║
║          delegate      delegate       delegate                    ║
║                │             │              │                     ║
║         ┌──────▼──────┐ ┌───▼──────┐ ┌───▼──────────┐          ║
║         │  WORKER A   │ │ WORKER B │ │   WORKER C   │          ║
║         │  (Security  │ │ (Code    │ │ (Documentation│          ║
║         │   Review)   │ │  Style   │ │  Completeness)│          ║
║         └──────┬──────┘ └───┬──────┘ └───┬──────────┘          ║
║                │             │              │                     ║
║          return result  return result  return result              ║
║                │             │              │                     ║
║          ┌─────▼─────────────▼──────────────▼────────┐          ║
║          │     ORCHESTRATOR synthesises results        │          ║
║          │     → Final structured report              │          ║
║          └────────────────────────────────────────────┘          ║
╚═══════════════════════════════════════════════════════════════════╝
```

---

### When to Use Orchestrator-Workers

Use this pattern when:

1. **The task requires multiple independent analyses** — e.g., reviewing code for security, style, and documentation separately
2. **The task is too large for one context window** — large codebase analysis requires reading different sections in parallel
3. **You need parallel speed** — three independent analyses can run simultaneously instead of sequentially
4. **You want isolation** — each worker focuses on one thing without being distracted by other concerns

Do NOT use orchestrator-workers for:
- Simple, single-step tasks (adds unnecessary overhead)
- Tasks where worker results are so interdependent that parallelisation doesn't help
- When a simple prompt chain (sequential steps) would suffice

---

### Task Decomposition Patterns

The orchestrator can decompose tasks in different ways:

```
APPROACH 1: Parallel Decomposition (Independent Subtasks)
───────────────────────────────────────────────────────
Goal: Review a pull request
  ├── Worker A: Security review
  ├── Worker B: Code style review      ← All run simultaneously
  └── Worker C: Documentation review

APPROACH 2: Sequential Decomposition (Dependent Subtasks)
──────────────────────────────────────────────────────────
Goal: Generate a research report
  Step 1: Worker A — Search for sources
       ↓ (results passed to next)
  Step 2: Worker B — Summarise each source
       ↓
  Step 3: Worker C — Write the final report
  (This is more like Prompt Chaining — see next chapter)
```

---

### Context Isolation — A Key Benefit

When you use subagents, each has its own isolated context window. This means:

- Worker A's security analysis doesn't get confused by Worker B's style notes
- Workers don't see each other's outputs (only the orchestrator does)
- The orchestrator gets clean, focused outputs from each worker
- This also prevents "context contamination" — exploratory reasoning by one worker influencing another

---

## 🇮🇳 Indian Real-Life Example

**Think of the Orchestrator-Workers pattern like a Project Manager at a construction site:**

A large building project cannot be done by one person. The Project Manager (orchestrator):
- Assigns the foundation work to the civil engineering team (Worker A)
- Assigns electrical layout to electricians (Worker B) — running simultaneously
- Assigns plumbing to plumbers (Worker C) — running simultaneously
- Reviews all three teams' work and compiles the final progress report

Each team only knows about their own work. The PM synthesises everything.

---

## 🔑 Exam-Focused Points

- ✅ The **orchestrator** breaks down goals, delegates to workers, synthesises results
- ✅ **Workers** receive focused, bounded subtasks — they don't see the full picture
- ✅ Workers can run in **parallel** for independent subtasks (faster than sequential)
- ✅ Each worker has its **own isolated context window** — preventing cross-contamination
- ✅ Use orchestrator-workers for: multi-domain tasks, parallelisation, large-document tasks
- ✅ Do NOT use for: simple tasks (overhead not worth it), highly sequential dependent tasks

---

## 🧩 Scenario-Based Thinking

**Scenario:** A code review agent must analyse pull requests for three things: code style compliance, potential security vulnerabilities, and documentation completeness. Each review requires reading files, running analysis tools, and generating a report section. Every PR needs all three reviews.

**Which decomposition pattern is most appropriate?**

- A) Single comprehensive prompt — all three aspects in one prompt
- B) Routing — classify PR type first, then decide which checks to run
- C) Orchestrator-workers — central orchestrator delegates to three parallel worker agents
- D) Prompt chaining — do style, then security, then documentation sequentially

**Answer:** D — Prompt chaining is correct for this specific scenario because the workflow follows the same structured, predictable sequence for every PR (three defined steps in order). The orchestrator-workers pattern adds parallelism overhead that is only beneficial when order does not matter or when tasks are truly independent enough to justify separate context windows. For this bounded, three-step workflow, chaining is simpler and sufficient.

> **Exam tip:** The difference between orchestrator-workers and prompt chaining is subtle. Orchestrator-workers = dynamic decomposition + parallelism. Prompt chaining = fixed sequence of steps. If the steps are the same every time and sequential, chaining is often the better answer.

---

## 💡 Memory Tricks

**Orchestrator = Film Director:** The director doesn't act in every scene — they direct different actors (workers) and assemble the final film.

**Workers = Isolated Booths:** Each worker is in a soundproof booth. They hear only their assignment and return only their result. No cross-talk.

---

## ❓ Chapter Practice Questions

**Q1.** What is the primary benefit of running multiple worker subagents in parallel compared to a single agent completing the same tasks sequentially?

- A) Parallel subagents share context, allowing them to build on each other's findings
- B) Parallel subagents reduce total token consumption
- C) Parallel subagents can complete independent subtasks simultaneously, reducing total elapsed time
- D) Parallel subagents prevent the orchestrator from making mistakes

**Answer:** C

**Explanation:** When subtasks are independent (do not depend on each other's outputs), running them in parallel reduces wall-clock time. Each subagent works on its own part simultaneously. They do NOT share context (that would defeat the isolation benefit) and parallelism doesn't reduce tokens consumed.

---

**Q2.** An orchestrator assigns a security analysis subagent to review a codebase. The subagent should NOT be aware of the orchestrator's full reasoning or the outputs from other parallel subagents. Why?

- A) To save context window tokens for the orchestrator
- B) Context isolation ensures each subagent focuses on its specific task without being influenced by unrelated information from other workers
- C) Subagents cannot access the orchestrator's context due to technical limitations
- D) Sharing context between agents violates Anthropic's usage policy

**Answer:** B

**Explanation:** Context isolation is a design choice. By giving each subagent only the information relevant to its task, the orchestrator prevents cross-contamination — where one agent's reasoning (e.g., style concerns) influences another agent's analysis (e.g., security). This produces cleaner, more focused results.

---

**Q3.** A team is deciding between using a single agent or an orchestrator-workers pattern for a customer inquiry system where all customer questions go through a simple lookup → respond workflow. Which is more appropriate?

- A) Orchestrator-workers — always safer to have specialised workers
- B) Single agent — the task is simple and sequential; orchestrator overhead is not justified
- C) Orchestrator-workers — two workers are needed for lookup and respond
- D) Neither — use a rule-based system instead

**Answer:** B

**Explanation:** The orchestrator-workers pattern adds coordination overhead and complexity. For a simple, sequential, two-step workflow (lookup then respond), a single agent is simpler, faster, and equally effective. Add orchestrators only when tasks genuinely benefit from parallelism or isolation.

---

## 📌 Quick Revision Summary

- Orchestrator = breaks goal into subtasks, delegates, synthesises results
- Workers = focused, bounded subtasks in isolated context windows
- Parallel workers = faster for independent subtasks (no order dependency)
- Context isolation = each worker gets only what it needs — prevents cross-contamination
- Use when: multi-domain analysis, large-document tasks, parallelisation needed
- Don't use when: simple sequential task — prompt chaining or single agent is better

---

## 📎 References

- [Anthropic Documentation — Multi-Agent Systems](https://docs.anthropic.com/en/docs/agents)

---

*Notes by certification-study-hub. Chapter 02 — Orchestrator & Workers.*
