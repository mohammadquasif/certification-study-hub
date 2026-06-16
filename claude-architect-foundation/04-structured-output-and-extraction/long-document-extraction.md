# Long Document Extraction — Chunking, Context Limits & Merging

> **Exam:** Claude Architect Foundation
> **Chapter:** 04 — Structured Output & Extraction
> **Difficulty:** Intermediate
> **Last Updated:** 2026-06

---

## 📖 What Is This Topic About?

Claude's context window is large — up to 200,000 tokens. But many documents are even larger, or documents share the context window with system prompts and tool definitions, leaving less room than you'd expect. This chapter covers how to handle documents that approach or exceed context limits: chunking, merging, and understanding how tool schemas and system prompts consume context.

---

## 🧠 Key Concepts

- **Context Length Exceeded** — An error returned when the total tokens in a request exceed the model's context window limit
- **Chunking** — Splitting a large document into smaller pieces and processing each chunk separately
- **Chunk Overlap** — Including a small amount of content from the end of one chunk at the beginning of the next, to preserve context across boundaries
- **Merging** — Combining extracted results from multiple chunks into a single, deduplicated output
- **Effective Context** — The actual tokens available for document content after accounting for system prompt, tool definitions, and conversation history
- **Context Pressure** — The degradation in extraction accuracy that occurs as the total tokens approach the context limit

---

## 📘 Detailed Explanation

### The Context Window is Shared — Not All for Documents

A common misconception: "Claude has a 200K context window, so I can process any document under 200K tokens." This is incorrect:

```
┌─────────────────────────────────── 200K CONTEXT WINDOW ─────────────────────────────┐
│                                                                                      │
│  ┌────────────────┐  ┌──────────────────────┐  ┌─────────────────────────────────┐ │
│  │ System Prompt  │  │   Tool Definitions    │  │  Conversation History           │ │
│  │ ~1,000 tokens  │  │  12-field schema:     │  │  (if multi-turn conversation)   │ │
│  │                │  │  ~2,500 tokens        │  │  ~500–5,000+ tokens             │ │
│  └────────────────┘  └──────────────────────┘  └─────────────────────────────────┘ │
│                                                                                      │
│  REMAINING FOR DOCUMENT:                                                             │
│  200,000 - 1,000 - 2,500 - 1,000 (conversation) = ~195,500 tokens                  │
│                                                                                      │
│  → A 195,000-token document fills 99.7% of remaining space                          │
│  → Very little room for Claude's response                                            │
│  → Accuracy on the last 10–20% of the document degrades significantly               │
│                                                                                      │
│  ┌────────────────────────────────────────────────────────────────────────────────┐ │
│  │                      DOCUMENT CONTENT (195,000 tokens)                         │ │
│  │  [Sections 1–8 processed well] [Section 9 degraded] [Section 10 often missed] │ │
│  └────────────────────────────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────────────────────────────┘
```

---

### Chunking Strategy

When a document cannot fit comfortably in the effective context, split it into chunks:

```
┌────────────────────────────────────────────────────────────────────┐
│                    CHUNKING STRATEGY                               │
│                                                                    │
│  LARGE DOCUMENT (500,000 tokens = ~375,000 words)                  │
│         │                                                          │
│    Split into chunks                                               │
│         │                                                          │
│    ┌────┴────────────────────────────────────────┐                │
│    │                                             │                │
│    ▼                                             ▼                │
│  CHUNK 1                                      CHUNK 2             │
│  Tokens 1–100K                                Tokens 95K–195K     │
│  (+ 5K overlap from previous)                 (+ 5K overlap)      │
│                                                                    │
│  Extract fields from Chunk 1    Extract fields from Chunk 2        │
│         │                                │                        │
│         └────────────────┬───────────────┘                        │
│                          │                                        │
│                          ▼                                        │
│              MERGE RESULTS                                         │
│              • Keep most confident value per field                 │
│              • Deduplicate list fields (citations, items)          │
│              • For conflicting values: flag for review             │
└────────────────────────────────────────────────────────────────────┘
```

**Why chunk overlap?** If a sentence spans the boundary between chunks — e.g., a payment term begins at the end of Chunk 1 and concludes at the start of Chunk 2 — neither chunk has the complete information. Including the last 1,000–2,000 tokens of Chunk 1 at the beginning of Chunk 2 ensures boundary-spanning content is captured in at least one chunk.

