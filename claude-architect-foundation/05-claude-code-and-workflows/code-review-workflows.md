# Code Review Workflows — Independent Review & Batch Analysis

> **Exam:** Claude Architect Foundation
> **Chapter:** 05 — Claude Code & Developer Workflows
> **Difficulty:** Intermediate
> **Last Updated:** 2026-06

---

## 📖 What Is This Topic About?

Claude Code can automate and enhance code review workflows — from reviewing individual pull requests to batch-processing large review queues. The most important concept is the **independent review pattern**: using a fresh Claude instance to review code, rather than asking the generator to review its own work. This avoids confirmation bias and catches more real issues.

---

## 🧠 Key Concepts

- **Independent Review** — Using a separate Claude instance to review code, one that has no knowledge of the code's purpose or the generator's intent
- **Confirmation Bias** — The tendency for a reviewer to approve what they already believe to be correct; a generator reviewing its own code is highly susceptible to this
- **Review Chain** — A prompt chain that performs multiple independent review passes (security, style, documentation) on the same code
- **Batch Review** — Processing large review queues asynchronously, often overnight, to reduce cost
- **Structured Review Output** — Using tool use schemas to enforce consistent review report formats (severity, category, file, line, recommendation)
- **Multi-Aspect Review** — Reviewing the same code for different dimensions (security, performance, style) in separate, focused passes

---

## 📘 Detailed Explanation

### Why Independent Review Matters

When the same Claude instance that generated the code also reviews it, it has **confirmation bias**:

```
GENERATOR SESSION (knows intent, knows what it wrote):
"I implemented the payment function this way because it handles edge case X.
The function is correct."

REVIEW IN SAME SESSION:
"Review this payment function" → Claude looks at its own code through the lens 
of knowing why it wrote it that way → Tends to overlook flaws that conflict 
with the original intent

INDEPENDENT REVIEWER (fresh session, no context about generation):
"Review this payment function" → Claude reads the code at face value →
Spots issues that seem obvious in isolation but were invisible to the generator
```

**Independent review** = start a new session, provide only the code (no intent, no context, no "I already reviewed this") and ask for a fresh, unbiased analysis.

---

### The Review Chain Pattern

For thorough reviews, chain multiple focused passes:

```
┌───────────────────────────────────────────────────────────────────┐
│                        REVIEW CHAIN                               │
│                                                                   │
│  CODE CHANGE (PR diff or set of changed files)                    │
│                    │                                              │
│           ┌────────┴──────────────────────────────┐              │
│           │                                       │              │
│           ▼                                       │              │
│  ┌─────────────────────┐                          │              │
│  │  PASS 1:            │                          │              │
│  │  Security Review    │                          │              │
│  │  • Input validation │                          │              │
│  │  • Auth/authorisation│                         │              │
│  │  • SQL injection,   │                          │              │
│  │    XSS, path traversal                         │              │
│  └─────────┬───────────┘                          │              │
│            │  structured findings                 │              │
│            ▼                                      │              │
│  ┌─────────────────────┐                          │              │
│  │  PASS 2:            │                          │              │
│  │  Code Style Review  │◄─────────────────────────┘              │
│  │  • Naming conventions│                                         │
│  │  • Complexity, DRY  │                                         │
│  │  • Test coverage    │                                         │
│  └─────────┬───────────┘                                         │
│            │  structured findings                                 │
│            ▼                                                      │
│  ┌─────────────────────┐                                         │
│  │  PASS 3:            │                                         │
│  │  Documentation Review│                                        │
│  │  • Docstrings       │                                         │
│  │  • Inline comments  │                                         │
│  │  • README updates   │                                         │
│  └─────────┬───────────┘                                         │
│            │  structured findings                                 │
│            ▼                                                      │
│  SYNTHESIS PASS: Combine all findings into final review report    │
└───────────────────────────────────────────────────────────────────┘
```

---

### Structured Review Output Schema

Enforce consistent review output using tool use:

