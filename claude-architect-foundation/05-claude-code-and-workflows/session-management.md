# Session Management — Continue, Resume & Fork

> **Exam:** Claude Architect Foundation
> **Chapter:** 05 — Claude Code & Developer Workflows
> **Difficulty:** Beginner → Intermediate
> **Last Updated:** 2026-06

---

## 📖 What Is This Topic About?

Claude Code sessions are not disposable. After spending 45 minutes investigating a codebase, that accumulated knowledge is valuable. Session management covers how to **continue** an ongoing session, **resume** a previously paused one, and **fork** a session to explore alternative approaches — all without losing the context built up so far.

---

## 🧠 Key Concepts

- **Session** — A Claude Code conversation thread that includes all tool calls, file reads, and reasoning from start to current
- **`--continue`** — Resumes the most recent session without specifying a name or ID
- **`--resume <name>`** — Resumes a previously named session by its name
- **`--session-id <UUID>`** — Resumes a session by its unique identifier
- **Session Fork** — Creates two independent branches from the same session checkpoint; each branch can explore a different approach without contaminating the other
- **Session Naming** — Giving a session a memorable name (e.g., `auth-module-refactor`) so it can be resumed by name

---

## 📘 Detailed Explanation

### Why Session Management Matters

When a session ends (connection drop, end of day, system restart), the default behaviour is a fresh start — all accumulated knowledge is lost. Session management prevents this:

```
WITHOUT SESSION MANAGEMENT:
Day 1: 2 hours investigating the payment processing module
        → Session ends → Context lost
Day 2: Start fresh → 2 more hours to rebuild understanding
        → Total: 4 hours of investigation for one task

WITH SESSION MANAGEMENT:
Day 1: 2 hours investigating the payment processing module
        → Named session: "payment-module-investigation"
        → Session paused
Day 2: Resume "payment-module-investigation"
        → Continue from exactly where you stopped
        → Total: 2 hours (+ brief catch-up on overnight changes)
```

---

### The Three Resume Options

```
┌───────────────────────────────────────────────────────────────────┐
│                    SESSION RESUME OPTIONS                          │
│                                                                    │
│  1. --continue (most recent session)                              │
│  ─────────────────────────────────────────────────────────────    │
│  $ claude --continue                                              │
│  → Picks up the last session automatically                        │
│  → No need to remember a session name                             │
│  → Best for: returning to work interrupted moments ago            │
│                                                                    │
│  2. --resume <name> (named session)                               │
│  ─────────────────────────────────────────────────────────────    │
│  $ claude --resume payment-module-investigation                   │
│  → Resumes the session you named                                  │
│  → Good for: returning to a long-running investigation or feature  │
│  → Best practice: name sessions with descriptive kebab-case names │
│                                                                    │
│  3. --session-id <UUID> (by session identifier)                   │
│  ─────────────────────────────────────────────────────────────    │
│  $ claude --session-id 7f3a9b21-4c8d-11ef-b864-0242ac120002      │
│  → Resumes using the UUID from the session's log file             │
│  → Good for: automation scripts, referencing a specific session    │
│              precisely                                             │
└───────────────────────────────────────────────────────────────────┘
```

---

### Critical Rule: Notify the Agent of Changes During Pause

When resuming a session, Claude's knowledge is a **snapshot from when the session was paused**. If anything changed while the session was inactive, Claude doesn't know:

```
EXAMPLE:
Session paused at 6 PM: "The auth module uses AuthService class"
Overnight: A colleague renamed AuthService to JWTAuthService in a PR
Session resumed at 9 AM: Claude still believes the class is AuthService

WHAT HAPPENS WITHOUT NOTIFICATION:
Claude calls grep for "AuthService" → finds nothing
Claude is confused, may spend time investigating the "missing" class

WHAT HAPPENS WITH NOTIFICATION:
"Before we continue: AuthService was renamed to JWTAuthService 
 in a PR merged overnight."
→ Claude immediately updates its understanding and proceeds correctly
```

**Always brief the agent at the start of a resumed session about:**
- Files that changed while the session was paused
- PRs that were merged or reverted
- Environment changes (config updates, dependency upgrades)
- Any decisions made by the team while the agent was inactive

---

### Session Forking — Exploring Two Paths Simultaneously

When you reach a decision point and want to explore two approaches without one contaminating the other:

```
┌─────────────────────────────────────────────────────────────────┐
│                     SESSION FORKING                             │
│                                                                 │
│  ORIGINAL SESSION:                                             │
│  "We've analysed the authentication module for 40 minutes.     │
│   It has a known performance bottleneck.                        │
│   Two approaches are possible:                                  │
│   (A) Implement Redis caching                                   │
│   (B) Optimise the DB query directly"                           │
│                    │                                            │
│         ┌──────────┴──────────┐                                │
│         │                     │                                │
│      FORK A                FORK B                              │
│  "Explore Redis             "Explore DB query                   │
│   caching approach"          optimisation approach"            │
│   (independent context)      (independent context)             │
│         │                     │                                │
│         ▼                     ▼                                │
│  Developer reviews both branches' findings                     │
│  and selects the better approach                               │
│         │                                                      │
│  CONTINUE with chosen approach                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Why not just explore both in one session sequentially?**
- Approach A's exploration influences Approach B's
- If Approach A leads to a dead end, the context is polluted with failed attempts
- Forking gives clean, independent, comparable results from the same starting knowledge base

---

### When to Start Fresh vs Resume

```
START FRESH WHEN:
✅ The previous session's context is no longer relevant
✅ The codebase has changed so significantly that prior knowledge is misleading
✅ You're starting a completely different task
✅ The session ran into fundamental misunderstandings that were never corrected

