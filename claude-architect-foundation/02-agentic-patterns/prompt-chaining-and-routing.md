# Prompt Chaining & Routing — Sequential & Conditional Patterns

> **Exam:** Claude Architect Foundation
> **Chapter:** 02 — Agentic Patterns
> **Difficulty:** Intermediate
> **Last Updated:** 2026-06

---

## 📖 What Is This Topic About?

Not every task needs a full orchestrator-workers setup. Two simpler but powerful patterns — **prompt chaining** and **routing** — cover a large portion of real-world agentic workflows. Prompt chaining breaks a task into sequential steps. Routing classifies the input first, then sends it down the appropriate path. Knowing when to use which is an important exam skill.

---

## 🧠 Key Concepts

- **Prompt Chaining** — Breaking a task into a fixed sequence of steps where each step's output becomes the next step's input
- **Routing** — Classifying an input first, then routing to a different sub-prompt or sub-agent optimised for that category
- **Gate** — A validation or review step between chain links that can approve or reject before proceeding
- **Parallelisation** — Running multiple independent chain steps simultaneously (combining chaining with workers)
- **Workflow** — The overall structure of how prompts and steps connect to produce a final output

---

## 📘 Detailed Explanation

### Pattern 1: Prompt Chaining (Sequential Steps)

Prompt chaining is like an assembly line. Each step produces something that the next step uses.

```
┌─────────────────────────────────────────────────────────────────┐
│                    PROMPT CHAINING                              │
│                                                                 │
│  INPUT                                                          │
│    │                                                            │
│    ▼                                                            │
│  ┌──────────────┐                                              │
│  │  Step 1      │  e.g., "Translate contract to English"       │
│  └──────┬───────┘                                              │
│         │ output                                               │
│         ▼                                                       │
│  ┌──────────────┐                                              │
│  │  Step 2      │  e.g., "Extract key clauses from translation"│
│  └──────┬───────┘                                              │
│         │ output                                               │
│         ▼                                                       │
│  ┌──────────────┐                                              │
│  │  Step 3      │  e.g., "Flag any clauses with legal risk"    │
│  └──────┬───────┘                                              │
│         │ output                                               │
│         ▼                                                       │
│  FINAL OUTPUT                                                   │
└─────────────────────────────────────────────────────────────────┘
```

**When to use prompt chaining:**
- The task follows a predictable, fixed sequence of steps
- Each step genuinely needs the output of the previous step
- You want to isolate and test each step independently
- You want a gate between steps (e.g., human review before proceeding)

**Classic examples:**
- Research report: search → summarise sources → write report
- Code review: style check → security check → documentation check (sequential version)
- Contract analysis: extract → classify clauses → generate risk summary

---

### Adding Gates Between Steps

A gate is a validation step between two chain links. The system only proceeds if the gate passes:

```
Step 1 Output → [GATE: Is output valid?] → Yes → Step 2
                                        → No  → Retry Step 1 / Flag for human
```

Gates are used for:
- Quality checks (did Step 1 produce complete output?)
- Compliance checks (does Step 2's output meet policy requirements?)
- Human review checkpoints (does a human approve before Step 3 executes?)

---

### Pattern 2: Routing (Classify → Dispatch)

Routing is used when different types of inputs need fundamentally different handling. Instead of one universal prompt, you first classify the input and then use a specialised path for each category.

```
┌─────────────────────────────────────────────────────────────────┐
│                         ROUTING                                 │
│                                                                 │
│  INPUT: Customer message                                        │
│         │                                                       │
│         ▼                                                       │
│  ┌──────────────────────────────────────────┐                  │
│  │  CLASSIFIER                               │                  │
│  │  "What type of request is this?"          │                  │
│  │  → refund_request                         │                  │
│  │  → technical_support                      │                  │
│  │  → billing_inquiry                        │                  │
│  └───┬──────────────┬───────────────┬────────┘                 │
│      │              │               │                           │
│      ▼              ▼               ▼                           │
│  ┌───────┐     ┌───────┐     ┌───────────┐                    │
│  │Refund │     │Tech   │     │Billing    │                    │
│  │Agent  │     │Support│     │Agent      │                    │
│  │Prompt │     │Prompt │     │Prompt     │                    │
│  └───────┘     └───────┘     └───────────┘                    │
└─────────────────────────────────────────────────────────────────┘
```

**When to use routing:**
- Different types of inputs require very different expertise or tool access
- You want optimised, specialised prompts for each category
- You want to send different request types to different models (cost optimisation)
- Natural language input needs to map to a structured category

---

### Pattern Comparison Table

| Dimension | Prompt Chaining | Routing | Orchestrator-Workers |
|---|---|---|---|
| **Structure** | Linear sequence | Classify → branch | Hub and spoke |
| **Steps** | Fixed, known upfront | Single classification + one path | Dynamic, determined by orchestrator |
| **Use when** | Task has predictable steps | Input type determines handling | Task is complex + parallel subtasks |
| **Complexity** | Low | Low | High |
| **Example** | Translate → Extract → Summarise | Classify intent → Run specialist | Analyse codebase across 5 areas |

---

### Combining Patterns

Real-world systems often combine patterns:

```
ROUTING + CHAINING:
  Classify input → Route to specialist chain → Execute steps

CHAINING + PARALLELISATION:
  Step 1 (sequential) → [Step 2A ∥ Step 2B ∥ Step 2C] (parallel) → Step 3 (synthesis)
```

---

## 🇮🇳 Indian Real-Life Example

**Prompt Chaining = Tiffin Box Assembly Line:**

At a large corporate cafeteria, preparing 500 tiffin boxes follows a fixed chain:
1. Cook rice (Step 1)
2. Prepare dal using today's cooked rice as base (Step 2)
3. Pack both into tiffin using today's dal and rice (Step 3)

Each step uses the output of the previous step. The sequence never changes.

**Routing = Hospital Reception Desk:**

When a patient arrives at a hospital:
1. Receptionist determines their need: Emergency? OPD? Lab test? Pharmacy?
2. Routes them to the appropriate department

A cardiologist shouldn't be handling pharmacy queries. Routing sends each patient to the right specialist.

---

## 🔑 Exam-Focused Points

- ✅ **Prompt Chaining** = fixed sequential steps, each using the previous step's output
- ✅ **Routing** = classify first, then dispatch to a specialised path or sub-prompt
- ✅ Gates between chain steps = validation/review checkpoints that can pause or fail
- ✅ Chaining is simpler than orchestrator-workers; use chaining for predictable linear workflows
- ✅ Routing is best when different **types** of inputs need different handling
- ✅ Patterns can be combined: routing + chaining, chaining + parallelisation

---

## 🧩 Scenario-Based Thinking

**Scenario:** A code review assistant must always perform three analyses for every pull request: code style compliance, potential security issues, and documentation completeness — in that defined order.

**Which pattern is most appropriate?**

- A) Routing — classify PR type first, route to different review prompts
- B) Orchestrator-workers — central LLM dynamically determines needed checks
- C) Single comprehensive prompt — handle all three simultaneously
- D) Prompt chaining — three sequential analysis steps, each building on prior output

