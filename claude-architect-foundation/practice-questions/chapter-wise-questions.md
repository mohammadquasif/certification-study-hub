# Chapter-Wise Practice Questions — Claude Architect Foundation

> **Exam:** Claude Architect Foundation
> **Format:** 5–8 original scenario-based questions per chapter
> **Last Updated:** 2026-06

---

## 📚 Chapter 01 — Claude Fundamentals

**Q1.** A developer builds a customer support product using Claude. Stakeholders are concerned that Claude might provide overconfident answers about technical issues it isn't certain about, misleading customers. Which core Claude principle most directly addresses this concern?

- A) Helpful — Claude is designed to be genuinely useful
- B) Harmless — Claude avoids causing psychological harm
- C) Honest — Claude is trained to express genuine uncertainty rather than provide false confidence
- D) Constitutional — Claude self-critiques before answering

**Answer:** C | **Category:** HHH Principles

---

**Q2.** A team is debating whether their 190,000-token legal document can be processed by Claude in a single API call. Their tool schema is 4,000 tokens and their system prompt is 1,500 tokens. Claude's context window is 200,000 tokens. Is a single-call extraction reliable?

- A) Yes — the document is under 200K tokens
- B) No — the combined total of 195,500 tokens leaves only ~4,500 tokens for Claude's response, causing accuracy degradation especially for content in the final sections
- C) Yes — context window limits only apply to the response, not the input
- D) No — tool schemas cannot be used with documents exceeding 150,000 tokens

**Answer:** B | **Category:** Context Window

---

**Q3.** A fintech startup wants to use Claude to automatically approve loan applications. The system would operate with no human review for loans under ₹5 lakh. What is the primary responsible deployment concern?

- A) Claude cannot access financial data
- B) The context window is insufficient for loan applications
- C) Automated financial decisions of this scale, with no human oversight, is a responsible deployment failure — errors can cause significant financial and legal harm
- D) Claude's honesty principle prevents it from making approval decisions

**Answer:** C | **Category:** Responsible Deployment

---

**Q4.** Which statement about Claude's stateless nature is correct for developers to understand?

- A) Claude automatically stores conversation history across sessions in the cloud
- B) Each API call is independent; the application must explicitly send previous conversation turns with every request to maintain context
- C) Claude's stateless nature means it cannot use tools
- D) Stateless means Claude forgets tool definitions between tool calls in the same session

**Answer:** B | **Category:** Stateless Architecture

---

## 📚 Chapter 02 — Agentic Patterns

**Q5.** An HR agent receives a request to remove a team member from a project management system. The agent verifies the request but the member has 3 active projects currently assigned. What should the agent do?

- A) Proceed with removal immediately since the request was verified
- B) Decline the removal and inform the user that active projects prevent it
- C) Present the impact (3 active projects affected) and require explicit human confirmation before proceeding
- D) Reassign the 3 projects automatically before removing the member

**Answer:** C | **Category:** Human-in-the-Loop

---

**Q6.** A content pipeline processes blog posts. Each post must be: (1) translated from Hindi to English, (2) checked for factual accuracy, (3) optimised for SEO. The three steps always happen in this order for every post. Which pattern is most appropriate?

- A) Orchestrator-workers — three parallel workers for translation, accuracy, and SEO
- B) Routing — classify the post type and send to appropriate specialist
- C) Prompt chaining — three sequential steps, each using the prior output
- D) Single-pass — do all three in one comprehensive prompt

**Answer:** C | **Category:** Agentic Patterns

---

**Q7.** A coding agent spent 3 hours building context about a complex microservices architecture. The developer wants to explore refactoring Service A using both Approach X and Approach Y, then choose the better option. What is the most effective approach?

- A) Have the agent explore Approach X fully, then continue with Approach Y in the same session
- B) Fork the session at the current checkpoint; explore Approach X in Branch A and Approach Y in Branch B
- C) Start two fresh sessions and brief each on the architecture
- D) Have the agent choose the better approach based on its analysis without exploring both

**Answer:** B | **Category:** Session Forking

---

**Q8.** An orchestrator agent delegates subtasks to three worker agents. What is the key advantage of worker agents having isolated context windows?

- A) Isolated contexts reduce overall token costs
- B) Workers can run faster with less context
- C) Each worker focuses on its specific task without being influenced by other workers' reasoning or findings
- D) Isolation prevents workers from making conflicting tool calls

**Answer:** C | **Category:** Multi-Agent Context Isolation

---

## 📚 Chapter 03 — Tool Use & MCP

**Q9.** A tool description reads: "Gets data." During testing, Claude calls this tool for unrelated purposes. Which intervention most directly improves tool selection accuracy?

