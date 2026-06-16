# Handling Ambiguity & Nulls — Designing Robust Extraction Schemas

> **Exam:** Claude Architect Foundation
> **Chapter:** 04 — Structured Output & Extraction
> **Difficulty:** Intermediate
> **Last Updated:** 2026-06

---

## 📖 What Is This Topic About?

Real-world documents are messy. Values contradict each other. Fields don't fit neatly into fixed categories. New types emerge that weren't anticipated when the schema was designed. This chapter covers how to design schemas that handle ambiguity gracefully — without fabricating values, without breaking on unexpected inputs, and without requiring constant schema rewrites.

---

## 🧠 Key Concepts

- **Null Field** — A field that explicitly returns `null` when the information is absent, ambiguous, or unclear
- **Enum with Other** — An enum that includes an `other` value plus a companion detail field for capturing unrecognised cases
- **Conflicting Values** — When the same piece of information appears differently in multiple places in the same document
- **Source Attribution** — Recording where in the document a value came from (section, page, clause number)
- **Schema Versioning** — Managing changes to extraction schemas over time without breaking existing pipelines
- **Extraction Instruction** — Text in the prompt or field description that guides Claude on how to handle edge cases

---

## 📘 Detailed Explanation

### Pattern 1: Null Over Fabrication

When information is absent, Claude should return `null` — not guess, not estimate, not paraphrase adjacent content.

```
DOCUMENT: "The property at 14 MG Road features open floor plans and 
            high ceilings with large windows."

SCHEMA FIELD: property_type (enum: house, apartment, condo, townhouse)
DOCUMENT MENTIONS: "property" — no specific type given

WITHOUT NULL GUIDANCE: Claude picks "apartment" (a guess based on "MG Road")
WITH GUIDANCE ("return null if not explicitly stated"): Claude returns null ✅

BETTER STILL: Allow null in the enum type definition
"property_type": {
  "type": ["string", "null"],
  "enum": ["house", "apartment", "condo", "townhouse", null],
  "description": "Property type as stated in the listing. Return null if 
                  the type is not explicitly identified."
}
```

---

### Pattern 2: Enum + Other + Detail Field

Fixed enums break when real-world data exceeds anticipated categories. The solution: add `other` to the enum and a companion detail field.

```
ORIGINAL SCHEMA (breaks on new types):
"property_type": {
  "type": "string",
  "enum": ["house", "apartment", "condo", "townhouse"]
}
→ Fails when listing says "studio", "loft", "duplex", "warehouse"

IMPROVED SCHEMA (handles new types gracefully):
"property_type": {
  "type": "string",
  "enum": ["house", "apartment", "condo", "townhouse", "other"],
  "description": "Property type. Use 'other' for types not in this list."
},
"property_type_detail": {
  "type": ["string", "null"],
  "description": "If property_type is 'other', specify the actual type here 
                  (e.g., 'studio', 'loft', 'warehouse'). Null if property_type 
                  is one of the standard values."
}
```

This pattern:
- Maintains a clean, fixed enum for filtering and analytics
- Captures the actual value for audit and future schema expansion
- Never fails on unexpected input
- Allows the team to identify which new types appear most often and decide whether to add them to the enum

---

### Pattern 3: Handling Sentiment and Ambiguous Classifications

For fields that classify qualitative content (sentiment, tone, category), some inputs are genuinely ambiguous:

```
REVIEW: "Great product!"
→ pros: ["great product"] ← Claude fabricates specifics for a vague review
→ Fix: Allow null for pros/cons

REVIEW: "Well that was... interesting"
→ Sarcasm — sentiment could be negative or neutral
→ Fix: Add "unclear" to the sentiment enum

IMPROVED SCHEMA FOR SENTIMENT:
"overall_sentiment": {
  "type": "string",
  "enum": ["positive", "negative", "mixed", "unclear"],
  "description": "Overall sentiment. Use 'unclear' for sarcastic, ironic, or 
                  genuinely ambiguous reviews where sentiment cannot be determined."
},
"pros": {
  "type": ["array", "null"],
  "items": {"type": "string"},
  "description": "Specific positive points. Return null (not empty array) 
                  if the review is too brief or vague to identify specific pros."
},
"cons": {
  "type": ["array", "null"],
  "items": {"type": "string"},
  "description": "Specific negative points. Return null if not identifiable."
}
```

