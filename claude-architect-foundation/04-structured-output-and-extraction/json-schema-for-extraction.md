# JSON Schema for Extraction — Reliable Structured Data from Claude

> **Exam:** Claude Architect Foundation
> **Chapter:** 04 — Structured Output & Extraction
> **Difficulty:** Intermediate
> **Last Updated:** 2026-06

---

## 📖 What Is This Topic About?

One of the most common real-world uses of Claude is extracting structured data from unstructured text — pulling product details from listings, event metadata from articles, contract terms from legal documents. The key to reliable, schema-compliant extraction is **using tool use**, not just asking Claude to "return JSON." This chapter explains why, and how to design extraction schemas well.

---

## 🧠 Key Concepts

- **Structured Extraction** — Pulling specific fields from unstructured text and organising them into a defined schema
- **JSON Schema** — A standard for describing the structure, types, and constraints of a JSON object
- **Tool Use for Extraction** — Defining the desired output schema as a tool; Claude fills in the tool parameters during extraction
- **Schema Field** — One piece of data to be extracted (e.g., `product_name`, `price`, `date_published`)
- **Nullable Field** — A field that can hold `null` when the information is absent from the source document
- **Required Field** — A field that must always be present in the output (though it may be null)
- **Few-Shot Examples** — Example input-output pairs included in the prompt to demonstrate the desired extraction behaviour

---

## 📘 Detailed Explanation

### Why Tool Use for Extraction (Not Prompt-Only)

A common approach is to instruct Claude: *"Read this document and return a JSON object with these fields."* This fails unpredictably:

```
PROMPT-ONLY APPROACH (unreliable):
Instruction: "Extract event details and return as JSON"
Claude might return:
• Valid JSON ✅ sometimes
• JSON with extra text before/after ❌ sometimes  
• Slightly different field names than requested ❌ sometimes
• Free-form text explanation with no JSON ❌ rarely

TOOL USE APPROACH (reliable):
Define the extraction schema as a tool's input_schema
Claude must fill in the schema — it cannot return free text
• Always valid JSON ✅
• Always correct field names ✅
• Required fields always present ✅
• Type constraints enforced ✅
```

---

### Designing an Extraction Schema

Think of the extraction schema as a form that Claude will fill in. Design it with:

```json
{
  "name": "extract_event_metadata",
  "description": "Extract structured event information from the provided article text.",
  "input_schema": {
    "type": "object",
    "properties": {
      "event_name": {
        "type": "string",
        "description": "The name or title of the event as stated in the article"
      },
      "event_date": {
        "type": ["string", "null"],
        "description": "The date of the event in ISO 8601 format (YYYY-MM-DD). 
                        Return null if the date is not mentioned in the article."
      },
      "location": {
        "type": ["string", "null"],
        "description": "City, venue, or location of the event as stated. 
                        Return null if not mentioned."
      },
      "organiser": {
        "type": ["string", "null"],
        "description": "Name of the organising entity. Return null if not mentioned."
      },
      "attendee_count": {
        "type": ["integer", "null"],
        "description": "Number of attendees as stated in the article. 
                        Return null if not mentioned. Do NOT estimate."
      }
    },
    "required": ["event_name", "event_date", "location", "organiser", "attendee_count"]
  }
}
```

**Key design decisions:**
1. All fields are in `required` — Claude will always include them
2. Fields that may be absent are typed as `["string", "null"]` — Claude can return null when appropriate
3. Descriptions explicitly say "Return null if not mentioned" — preventing fabrication
4. `attendee_count` description says "Do NOT estimate" — reinforcing the null-over-fabrication principle

---

### Extraction vs Fabrication Problem

Without explicit null guidance, Claude may fabricate plausible values for fields not mentioned:

```
DOCUMENT: "The annual technology summit brought together leading innovators 
           from across the country. The keynote address was delivered by Dr. 
           Mehta from IISc Bangalore."

SCHEMA FIELD: attendee_count (integer)

WITHOUT GUIDANCE: Claude outputs 500 (plausible but fabricated)
WITH GUIDANCE ("return null if not mentioned"): Claude outputs null ✅
```

The solution is two-pronged:
1. Type the field as `["integer", "null"]`
2. Add explicit instruction: "Return null for any field where information is not directly stated in the source"

---

### Handling Varied Document Structures with Few-Shot Examples

When documents have inconsistent structures (inline citations vs references sections, methodology in introduction vs dedicated section), Claude may fail to locate information. Few-shot examples demonstrate how to extract from different layouts:

```
FEW-SHOT EXAMPLE IN EXTRACTION PROMPT:

EXAMPLE DOCUMENT A (inline citation style):
"According to Smith et al. [1], the results showed a 23% improvement..."

EXPECTED EXTRACTION:
citations: ["Smith et al. - referenced as [1]"]

EXAMPLE DOCUMENT B (references section style):
"The results showed a 23% improvement (see References section)...
References: [1] Smith, J. et al. (2024). 'Efficiency Gains in...' Journal of AI, 12(3)."

EXPECTED EXTRACTION:
citations: ["Smith, J. et al. (2024). 'Efficiency Gains in...' Journal of AI, 12(3)."]
```

Few-shot examples are particularly effective for showing Claude how to recognise the same information in different formats — something that abstract instructions ("find citations in any format") cannot reliably achieve.

---

### Schema Evolution — Adding Fields Over Time

Extraction schemas often need to evolve as new cases emerge. Best practices:

| Change Type | Risk | Recommendation |
|---|---|---|
| Add a new nullable field | Low | Safe to add — null for existing docs with no data |
| Add a new required non-null field | High | Will fail on all docs where info is absent |
| Change a type (string → integer) | Medium | Test on existing data before deploying |
| Remove a field | Low | Remove from schema; downstream code must be updated |