- A) Reduce the number of available tools
- B) Rewrite the description to specify: what data it retrieves, what format it returns, and when to use it vs other tools
- C) Add more parameters to the tool to make its purpose clearer
- D) Move the tool to an MCP server

**Answer:** B | **Category:** Tool Design

---

**Q10.** A payment tool returns a timeout error after 30 seconds without receiving confirmation that the payment was processed. What should the tool return?

- A) An `isError: false` response indicating the payment is likely successful
- B) An `isError: true` response with `isRetryable: true` to trigger automatic retry
- C) An `isError: true` response noting the uncertain state and recommending against automatic retry to prevent duplicate charges
- D) An empty response to let Claude infer from context

**Answer:** C | **Category:** Error Handling

---

**Q11.** A developer tests an extraction pipeline and finds Claude calls `search_documents` instead of `get_document_content` for retrieving a specific document's text. Both tools are defined. What is the most effective fix?

- A) Remove `search_documents` when retrieving specific documents is needed
- B) Update `get_document_content`'s description to clarify it retrieves full text for a specific known document ID, and update `search_documents` to clarify it is for query-based discovery of unknown documents
- C) Merge both tools into one unified document tool
- D) Add a routing step that decides which tool to call before Claude does

**Answer:** B | **Category:** Tool Description Clarity

---

**Q12.** Which of the following is correctly handled as a Protocol Error (JSON-RPC level) rather than a Tool Error (isError: true)?

- A) A user record not found in the database
- B) The calendar service returning 503 during scheduled downtime
- C) A request missing the required `jsonrpc` field in the MCP message structure
- D) Insufficient permissions to access a restricted document

**Answer:** C | **Category:** MCP Error Handling

---

**Q13.** An `archive_document` tool is redesigned with two steps: `preview_archive(document_id)` returning impact details and a confirmation token, and `execute_archive(document_id, confirmation_token)`. Why is this more reliable than a single tool with a `dry_run: false` parameter?

- A) Two-step tools are faster
- B) A confirmation token can only be obtained by completing the preview step; it is scoped to the specific action and expires — making it impossible for the agent to bypass confirmation, unlike a boolean which can be set to false directly
- C) Two-step tools reduce token consumption
- D) Confirmation tokens provide cryptographic security

**Answer:** B | **Category:** Safety Gates

---

## 📚 Chapter 04 — Structured Output & Extraction

**Q14.** An extraction pipeline uses prompt instructions to have Claude return JSON. Downstream parsing fails 8% of the time due to malformed JSON. What is the most reliable architectural fix?

- A) Add a JSON validation and retry step after extraction
- B) Define the extraction schema as a tool's input_schema and use tool use — guaranteeing schema-compliant JSON output
- C) Increase the output temperature to reduce randomness
- D) Add a post-processing step to extract JSON from mixed text

**Answer:** B | **Category:** Tool Use for Extraction

---

**Q15.** A review extraction schema has `overall_sentiment` as an enum (positive, negative, mixed). Evaluations show mixed accuracy for sarcastic reviews like "Exactly what I was looking for if I wanted a product that expires in a week." What schema improvement is most appropriate?

- A) Remove sentiment from the schema — it is subjective
- B) Change sentiment to a free-form string field
- C) Add `unclear` to the enum and include a field description explaining: "Use 'unclear' for reviews where sentiment cannot be determined due to irony, sarcasm, or contradictory signals"
- D) Add few-shot examples of sarcastic reviews

**Answer:** C | **Category:** Ambiguity Handling

---

**Q16.** A pipeline processes 8,000 financial documents daily using the Message Batches API. The results file shows 240 failures with `context_length_exceeded`. What is the most cost-effective recovery strategy?

- A) Resubmit all 8,000 documents with smaller prompts
- B) Skip the failures — 3% error rate is acceptable
- C) Identify only the 240 failed documents, chunk each into smaller sections, and resubmit as a new batch; merge partial extractions
- D) Process the 240 failures using real-time API immediately

**Answer:** C | **Category:** Batch Processing

---

**Q17.** An extraction pipeline achieves 97% field accuracy across 10,000 daily extractions. Sampling reveals that 9% of high-confidence (≥0.90) extractions still contain semantic errors. What quality control strategy best addresses this?

- A) Lower the routing threshold from 0.90 to 0.70 so more extractions are reviewed
- B) Rely on the 97% overall accuracy — high enough for production
- C) Implement stratified random sampling: route all low-confidence extractions to review PLUS a random 5% of high-confidence extractions weekly for error rate monitoring
- D) Add confidence scoring to every field and route any extraction with one low-confidence field to review

