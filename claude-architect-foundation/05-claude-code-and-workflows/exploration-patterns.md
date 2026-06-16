# Exploration Patterns — How Claude Code Investigates Codebases

> **Exam:** Claude Architect Foundation
> **Chapter:** 05 — Claude Code & Developer Workflows
> **Difficulty:** Intermediate
> **Last Updated:** 2026-06

---

## 📖 What Is This Topic About?

Before making changes to a codebase, Claude Code must *understand* the codebase. How it explores — starting with architecture or diving into files, using Grep or Read, breadth-first or depth-first — determines how quickly and accurately it builds a working model of the code. This chapter covers the most effective exploration patterns.

---

## 🧠 Key Concepts

- **Architecture-First Exploration** — Starting by understanding high-level structure (directory layout, key modules, entry points) before reading individual files
- **File-First Exploration** — Reading specific files directly, often less efficient for unfamiliar codebases
- **Adaptive Investigation** — Adjusting the exploration strategy based on what is discovered (start broad, narrow to relevant areas)
- **Breadth-First** — Exploring many areas shallowly before reading any area deeply
- **Depth-First** — Reading one area completely before moving to the next
- **Plan Mode Exploration** — Using Plan Mode to explore and form a plan without making any file changes

---

## 📘 Detailed Explanation

### Architecture-First vs File-First

For unfamiliar codebases, architecture-first is almost always more efficient:

```
FILE-FIRST EXPLORATION (less efficient):
1. Read routes/user.py → understand user endpoints
2. Read routes/products.py → understand product endpoints
3. Read models/user.py → understand User model
... (read 25 files before understanding how they connect)
Problem: Lots of reading before understanding the structure

ARCHITECTURE-FIRST EXPLORATION (more efficient):
1. List the top-level directories → understand project layout
2. Read CLAUDE.md / README.md → understand project overview
3. Glob for entry points (main.py, app.py, manage.py)
4. Read the main entry point → understand how the app is initialised
5. Grep for key class/function imports → understand module relationships
6. THEN read specific files for the areas relevant to the task

Result: Understand structure first, then read only what's needed
```

---

### The Adaptive Investigation Pattern

For most real-world tasks, the best approach is adaptive: start broad, then narrow to what matters:

```
┌──────────────────────────────────────────────────────────────────┐
│                   ADAPTIVE INVESTIGATION                         │
│                                                                  │
│  PHASE 1: ORIENT (broad view)                                    │
│  ────────────────────────────────────────────────────────────    │
│  • List directory structure                                      │
│  • Read README / CLAUDE.md                                       │
│  • Identify main entry points                                    │
│  • Understand technology stack                                   │
│  Goal: Form a map of the codebase                               │
│                     │                                            │
│                     ▼                                            │
│  PHASE 2: LOCATE (narrow to relevant area)                       │
│  ────────────────────────────────────────────────────────────    │
│  • Grep for the class, function, or concept related to the task  │
│  • Glob for relevant file patterns (*.model.ts, *service.py)     │
│  • Identify the 3–5 most relevant files                          │
│  Goal: Narrow from hundreds of files to the handful that matter  │
│                     │                                            │
│                     ▼                                            │
│  PHASE 3: UNDERSTAND (read deeply)                               │
│  ────────────────────────────────────────────────────────────    │
│  • Read the identified relevant files                            │
│  • Follow imports to understand dependencies                     │
│  • Read test files to understand expected behaviour              │
│  Goal: Build deep understanding of the specific area to change   │
│                     │                                            │
│                     ▼                                            │
│  PHASE 4: PLAN then ACT                                          │
│  ────────────────────────────────────────────────────────────    │
│  • Document the planned changes (Plan Mode)                      │
│  • Identify all files that must change                           │
│  • Execute changes in dependency order                           │
└──────────────────────────────────────────────────────────────────┘
```

---

### When to Use Grep vs When to Read

