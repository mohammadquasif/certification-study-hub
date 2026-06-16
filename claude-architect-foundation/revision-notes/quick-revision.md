# ⚡ Quick Revision Guide — Day Before the Exam

> **Exam:** Claude Architect Foundation
> **Last Updated:** 2026-06

---

## 🤖 Domain 1 — Claude Fundamentals (~15%)

- **Claude** = AI assistant by **Anthropic** (not Google, not OpenAI, not Amazon)
- **HHH** = Helpful, Harmless, Honest — the three core design principles
- **Constitutional AI** = Claude self-evaluates responses against guiding principles during training
- **Context window** = up to **200K tokens** — shared by: system prompt + tool definitions + history + documents + response
- **Stateless** = Claude has NO memory between API calls; your application must re-send conversation history
- **Four memory types**: in-context, external (retrieved via tools), cached, in-weights (training)
- **When NOT to use Claude**: guaranteed deterministic output, safety-critical real-time decisions, zero-hallucination legal/medical with no human review
- **Responsible deployment** = developer's co-responsibility; a safe model + unsafe deployment = unsafe system

---

## 🔄 Domain 2 — Agentic Patterns (~25%)

### The Agentic Loop
**Observe → Think → Act → Observe** (repeats until complete)
- Tool results are added to context; Claude reasons about next step each iteration

### Patterns — Quick Comparison

| Pattern | Structure | Use When |
|---|---|---|
| **Prompt Chaining** | Fixed sequential steps | Same steps every time; predictable workflow |
| **Routing** | Classify → dispatch | Different input types need different handling |
| **Orchestrator-Workers** | Hub + parallel branches | Multi-domain, parallel, large-scale |
| **Single Agent** | Direct tool calls | Simple focused task |

### Key Rules
- **Human-in-the-loop**: irreversible actions, high-stakes decisions, amounts above threshold
- **Escalation = structured handoff**: not just transcript — include diagnosis + next recommended action
- **Context fills during long sessions** → accuracy degrades for early content → summarise and use external memory
- **Session forking** = two independent branches from same checkpoint; prevents cross-contamination
- **Resume sessions** → always brief agent about changes that happened during the pause
- **Sliding window** → risky; silently drops critical early information

---

## 🔧 Domain 3 — Tool Use & MCP (~25%)

### Tool Use Fundamentals
- **Claude requests**; **application executes** — Claude never runs tools directly
- Tool definitions consume **context window tokens** — complex schemas reduce document space
- **Tool use = guaranteed schema-compliant JSON**; prompt-only JSON = unreliable
- Claude selects tools based on their **descriptions** → write specific, distinct descriptions

### Tool Design
- **Enum** for fixed value sets; **string** for open-ended user input
- Mark all essential params in **`required` array** — optional params may be omitted
- Include "when NOT to use" in description when tools might be confused with each other

### Error Handling
| Error Type | Handle Where | How |
|---|---|---|
| **Transient** (503, timeout) | Inside tool | Retry with backoff; surface only if exhausted |
| **Permanent** (403, not found) | Surface to Claude | `isError: true` + errorCategory + isRetryable: false |
| **Ambiguous** (write timeout) | Surface to Claude | `isError: true` + isRetryable: false + warn against retry |

- ❌ Never throw exceptions from tool handlers
- ❌ Never return empty result with no error flag — Claude may fabricate a response

### MCP
- MCP = open standard (JSON-RPC) for connecting Claude to external tools and data
- **Tools** = actions (side effects possible); **Resources** = readable data (no side effects)
- Resources reduce exploratory tool calls by exposing content catalogs upfront
- **Protocol errors** = malformed JSON-RPC structure; **Tool errors** = business logic failures (isError: true)
- MCP tool descriptions must be as specific as built-in tools to win selection

### Safety Gates
- Business rules = **application layer enforcement**, NOT prompts alone
- **Two-step pattern**: preview (generates token) + execute (requires token)
- Confirmation tokens: scoped to specific action, time-limited, single-use
- Good confirmation dialogs: full content + target account + scheduled time + platform

---

## 📊 Domain 4 — Structured Output & Extraction (~20%)