---

### Merging Chunk Results

After extracting from multiple chunks, results must be merged intelligently:

```
CHUNK 1 EXTRACTION:
{
  "contract_value": "₹25,00,000",
  "payment_terms": "Net 30",
  "parties": ["ABC Corp", "XYZ Ltd"],
  "effective_date": "2026-01-01"
}

CHUNK 2 EXTRACTION:
{
  "contract_value": null,  ← not found in chunk 2
  "payment_terms": "Net 30",  ← consistent, keep
  "parties": ["ABC Corp", "XYZ Ltd", "Global Services Inc."],  ← new party found
  "effective_date": "2026-01-01",  ← consistent, keep
  "termination_clause": "Either party may terminate with 60 days notice"
}

MERGED RESULT:
{
  "contract_value": "₹25,00,000",  ← from Chunk 1 (non-null wins)
  "payment_terms": "Net 30",       ← consistent across chunks
  "parties": ["ABC Corp", "XYZ Ltd", "Global Services Inc."],  ← union
  "effective_date": "2026-01-01",  ← consistent
  "termination_clause": "Either party may terminate with 60 days notice"
}
```

---

### Long Transcripts — Scatter-Gather Pattern

For long meeting transcripts where discussions revisit the same topic multiple times (information scattered throughout), a different strategy works better than naive chunking:

```
PROBLEM:
A 90-minute transcript discusses Q3 budget in:
• Turn 15 (initial proposal)
• Turn 42 (revision after break)
• Turn 78 (final decision)

NAIVE CHUNKING: Each chunk sees only one piece of the budget discussion
→ 68% accuracy (each chunk misses the full picture)

BETTER: SCATTER-GATHER
Step 1: Split transcript into chunks
Step 2: Extract topics and data points from each chunk independently
Step 3: Merge and deduplicate across chunks
         (Merge logic: later decisions overwrite earlier proposals for same topic)
Step 4: Output: unified, deduplicated extraction

RESULT: Higher accuracy because each chunk's extraction is partial,
         but the merge step assembles the complete picture
```

---

### Reducing Schema Token Footprint

If tool schema tokens are consuming too much context, reduce schema verbosity:

| Approach | Token Savings | Trade-off |
|---|---|---|
| Shorten field descriptions | Low | May reduce extraction accuracy on edge cases |
| Remove rarely needed fields | Medium | Lose those fields from extraction |
| Use a separate schema for different document sections | Medium | More complex pipeline |
| Summarise document before full extraction | High | Two API calls; adds latency and cost |

---

## 🇮🇳 Indian Real-Life Example

**Think of chunking like transcribing a long Lok Sabha session:**

A Lok Sabha session runs for 8 hours. One transcriber cannot process 8 hours of speech with full attention. Instead:
- Transcriber A handles Hours 1–3 (Chunk 1)
- Transcriber B handles Hours 2.5–5.5 (Chunk 2 — with overlap to catch boundary discussions)
- Transcriber C handles Hours 5–8 (Chunk 3 — with overlap)

An editor (merge step) then combines all three transcriptions, removes duplicates where overlaps created repetition, and resolves any conflicts where transcribers heard the same statement differently.

---

## 🔑 Exam-Focused Points

- ✅ Effective context = context window − system prompt − tool definitions − conversation history
- ✅ As total tokens approach the context limit, **accuracy degrades for content near the end**
- ✅ Tool definitions consume tokens — a 12-field schema with detailed descriptions can use 2,500+ tokens
- ✅ **Chunking** = split document, extract per chunk, **merge and deduplicate** results
- ✅ Chunk overlap preserves context at boundaries — prevents losing information that spans chunk edges
- ✅ For long transcripts with revisited topics: chunk → extract → merge (scatter-gather pattern)

---

## 🧩 Scenario-Based Thinking

**Scenario:** A processing pipeline achieves 98% extraction accuracy on documents under 150K tokens but only 71% accuracy on documents between 175K–190K tokens. The model's context window is 200K tokens. The tool schema totals approximately 2,500 tokens and the system prompt is 1,000 tokens.

**What is the most likely explanation?**