```
USE GREP WHEN:
• You're looking for a specific pattern across many files
• "Find all usages of deprecated function X"
• "Find all files that import module Y"
• "Find all TODO comments"
• "Find all API endpoints that accept POST requests"
→ Grep returns targeted results without reading entire files

USE READ WHEN:
• You need to understand the full context of specific code
• "What does this authentication function actually do, step by step?"
• "What are all the config options in this settings file?"
• "Understand the complete test suite for this module"
→ Read gives full file content for deep understanding

THE DECISION:
"Do I know what I'm looking for (pattern)?" → GREP
"Do I need to understand a complete section?" → READ
```

---

### Plan Mode — Explore Without Changing

For large or risky changes, use Plan Mode: Claude explores the codebase fully and produces a detailed plan before making any file changes.

```
PLAN MODE WORKFLOW:

Claude Code enters Plan Mode when asked for complex changes:
"Refactor the user authentication module to use JWT tokens instead of session cookies"

Step 1: EXPLORE
→ Read the current authentication implementation
→ Grep for all session-related code across the codebase
→ Identify all files that must change
→ Understand test coverage of auth module

Step 2: PLAN
→ Document: "I will change these files in this order:
   1. models/session.py → models/token.py (new model)
   2. services/auth.py → Update token generation
   3. middleware/auth.py → Update token validation
   4. tests/test_auth.py → Update 14 test cases"

Step 3: USER REVIEWS PLAN
→ User can approve, modify, or reject before any file changes happen

Step 4: EXECUTE (only after approval)
→ Make changes in the planned order

BENEFIT:
If the plan is wrong, you caught it BEFORE any files were changed.
```

---

### Depth-First vs Breadth-First — When Each Helps

| Strategy | Description | Best For |
|---|---|---|
| **Depth-First** | Fully understand one module before moving to the next | Deeply understanding one specific component (e.g., "how does auth work?") |
| **Breadth-First** | Survey all relevant areas shallowly first | Understanding cross-cutting concerns (e.g., "where is auth used?") |
| **Adaptive (recommended)** | Start breadth-first for orientation, shift to depth-first for relevant areas | Most real-world tasks |

---

## 🇮🇳 Indian Real-Life Example

**Think of codebase exploration like a new CBI investigator arriving at a case:**

1. **Orient:** Read the case summary and all prior reports first — before examining any specific evidence. (Architecture-first)
2. **Locate:** Identify which specific exhibits or witnesses are relevant to the current investigation question. (Grep/Glob)
3. **Understand:** Read those specific exhibits thoroughly. (Read)
4. **Plan:** Document the investigation approach before taking any actions.

An investigator who immediately starts reading unrelated case files wastes time. An investigator who understands the case layout first, then focuses on relevant evidence, solves the case faster.

---

## 🔑 Exam-Focused Points

- ✅ **Architecture-first** exploration is more efficient than file-first for unfamiliar codebases
- ✅ **Adaptive investigation**: Orient (broad) → Locate (narrow) → Understand (deep) → Plan → Act
- ✅ **Grep** for finding patterns across files; **Read** for deep understanding of specific files
- ✅ **Plan Mode**: explore and plan without making changes; user reviews before execution
- ✅ For large changes: ALWAYS explore and plan before touching files — prevents cascading mistakes
- ✅ Read test files as part of understanding: they reveal expected behaviour better than implementation

---

## 🧩 Scenario-Based Thinking

**Scenario:** Claude Code is asked to add a new field to a database model. The model is part of a large Django application with 50+ models. Claude immediately writes a migration file and edits the model. After running tests, 12 tests fail because several serialisers, views, and admin registrations that Claude didn't investigate also need to be updated.

**What exploration step did Claude skip?**

- A) Reading the project's CLAUDE.md
- B) Using Grep to locate all usages of the model (serialisers, views, admin registrations) before making changes
- C) Reading the migration files directory
- D) Using Plan Mode's test suite analysis