```json
{
  "name": "submit_code_review_finding",
  "description": "Submit a single code review finding",
  "input_schema": {
    "type": "object",
    "properties": {
      "severity": {
        "type": "string",
        "enum": ["critical", "major", "minor", "suggestion"],
        "description": "critical = security vulnerability or data loss risk; major = functional bug; minor = code quality issue; suggestion = improvement idea"
      },
      "category": {
        "type": "string",
        "enum": ["security", "performance", "logic", "style", "documentation", "testing"]
      },
      "file": {
        "type": "string",
        "description": "Relative file path"
      },
      "line_range": {
        "type": "string",
        "description": "Line number or range, e.g. '45' or '45-52'"
      },
      "description": {
        "type": "string",
        "description": "Clear description of the issue"
      },
      "recommendation": {
        "type": "string",
        "description": "Specific, actionable fix recommendation"
      }
    },
    "required": ["severity", "category", "file", "description", "recommendation"]
  }
}
```

This schema ensures every finding includes severity, category, location, and a specific recommendation — making findings directly actionable.

---

### Batch vs Real-Time Review Routing

Code reviews have varying urgency:

```
REAL-TIME REVIEW (Messages API):
Use when:
• PR is blocking a release or deployment
• Team is waiting for review to begin work
• Hotfix for production incident
• SLA: results within minutes

BATCH REVIEW (Message Batches API):
Use when:
• Large review queue (50+ PRs) accumulated during sprint
• Overnight security scan of the full codebase
• Weekly code quality report
• SLA: results within hours, not minutes
• 50% cost savings for non-urgent reviews
```

---

### Avoiding Confirmation Bias — Practical Checklist

```
┌────────────────────────────────────────────────────────────────┐
│           INDEPENDENT REVIEW CHECKLIST                         │
│                                                                │
│  ✅ Start a FRESH session (not the session that generated code)│
│  ✅ Provide ONLY the code — no design intent, no "this should   │
│     work because I intended X"                                 │
│  ✅ If reviewing your own PR, wait before reviewing (let context│
│     "fade") or have a teammate request the review              │
│  ✅ Use structured output schemas — forces complete findings    │
│  ✅ Separate review passes by focus area — don't do security + │
│     style + docs all at once in one pass                       │
│  ✅ Compare findings across reviewers — disagreements highlight │
│     truly ambiguous areas                                      │
└────────────────────────────────────────────────────────────────┘
```

---

## 🇮🇳 Indian Real-Life Example

**Think of independent review like the auditing process at a Chartered Accountancy firm:**

A junior CA prepares a client's tax return. Before it is filed, a senior CA — who was NOT involved in the preparation — independently audits it. This senior CA brings no knowledge of the junior's reasoning; they see only the numbers.

This catches:
- Incorrect deduction categories that the junior thought were valid
- Missed income sources that the junior had already "mentally filed" as excluded
- Calculation errors that seemed right in context of the junior's approach

The independent audit catches what internal review misses. Claude Code's independent review pattern works exactly the same way — a fresh session sees only the code, not the intentions.

---

## 🔑 Exam-Focused Points

- ✅ **Independent review** = fresh session, no context about the generator's intent
- ✅ Independent review catches errors that the generator would overlook due to **confirmation bias**
- ✅ **Review chains** (prompt chaining) run multiple focused passes: security → style → documentation
- ✅ **Structured review output** (tool use schema) ensures every finding has severity, category, location, and recommendation
- ✅ **Batch review** for non-urgent large queues: 50% cost savings, overnight processing
- ✅ **Real-time review** for blocking PRs, release-critical changes, production hotfixes

---

## 🧩 Scenario-Based Thinking

**Scenario:** A developer asks Claude Code to generate a caching layer for a database-heavy API. After generation, they ask the same Claude Code session to review the code for security vulnerabilities. Two weeks later, an audit finds a cache poisoning vulnerability that the review missed.

**What is the most likely root cause?**