---

## 🇮🇳 Indian Real-Life Example

**Think of tool use extraction like filling out a standard government form:**

When you submit a GST filing, there's a specific form with defined fields — Trade Name, GSTIN, Turnover, Tax Liability. The form cannot be submitted with missing required fields or wrong data types (text in a number field).

Claude filling in an extraction schema is like an intelligent form-filler. Given a business document (invoice, contract, receipt), it identifies the relevant information and fills in the correct form fields. If information isn't in the document, it leaves the field blank (null) rather than making something up.

---

## 🔑 Exam-Focused Points

- ✅ Tool use for extraction guarantees **schema-compliant JSON** — prompt-only JSON instructions do not
- ✅ Type fields as `["string", "null"]` to allow null when information is absent
- ✅ Add explicit null instructions: "Return null if information is not stated in the source" — prevents fabrication
- ✅ Few-shot examples are the best tool for handling varied document structures
- ✅ All extracted fields should be in `required` — but nullable types allow null values
- ✅ Adding new nullable fields to a schema is low-risk; adding required non-null fields is high-risk

---

## 🧩 Scenario-Based Thinking

**Scenario:** An extraction pipeline occasionally receives responses that don't parse as valid JSON, causing downstream failures. The team is currently using prompt instructions to tell Claude to return JSON.

**What is the most reliable improvement?**

- A) Add more examples to the prompt showing correct JSON format
- B) Post-process Claude's response to extract JSON from mixed text
- C) Define the extraction schema as a tool's input_schema and use tool use — this guarantees schema-compliant JSON output
- D) Lower the model's temperature to reduce output variability

**Answer:** C

**Explanation:** Tool use with a defined input_schema structurally constrains Claude's output to valid, schema-compliant JSON. This works even when the document is complex or Claude's reasoning is verbose — the output is always the tool invocation, not free-form text. Prompt instructions (A, D) and post-processing (B) are workarounds that still fail unpredictably.

---

## 💡 Memory Tricks

**Tool Schema = Sarkaari Form:** The government form (schema) is fixed. You fill in what's asked. If a field doesn't apply, write N/A (null). You can't invent information that isn't in the supporting documents.

**Required ≠ Non-null:** A field can be required (always present in the output) AND nullable (can have a null value when the information is absent). Always mark fields as required; use null types for fields that might not appear in every document.

---

## ❓ Chapter Practice Questions

**Q1.** An extraction pipeline processes news articles using a JSON schema. The schema includes an `attendee_count` field (integer). During evaluation, the model frequently outputs plausible but incorrect numbers for this field even when the article doesn't mention attendance. What is the most effective fix?

- A) Change the field type to string so Claude can write "not mentioned"
- B) Remove the attendee_count field from the schema
- C) Change the type to `["integer", "null"]` and add an explicit instruction in the field description: "Return null if attendance information is not directly stated in the article. Do not estimate."
- D) Add a few-shot example showing the correct integer value for attendee_count

**Answer:** C

**Explanation:** The root cause is that Claude is fabricating plausible values rather than returning null. Two changes address this: (1) typing the field as nullable signals that null is a valid return value, and (2) the explicit instruction removes ambiguity about when to return null vs a number. Together, these prevent fabrication while preserving the ability to extract real attendance data when it is present.

---

**Q2.** An extraction system processes research papers with varied citation styles — some use inline references [1], others use footnotes, and others have a References section. The extraction schema has a `citations` field (array of strings). The model frequently returns empty arrays for papers that do contain citations. What is the most effective solution?

- A) Add a description to the citations field listing all possible citation styles
- B) Add few-shot examples demonstrating how to extract citations from each citation style — showing the expected output for inline, footnote, and references-section formats
- C) Create three separate extraction tools, one for each citation style
- D) Route papers to different prompts based on detected citation style

**Answer:** B

**Explanation:** The challenge is recognising the same type of information across different formats. Abstract descriptions ("find citations in any format") are less effective than concrete examples. Few-shot examples showing the actual extraction for each style demonstrate to Claude exactly where to look and what to extract — this is more powerful than instructions alone.

---

**Q3.** An extraction schema currently has 8 fields. The team wants to add a new `compliance_flags` field (array of strings) to flag regulatory concerns in documents. Which statement correctly describes the risk of this addition?

- A) Adding a new array field is high-risk because it changes the schema version
- B) Adding a new nullable field (`["array", "null"]`) is low-risk — documents without compliance concerns will return null, and existing processing logic handles null arrays gracefully
- C) No fields should be added to a deployed schema — create a new schema instead
- D) Adding any new field requires re-processing all historical documents

**Answer:** B

**Explanation:** Adding a nullable field to an existing schema is low-risk. Documents that have no compliance concerns return null for that field. Documents that do have concerns return the extracted flags. Existing documents don't need re-processing (unless you want their compliance analysis), and downstream code can handle null gracefully. The high-risk change would be adding a required, non-null field to documents where the data might not exist.

---

## 📌 Quick Revision Summary

- Use tool use for extraction — guarantees schema-compliant JSON; prompt-only does not
- Type fields as `["string", "null"]` for fields that may be absent in some documents
- Explicit null instructions: "Return null if not stated in source" prevents fabrication
- Few-shot examples are the best solution for varied document structures
- All fields should be in `required` — but nullable types allow null when data is absent
- Adding nullable fields = low risk; adding required non-null fields = high risk

---

## 📎 References

- [Anthropic Documentation — Structured Outputs](https://docs.anthropic.com/en/docs/structured-outputs)
- [Anthropic Documentation — Tool Use for Extraction](https://docs.anthropic.com/en/docs/tool-use)

---

*Notes by certification-study-hub. Chapter 04 — JSON Schema for Extraction.*