- A) The model processes shorter documents with a different algorithm than longer ones
- B) The schema has too many fields for large documents
- C) Tool definitions and system prompt consume approximately 3,500 tokens; combined with 190,000-token documents, the total approaches 200K, causing context pressure and degraded accuracy for content in the final sections
- D) The Batches API should be used for documents above 150K tokens

**Answer:** C

**Explanation:** At 190K tokens of document content + 3,500 tokens of system prompt and tool schema = 193,500 tokens total, very close to the 200K limit. Context pressure at this scale causes the model to attend poorly to content near the end of the document — explaining the consistent 71% accuracy with "information from the final sections consistently missed." The solution is to chunk documents above ~150K or reduce schema token footprint.

---

## 💡 Memory Tricks

**Chunk Overlap = Page Overlap in a Book:** When you scan a book in sections, you photograph the last page of the current section again at the start of the next section — to make sure you don't miss sentences that appear on the boundary.

**Effective Context = Net Salary:** Your gross context window is 200K tokens. System prompt, tools, and history are "deductions." Effective context is what's left for document content — the take-home.

---

## ❓ Chapter Practice Questions

**Q1.** An extraction system processes meeting transcripts. Accuracy is 94% for 30-minute transcripts but only 68% for 60-minute transcripts, where discussions revisit the same topic multiple times. Both transcript lengths fit within the context window. What approach most effectively improves accuracy on long transcripts?

- A) Increase the context window limit to process longer transcripts without degradation
- B) Use a different extraction schema for long transcripts
- C) Split long transcripts into chunks, extract separately from each chunk, then merge and deduplicate the results — capturing scattered information across all chunks
- D) Truncate transcripts at 30 minutes and discard later content

**Answer:** C

**Explanation:** The problem is not context length (both fit in the window) — it's that information about a single topic is scattered across the full transcript. Chunking with separate extraction and a merge-deduplicate step assembles the complete picture from scattered pieces. Simple single-pass extraction misses later occurrences of the same topic.

---

**Q2.** An extraction pipeline processes legal contracts averaging 150K tokens. The tool schema (10 fields with detailed descriptions) totals 3,000 tokens. The system prompt is 1,500 tokens. The context window is 200K tokens. For contracts between 185K–195K tokens, extraction accuracy drops for clauses in the final sections. What is the most effective solution?

- A) Reduce the number of fields in the extraction schema to lower token consumption
- B) Switch to a model with a larger context window
- C) Chunk contracts above 150K tokens into two sections with 5,000-token overlap, extract from each, and merge results
- D) Add prompt instructions to focus on the final sections of the contract

**Answer:** C

**Explanation:** At 190K document + 4,500 overhead = 194,500 tokens — very close to the 200K limit, causing accuracy degradation for late-appearing content. Chunking large contracts and merging results is the most reliable solution: each chunk has plenty of context headroom, ensuring thorough processing of all sections. Prompt instructions (D) don't address the fundamental context pressure issue.

---

**Q3.** When merging extraction results from multiple document chunks, the `payment_terms` field returns "Net 30" from Chunk 1 and "Net 45" from Chunk 2. How should the merge step handle this conflict?

- A) Take the value from the first chunk — it appeared first in the document
- B) Take the value from the last chunk — it appeared last and may be an amendment
- C) Flag the extraction for human review and record both values with their source chunk locations
- D) Return null for the field to indicate uncertainty

**Answer:** C

**Explanation:** Conflicting values across chunks may indicate a legitimate amendment (Net 30 → Net 45 in a later section) or an extraction error. The merge step should not silently discard one value. Flagging the conflict for human review with both values and their source locations preserves all information and surfaces the ambiguity appropriately. This is especially important for legal documents where the correct value has real consequences.

---

## 📌 Quick Revision Summary

- Effective context = 200K − system prompt − tool definitions − conversation history
- Context pressure near the limit: accuracy degrades for content toward the end of documents
- Tool schemas consume tokens — a 12-field detailed schema can use 2,500+ tokens
- Chunking: split document → extract per chunk → merge and deduplicate results
- Chunk overlap: include last N tokens of previous chunk at start of next, preserving boundary content
- Conflicting values in merge: flag for human review, record both values with source locations

---

## 📎 References

- [Anthropic Documentation — Long Context Best Practices](https://docs.anthropic.com/en/docs/long-context)

---

*Notes by certification-study-hub. Chapter 04 — Long Document Extraction.*