RESUME WHEN:
✅ The task is not yet complete and the prior investigation is still valid
✅ Only minor changes occurred during the pause
✅ You explicitly inform the agent of any changes before continuing
✅ The accumulated understanding of the codebase is valuable
```

---

## 🇮🇳 Indian Real-Life Example

**Session Management = A Court Case File at a Law Office:**

A junior advocate working on a complex corporate case doesn't start from scratch every morning. They have a case file (session) with all the research, notes, and findings from previous days.

- **`--continue`** = picking up the file from your desk where you left it last night
- **`--resume case-name`** = retrieving a specific case file from the filing cabinet
- **Forking** = making a photocopy of the current case file to explore two different legal strategies in parallel without mixing their notes
- **Briefing after pause** = "Before we continue, the opposing counsel filed a new exhibit this morning — here it is"

---

## 🔑 Exam-Focused Points

- ✅ `--continue` = resume most recent session; `--resume <name>` = resume by name
- ✅ Session knowledge is a **snapshot from when the session was paused** — always notify agent of changes
- ✅ **Forking** = two independent branches from same checkpoint; prevents cross-contamination
- ✅ Fork for: exploring two approaches, comparing implementations, avoiding dead-end contamination
- ✅ Start fresh when: context is outdated, task completely changed, fundamental misunderstandings persist
- ✅ Resume when: task is incomplete, prior knowledge is valid, changes can be summarised briefly

---

## 🧩 Scenario-Based Thinking

**Scenario:** A developer spent 2 hours with Claude Code analysing a large monolith and identifying its module boundaries. They now want Claude to simultaneously draft architecture diagrams for two different decomposition strategies: microservices and modular monolith. How should they structure this?

- A) Have Claude draft both architectures in the same session — more efficient
- B) Fork the session at the current point; explore microservices in Branch A, modular monolith in Branch B
- C) Start two completely fresh sessions with a summary of the analysis
- D) Resume the session and complete one architecture, then continue for the second

**Answer:** B

**Explanation:** Forking preserves the full 2-hour analysis in both branches and prevents each strategy's design decisions from contaminating the other. Starting fresh (C) loses the accumulated analysis (or requires time to re-state it). Doing both in sequence (D) means the second architecture is influenced by the reasoning developed during the first.

---

## 💡 Memory Tricks

**`--continue` = Ctrl+Z for your whole session:** Just un-close what you had. The last session comes right back.

**Forking = Git Branch:** Just like `git branch feature-A` creates a diverging path from `main`, session forking creates a diverging path from the current session checkpoint. Both can be compared and one can be chosen.

---

## ❓ Chapter Practice Questions

**Q1.** A developer was debugging a complex race condition in a payment service for 90 minutes when their laptop crashed. They restart and want to continue. What is the most efficient approach?

- A) Start a fresh session and rebuild the investigation from scratch
- B) Use `--continue` to resume the most recent session and then brief Claude on the crash and whether any files changed during recovery
- C) Use `--resume` with the session ID found in the session log file
- D) Summarise the 90 minutes of findings in a new session prompt

**Answer:** B (or C — both are correct; B is more convenient for the most recent session)

**Explanation:** `--continue` is the simplest way to pick up the most recent session. After resuming, briefly inform Claude that the laptop crashed and whether any files were auto-saved or reverted. This preserves all investigation context without having to re-discover the race condition location, attempted solutions, and analysis.

---

**Q2.** After resuming a Claude Code session, a developer notices Claude references a service class that was deleted and replaced by a completely different class during the session pause. Claude's suggestions are therefore incorrect. What is the primary cause and fix?

- A) Claude's in-weights knowledge is outdated; nothing can be done
- B) The developer failed to notify Claude of the structural change at the start of the resumed session; they should now inform Claude of the deleted class and its replacement to update its working understanding
- C) The session should be abandoned because any prior analysis is now invalid
- D) Resume sessions should not be used when code changes occur

**Answer:** B

**Explanation:** Claude's session knowledge is a snapshot from the pause point. It doesn't automatically detect changes. The fix is straightforward: inform Claude of the change ("ServiceA was deleted and replaced by ServiceB with these key differences..."). After this briefing, Claude can update its working model and continue effectively.

---

**Q3.** A developer wants to compare two caching strategies for a database-heavy API. They have accumulated 45 minutes of analysis about the API's data access patterns. What approach best supports exploring both strategies without one influencing the other?

- A) Explore Strategy A until complete, then continue in the same session with Strategy B
- B) Start two fresh sessions with a brief summary of the data access patterns
- C) Fork the session at the 45-minute checkpoint; explore Strategy A in Branch A and Strategy B in Branch B independently
- D) Add both strategies to the same session and let Claude choose the better one

**Answer:** C

**Explanation:** Forking creates two independent branches from the same 45-minute analysis. Each branch explores its strategy in a clean context, uninfluenced by the other's findings. This preserves the accumulated analysis in both branches and allows a genuine, uncontaminated comparison. Sequential exploration (A) introduces contamination; fresh starts (B) lose 45 minutes of valuable context.

---

## 📌 Quick Revision Summary

- `--continue`: resume most recent session; `--resume <name>`: resume by session name; `--session-id <UUID>`: precise resume
- Session knowledge is a snapshot from pause time — always brief the agent on changes after resuming
- Forking: two independent branches from same checkpoint; for exploring alternative approaches
- Start fresh: outdated context, completely new task, fundamental misunderstandings
- Resume: task incomplete, prior knowledge valid, changes can be summarised briefly
- Forking ≠ starting fresh — forking preserves accumulated knowledge in both branches

---

## 📎 References

- [Anthropic Documentation — Claude Code Session Management](https://docs.anthropic.com/en/docs/claude-code)

---

*Notes by certification-study-hub. Chapter 05 — Session Management.*