**Answer:** B

**Explanation:** The adaptive investigation pattern requires locating all usages of what you're changing before making changes. Grepping for the model name across the codebase would have revealed the serialisers, views, and admin registrations that also needed updating. Making the model change first without this investigation caused cascading failures.

---

## 💡 Memory Tricks

**Orient → Locate → Understand → Plan → Act:** OLUPA — "Only Logical Understanding Prevents Accidents." Each step is a safety check before the next.

**Grep = Compass:** Tells you which direction to go (where the pattern appears). Read = Map: Shows you exactly what's in that territory. You use the compass to choose which map to read.

---

## ❓ Chapter Practice Questions

**Q1.** Claude Code is asked to remove a deprecated API endpoint from a large Express.js application. What exploration approach most effectively identifies all code that must change?

- A) Read all files in the routes/ directory
- B) Use Glob to find all *.js files, then read each one
- C) Use Grep to find all references to the deprecated endpoint across the codebase (route definition, documentation, tests, client calls, middleware), then read only those files
- D) Read only the routes/api.js file where the endpoint is defined

**Answer:** C

**Explanation:** Architecture-first with Grep: search for the endpoint name/path across all files to find every location that references it. This surfaces route definitions, documentation, tests, client code, and middleware — all of which may need updating. Reading only the routes file (D) will cause cascading failures from all the other references.

---

**Q2.** A developer asks Claude Code to refactor a payment processing module that is used by 8 other modules across the application. What is the most appropriate initial step?

- A) Start editing the payment module immediately to avoid wasting context on exploration
- B) Enter Plan Mode: Grep for all imports and usages of the payment module, identify all 8 dependent modules, read them to understand coupling, then document a complete list of files to change before making any edits
- C) Ask the developer to provide a list of all files that need changing before starting
- D) Read the payment module fully, then start making changes module by module

**Answer:** B

**Explanation:** A refactoring with 8 dependent modules is exactly the scenario where Plan Mode is essential. Grep for all usages, understand all coupling, document the complete change plan, and get the user's approval BEFORE making any changes. Starting to edit immediately (A) risks making breaking changes to the payment module before understanding all dependencies.

---

**Q3.** A developer reports that Claude Code is spending excessive time reading files that turn out to be irrelevant to the task. For a task involving database query optimisation, Claude has read 35+ files including unrelated frontend components. What is the most likely cause and fix?

- A) Claude Code should always read all files for complete context; this is expected behaviour
- B) The lack of a CLAUDE.md explaining project structure means Claude must explore broadly; adding a CLAUDE.md with architecture overview would help Claude target relevant areas faster
- C) Claude Code should be restricted to only the backend directory
- D) The task description is too vague; be more specific about which database queries to optimise

**Answer:** B

**Explanation:** Without a CLAUDE.md explaining project structure, Claude must discover the architecture through exploration — which often includes reading irrelevant files before understanding which files are relevant. A CLAUDE.md with an architecture overview ("backend is in /api/, frontend in /web/, queries optimised via /api/repositories/") allows Claude to immediately target the relevant area. This is one of CLAUDE.md's key value propositions.

---

## 📌 Quick Revision Summary

- Architecture-first: understand structure before reading individual files → more efficient
- Adaptive investigation: Orient → Locate (Grep) → Understand (Read) → Plan → Act
- Grep: finding patterns across files; Read: understanding specific file content in full
- Plan Mode: explore and plan without making changes; essential for large/risky refactors
- Read test files: reveal expected behaviour, which tests must stay passing after changes
- CLAUDE.md with architecture overview: helps Claude target relevant areas faster, reduces unnecessary exploration

---

## 📎 References

- [Anthropic Documentation — Claude Code Best Practices](https://docs.anthropic.com/en/docs/claude-code)

---

*Notes by certification-study-hub. Chapter 05 — Exploration Patterns.*