### Tool Use for Extraction
- Always use **tool use** for extraction — not "please return JSON"
- Type fields as `["string", "null"]` for optional data; add null instructions
- **Null over fabrication** — "Return null if information is not stated in the source"
- **Enum + other + detail field** = standard pattern for unanticipated category values

### Quality Control
- **Confidence scores** (per field) catch semantic errors that schema validation misses
- **Route to human review**: low confidence OR ambiguous/conflicting source
- **Stratified sampling**: review random ~5% of high-confidence extractions too
- Calibrate thresholds quarterly — document types and failure patterns evolve

### Batch API
| | Batch API | Real-Time API |
|---|---|---|
| **Cost** | 50% savings | Standard |
| **Speed** | Up to 24 hours | Seconds |
| **Tool calls** | ❌ Not supported | ✅ Supported |
| **Use for** | High-volume, non-urgent | Interactive, urgent |

- **Batch window formula**: submit interval = SLA − 24h (e.g., 30h SLA → submit every 6h)
- Handle failures: resubmit only failed docs (chunked), not the whole batch

### Long Documents
- **Effective context** = 200K − system prompt − tool schema − conversation history
- As context fills → accuracy degrades for content toward the end of documents
- **Chunking**: split → extract per chunk → **merge and deduplicate** results
- **Chunk overlap**: include last N tokens of previous chunk at next chunk start
- Conflicting values in merge → flag for human review; record both with source locations

---

## 💻 Domain 5 — Claude Code & Workflows (~15%)

### Claude Code Tools

| Tool | Use When |
|---|---|
| **Grep** | Finding patterns across files (preferred over Bash search) |
| **Glob** | Finding files by name/extension pattern (no content read) |
| **Read** | Deep understanding of specific file content |
| **Edit** | Targeted surgical changes (requires unique anchor text) |
| **Write** | New files or when Edit cannot find unique anchor |
| **Bash** | When no specialised tool exists for the task |

### CLAUDE.md vs Skills

| | CLAUDE.md | Skills |
|---|---|---|
| **Loaded** | Automatically every session | Invoked on demand by slash command |
| **Use for** | Universal instructions, project context, standards | Multi-step workflows used occasionally |
| **Project-level** | Committed to repo, applies to all team members | Same |
| **User-level** | Personal file, applies to all projects for that user | Same |

- Nested CLAUDE.md: loaded only when working in that subdirectory — good for monorepos
- Keep CLAUDE.md focused → large CLAUDE.md wastes context every session

### Session Management
- `--continue` = most recent session; `--resume <name>` = by name; `--session-id` = by UUID
- Always **notify agent of changes** when resuming a paused session
- **Forking** = two independent branches from same checkpoint
- Fresh start: context completely outdated, fundamental misunderstandings, entirely new task

### Code Review
- **Independent review** = fresh session, code only, no generator intent context → avoids confirmation bias
- **Review chain** = sequential passes: security → style → documentation
- **Structured output schema** for reviews: severity + category + file + line + recommendation
- Batch review for non-urgent large queues; real-time for blocking PRs

### Exploration Patterns
- **Architecture-first**: orient → locate (Grep) → understand (Read) → plan → act
- **Plan Mode**: explore without changing; produce plan for user review; essential for large refactors
- Reading test files reveals expected behaviour (often better than reading implementation)
- CLAUDE.md with architecture overview: helps Claude target relevant files without broad exploration

---

## 🎯 Final Exam Tips

1. **Tool Use Domains (Ch03 + Ch04) = ~50% of the exam** — spend the most time here
2. **Confirmation tokens > dry_run boolean** — always the safer pattern for consequential actions
3. **Null over fabrication** — a fundamental principle across extraction and schema design
4. **Application layer > prompts** — for enforcing business rules and safety constraints
5. **Independent review > self-review** — always a fresh session for unbiased code review
6. **Batch API does NOT support tool calls during processing** — this eliminates it for iterative workflows
7. **Stratified sampling** — not just low-confidence extractions; sample high-confidence too
8. **Chunk overlap** — prevents losing boundary-spanning content in document chunking
9. **Structured escalation** — a handoff is not just the transcript; it includes diagnosis and next steps
10. **Architecture-first, Plan before Act** — the two most important Claude Code habits

---

*Quick Revision for certification-study-hub. Claude Architect Foundation.*