**Answer:** C | **Category:** Confidence & Sampling

---

## 📚 Chapter 05 — Claude Code & Developer Workflows

**Q18.** Claude Code is tasked with finding all files in a React codebase that import a deprecated hook `useOldTheme`. What is the most efficient approach?

- A) Read every .tsx and .ts file in the project
- B) Use Glob to list all TypeScript files, then read each one to find the import
- C) Use Grep to search for the pattern `useOldTheme` across all .tsx and .ts files
- D) Run a Bash `find` command to search for the hook

**Answer:** C | **Category:** Claude Code Tools

---

**Q19.** A developer asks Claude Code to do all three of the following at once: refactor the authentication module, update the API documentation, and add test coverage for the new auth flow. After 40 minutes, Claude has made many changes but tests fail and documentation is incomplete. What workflow would have produced better results?

- A) Use a faster Claude model
- B) Enter Plan Mode first: explore all affected files, document a comprehensive plan covering the interdependencies, get user approval, then execute in a controlled order
- C) Give Claude more time to complete all three tasks
- D) Separate each task into different sessions on different days

**Answer:** B | **Category:** Plan Mode

---

**Q20.** A team wants team-wide standards applied every session (test patterns, naming conventions) AND a rarely-used security scan workflow. Which configuration is most appropriate?

- A) Add both to the project CLAUDE.md
- B) Add team-wide standards to the project CLAUDE.md; define the security scan as a Skill (`/security-scan`) invoked when needed
- C) Add both as Skill definitions to keep CLAUDE.md lightweight
- D) Add both to individual user CLAUDE.md files for each developer

**Answer:** B | **Category:** CLAUDE.md vs Skills

---

**Q21.** A developer resumes a Claude Code session that was paused for 36 hours. During the pause, a colleague merged a PR that renamed two core services and moved them to a new directory structure. What must the developer do when resuming?

- A) Start a fresh session — the codebase change invalidates the prior session's knowledge
- B) Resume the session and explicitly brief Claude on the renamed services and new directory structure before continuing any investigation
- C) Resume the session and let Claude rediscover the changes through its own exploration
- D) Ask the colleague to revert the changes before resuming

**Answer:** B | **Category:** Session Management

---

**Q22.** A code generator Claude Code session produces a data pipeline. The developer asks the SAME session to review the pipeline for performance bottlenecks. Three months later, a performance audit reveals issues the review missed. What is the most likely root cause?

- A) Performance review requires a dedicated performance analysis model
- B) The review was performed in the same session as the generation; the reviewer had context about the generator's design decisions, causing confirmation bias that led to missing bottlenecks inherent to those decisions
- C) Performance reviews should be done manually, not by Claude Code
- D) The code was too long for Claude to review accurately

**Answer:** B | **Category:** Independent Review

---

**Q23.** Claude Code is exploring a large monorepo to understand how authentication is implemented before making changes. It starts reading files in the `/auth` directory but also reads many frontend and infrastructure files. What exploration approach would have been more efficient?

- A) Read only the /auth directory
- B) Start with architecture orientation: list top directories, read CLAUDE.md, Grep for the main auth class/function, then read only the files that import or use it — focusing the exploration before reading deeply
- C) Use Glob to find all auth-related files and read all of them
- D) Ask the developer to provide all relevant file paths before starting

**Answer:** B | **Category:** Exploration Patterns

---

## 📚 Cross-Domain Questions

**Q24.** A conversation with Claude spans 45 turns covering 3 separate customer issues. As the conversation approaches turn 46, a support agent reports that Claude keeps "forgetting" key details from Turns 1–5. What is the best resolution?

- A) Restart the conversation — context memory is too short for long support sessions
- B) Extract and store key facts from resolved issues (order IDs, issue types, resolutions) in an external data store; retrieve them via tools when referenced in later turns
- C) Increase the context window for this agent's conversation
- D) Keep all raw turns and rely on Claude to re-read earlier content as needed

**Answer:** B | **Category:** Context Management

---

**Q25.** An architect is designing a document processing pipeline. Documents arrive continuously, 80% are standard quarterly reports (no urgency), 20% are urgent regulatory filings (needed within 2 hours). Which architecture best balances cost and SLA compliance?

- A) Process all documents through real-time API for reliability
- B) Process all documents through batch API for cost savings
- C) Route standard reports to the Batch API (50% cost savings, results within 24h) and urgent filings to the real-time API (immediate processing)
- D) Process all documents sequentially through a single worker

**Answer:** C | **Category:** Cross-Domain — Batch Routing

---

*All questions are original scenarios created for certification-study-hub. Not sourced from any official exam materials.*