---

### Pattern 4: Handling Conflicting Values in the Same Document

When a document contains contradictory information (original terms + amendments, summary vs detailed specifications table), a single-field schema hides the conflict.

```
DOCUMENT: 
Summary section: "Payment terms: Net 30 days"
Amendment 1 (signed 3 months later): "Payment terms revised to Net 45 days"

SINGLE-FIELD SCHEMA (loses information):
"payment_terms": "Net 30 days"  ← or "Net 45 days" — model picks inconsistently

MULTI-VALUE SCHEMA (preserves all information):
"payment_terms": [
  {
    "value": "Net 30 days",
    "source_section": "Original Contract, Section 4.2",
    "effective_date": "2025-01-01",
    "is_current": false
  },
  {
    "value": "Net 45 days",
    "source_section": "Amendment 1, Clause 2",
    "effective_date": "2025-04-01",
    "is_current": true
  }
]
```

This approach:
- Captures ALL values, not just one
- Preserves source location for audit
- Records effective dates for temporal accuracy
- Marks which version is currently in effect

---

### Pattern 5: Instruction-Based Conflict Resolution

When multiple values exist and one source is consistently more reliable, use extraction instructions to guide Claude — without changing the schema:

```
SCENARIO: Product specifications appear in both a summary section and a 
           detailed specifications table. The table is correct 90% of the time.

EXTRACTION INSTRUCTION (in system prompt or field description):
"When the same value appears in multiple sections, prefer values from the 
 detailed specifications table over values from summary sections or 
 introductory paragraphs. Extract the table value and proceed."

RESULT: 
• Schema stays simple (single battery_capacity_mAh field)
• Extraction accuracy improves by following source priority rules
• No schema redesign required
```

---

## 🇮🇳 Indian Real-Life Example

**Think of ambiguity handling like a legal interpreter in a court case:**

A legal interpreter doesn't make up testimony when something is unclear — they say "the witness's statement was unclear on this point." Similarly:
- When the document doesn't say → return null (don't fabricate)
- When the document says something unexpected → use the `other` category and record what it actually said
- When the document contradicts itself → record all versions with their source and date

A good interpreter is precise about uncertainty; a bad interpreter guesses and states guesses as facts.

---

## 🔑 Exam-Focused Points

- ✅ Return `null` when information is absent — never fabricate or guess
- ✅ **Enum + other + detail field** = the standard pattern for handling unanticipated enum values
- ✅ Add `"unclear"` (or equivalent) to classification enums for genuinely ambiguous cases
- ✅ Allow `null` for array fields when the source is too vague to extract specific items
- ✅ Multi-value fields with source attribution = correct approach for documents with amendments/conflicts
- ✅ Extraction instructions (in prompt/description) can resolve conflicts without schema changes when one source is consistently more reliable

---

## 🧩 Scenario-Based Thinking

**Scenario:** An extraction pipeline processes product reviews. The schema has `overall_sentiment` as an enum (positive, negative, mixed). During testing, sarcastic reviews like "Just what I needed. Another product that breaks in week one." cause inconsistent sentiment classifications. What schema modification best addresses this?

- A) Remove the sentiment field — sarcasm makes it unreliable
- B) Change sentiment to a free-form string field to capture nuance
- C) Add an `unclear` value to the sentiment enum for genuinely ambiguous or ironic reviews, with a field description explaining when to use it
- D) Add few-shot examples of sarcastic reviews only

**Answer:** C