- A) Claude Code's security analysis is unreliable for caching code
- B) The review was performed in the same session that generated the code; confirmation bias caused the reviewer to miss the vulnerability that was inherent to the design approach the generator chose
- C) The review pass was too brief; longer prompts would have caught the vulnerability
- D) Security review should have been performed before generation, not after

**Answer:** B

**Explanation:** When the same session reviews its own generated code, it has context about why it made certain design choices — and tends to evaluate those choices as correct. The cache poisoning vulnerability likely arose from a design decision that the generator "knew" was intentional, causing the reviewer (same session) to accept it. An independent reviewer, seeing only the code without design context, would have spotted the unexpected consequence.

---

## 💡 Memory Tricks

**Independent Review = New Set of Eyes:** Literally — a fresh session is a new set of eyes with no baggage. It sees the code exactly as a human reviewer who wasn't in the design meeting would see it.

**Review Chain = Assembly Line Quality Control:** Each station checks one specific thing. Security station, then style station, then documentation station. No single checker tries to catch everything — they each specialise.

---

## ❓ Chapter Practice Questions

**Q1.** A team uses Claude Code to generate API endpoints. The same developer who generated the code also asks Claude Code (in the same session) to review it. After deployment, several critical security issues are found. What workflow change would most directly address this?

- A) Use a different Claude model for code review
- B) Add security review prompts at the end of the generation session
- C) Always perform code review in a fresh session that receives only the code diff, with no information about the generator's intent or design decisions
- D) Perform code review before code generation

**Answer:** C

**Explanation:** The root cause is confirmation bias from the generator reviewing its own work. The fix is structural: independent review in a fresh session. The reviewer receives only the code — no design intent, no explanation of "why it was done this way." This removes the bias and allows genuine critical analysis.

---

**Q2.** A team has 75 pull requests that need review before a quarterly release. The release is in 5 days. Reviews don't need to block any specific deployment — they're used for the weekly code quality report. Which review approach is most cost-effective?

- A) Real-time API for all 75 PRs — reliability is most important
- B) Message Batches API for all 75 PRs — 50% cost savings, results within 24 hours, sufficient for the weekly report SLA
- C) Manual review for all 75 PRs — Claude should not review large batches
- D) Real-time for the first 10 critical PRs, skip the rest

**Answer:** B

**Explanation:** With a 5-day window before the release and no need to block specific work, the Batch API is ideal. 50% cost savings on 75 PR reviews is significant, and 24-hour turnaround fits comfortably within the 5-day window. Real-time API (A) for non-urgent reviews wastes cost.

---

**Q3.** A review chain runs three passes on a PR: security, style, and documentation. During integration, the team finds that security findings are frequently marked as "minor" in a combined review report, being overshadowed by more numerous style findings. What architecture change best addresses this?

- A) Run all three review passes in a single prompt with instructions to prioritise security
- B) Weight each pass equally in the final report generation
- C) Use separate structured output schemas per pass, with severity levels specific to each category, then synthesise into a final report where security findings above "minor" are always surfaced prominently
- D) Remove the synthesis step and present all three reports separately

**Answer:** C

**Explanation:** The problem is that mixing security and style findings in a combined report obscures critical security issues behind more numerous style findings. Separate structured schemas per pass ensure security severity is calibrated against security benchmarks (not style benchmarks), and the synthesis step can explicitly surface any security findings above a threshold — preventing them from being buried in volume.

---

## 📌 Quick Revision Summary

- Independent review = fresh session + code only; no generator context → avoids confirmation bias
- Review chain (prompt chaining): separate passes by focus area → security, style, documentation
- Structured review output (tool use schema): severity + category + file + line + recommendation
- Batch API for large review queues with non-urgent SLAs: 50% cost savings
- Real-time API for blocking PRs, hotfixes, and release-critical changes
- Multi-aspect review in separate passes: each specialises → no single dimension drowns out others

---

## 📎 References

- [Anthropic Documentation — Claude Code for Code Review](https://docs.anthropic.com/en/docs/claude-code)

---

*Notes by certification-study-hub. Chapter 05 — Code Review Workflows.*