**Answer:** D — Prompt chaining. The three steps are the same for every PR (no classification needed, so routing doesn't apply). The steps follow a defined order. This is a classic prompt chain: three sequential steps with each step potentially referencing prior analysis.

---

## 💡 Memory Tricks

**Chain = Train:** Each carriage follows the previous one, in a fixed order. The train can't leave until all carriages are hitched.

**Routing = Traffic Roundabout:** Traffic comes in, gets assessed (which exit?), and is sent down the correct road. No two cars take the same exit unless they have the same destination.

---

## ❓ Chapter Practice Questions

**Q1.** A system receives customer messages in English, Hindi, and Tamil. The system must first translate non-English messages, then classify the intent, then route to a specialist. Which pattern does this best describe?

- A) Pure routing
- B) Pure prompt chaining
- C) Routing embedded within a prompt chain
- D) Orchestrator-workers with three subagents

**Answer:** C

**Explanation:** The overall flow is a chain (translate → classify → route to specialist). The routing step is embedded within the chain. The first two steps are sequential (translate feeds into classify), and then routing dispatches to a specialist. This is chaining with routing as one of the steps.

---

**Q2.** A system must generate a marketing email in three steps: (1) draft the content, (2) review for compliance, (3) send via the email API. During testing, compliance review fails 15% of the time. What mechanism should be added between Steps 2 and 3?

- A) A retry loop that re-runs Step 1 with a different temperature
- B) A gate that blocks execution of Step 3 if Step 2 fails compliance review
- C) A parallel worker that handles failed compliance reviews simultaneously
- D) A router that classifies whether the email needs compliance review

**Answer:** B

**Explanation:** A gate between Steps 2 and 3 prevents the email from being sent if it fails compliance review. The gate checks the Step 2 output and either allows the chain to proceed to Step 3 or halts and flags for revision. This is the standard pattern for preventing downstream execution when a mid-chain validation fails.

---

**Q3.** Which pattern is MOST appropriate when a customer support system handles three completely different types of requests — returns, billing disputes, and product questions — each requiring different tool access and different specialised knowledge?

- A) Prompt chaining — three sequential steps for all request types
- B) Routing — classify the request type, then dispatch to a specialist sub-prompt with the appropriate tools
- C) Orchestrator-workers — use all three specialists in parallel for every request
- D) Single comprehensive prompt — include all three specialisations in one system prompt

**Answer:** B

**Explanation:** Routing is ideal when the *type* of input determines the handling. Each request type needs different tools and different expertise. Routing classifies the request and sends it to the most appropriate specialised sub-prompt. Running all three specialists in parallel for every request (C) is wasteful. A single giant prompt (D) dilutes specialisation.

---

## 📌 Quick Revision Summary

- Prompt Chaining = fixed sequential steps, each using prior output; simple and predictable
- Routing = classify input first, dispatch to specialised path or sub-prompt
- Gate = validation checkpoint between chain steps; blocks or retries on failure
- Chaining is simpler than orchestrator-workers; use it for linear, predictable workflows
- Routing is best when different input types need fundamentally different handling
- Patterns combine: routing + chaining, chaining + parallelisation — both are common

---

## 📎 References

- [Anthropic Documentation — Prompt Chaining](https://docs.anthropic.com/en/docs/prompt-chaining)
- [Anthropic Documentation — Building Effective Agents](https://docs.anthropic.com/en/docs/agents)

---

*Notes by certification-study-hub. Chapter 02 — Prompt Chaining & Routing.*