**Explanation:** Adding `unclear` to the enum acknowledges that some reviews are genuinely ambiguous. This is more reliable than a free-form string (which defeats the purpose of a structured schema) or removing the field entirely. Few-shot examples (D) help but the primary fix is schema-level — the model needs a valid value to select for ambiguous cases.

---

## 💡 Memory Tricks

**Null = Honest About Absence:** Returning null says "this information is not in the document." Returning a guessed value says "I made something up." One is honest; one causes data quality problems downstream.

**other + detail = Catch-All Bucket:** The `other` value is the catch-all bucket. You still catch everything — you just label it honestly as "other" and write what it actually was in the detail field. Future you can then decide whether "studio" is common enough to add to the main enum.

---

## ❓ Chapter Practice Questions

**Q1.** An extraction schema has `property_type` as an enum with four values. New listings keep mentioning property types not in the enum ("studio", "loft", "co-living"), causing validation failures. What is the most sustainable long-term fix?

- A) Add all new types to the enum whenever they appear
- B) Switch property_type to a free-form string field to accept any value
- C) Add `other` to the enum and a companion `property_type_detail` string field that captures the actual type when `other` is selected
- D) Return null for any property type not in the current enum

**Answer:** C

**Explanation:** Adding `other` + detail is the sustainable pattern. It captures all values without breaking validation, preserves the specific type for audit and analytics, and allows the team to periodically review which "other" types appear most frequently and decide whether to add them to the main enum. Returning null (D) loses valuable information. A free-form string (B) defeats the schema's type safety.

---

**Q2.** A contract extraction pipeline processes documents that frequently contain amendments. A single `payment_terms` field returns either the original or amended value inconsistently. What redesign best addresses this?

- A) Add a prompt instruction telling Claude to always prefer the most recent value
- B) Keep the single field but add a `payment_terms_note` text field for amendments
- C) Redesign the field to capture multiple values, each with source section and effective date, marking the current version
- D) Process the contract and its amendments as separate documents

**Answer:** C

**Explanation:** When a document legitimately contains multiple versions of the same data, a single field cannot represent the truth accurately. A multi-value structure with source section and effective date captures all versions, preserves audit trail information, and allows downstream systems to identify which version is current. Prompt instructions (A) help only if amendments are consistently placed — which is not guaranteed.

---

**Q3.** An extraction schema for product reviews has `pros` as a required array of strings. For very brief reviews like "Loved it!", the model fabricates specific pros (e.g., ["durable", "good value", "easy to use"]). What schema change best addresses this without losing pro extraction for detailed reviews?

- A) Remove the `pros` field from the schema
- B) Change `pros` from required array to `["array", "null"]` with a description instructing Claude to return null (not empty array) when the review is too brief to identify specific positive points
- C) Change `pros` to a single string field instead of an array
- D) Add a minimum length check on the review text before extracting pros

**Answer:** B

**Explanation:** Changing the type to `["array", "null"]` with null instructions addresses both cases: detailed reviews with specific pros get an array of extracted items; brief or vague reviews return null. This is more honest and useful than fabricated specifics (original behaviour) or always returning an empty array (which could mean "no pros" vs "too vague to determine pros").

---

## 📌 Quick Revision Summary

- Null > fabrication: always return null when information is absent from the source
- Enum + other + detail field: handle unexpected category values without breaking validation
- Add "unclear" to classification enums for genuinely ambiguous cases (sarcasm, irony, mixed signals)
- Allow null for array fields when source is too vague to extract specific items
- Multi-value fields with source attribution: correct pattern for documents with amendments
- Extraction instructions can resolve source conflicts without schema changes when one source is more reliable

---

## 📎 References

- [Anthropic Documentation — Structured Outputs](https://docs.anthropic.com/en/docs/structured-outputs)

---

*Notes by certification-study-hub. Chapter 04 — Handling Ambiguity & Nulls.*
