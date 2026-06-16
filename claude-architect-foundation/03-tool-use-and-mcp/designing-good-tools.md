# Designing Good Tools — Parameters, Descriptions & Schemas

> **Exam:** Claude Architect Foundation
> **Chapter:** 03 — Tool Use & MCP
> **Difficulty:** Intermediate
> **Last Updated:** 2026-06

---

## 📖 What Is This Topic About?

How you design your tools directly determines whether Claude uses them correctly. A well-designed tool has a clear description, well-named parameters with helpful descriptions, and an appropriate schema. Poor tool design leads to Claude selecting the wrong tool, passing incorrect parameters, or calling tools in the wrong order.

---

## 🧠 Key Concepts

- **Tool Description** — The text that explains to Claude what the tool does; this is Claude's primary guide for tool selection
- **Parameter Name** — The identifier Claude uses to pass a value into the tool; should be descriptive and unambiguous
- **Parameter Type** — The data type of the parameter (string, integer, boolean, array, object, enum)
- **Enum** — A parameter that only accepts one of a fixed list of allowed values (e.g., "research_papers", "internal_reports")
- **Required vs Optional** — Required parameters must always be provided; optional parameters have defaults or are conditionally needed
- **Schema Strictness** — Using `required`, `type`, and other JSON Schema constraints to prevent Claude from passing invalid values

---

## 📘 Detailed Explanation

### The Tool Description — Most Important Element

Claude selects tools based on their descriptions. A vague description leads to wrong selection:

```
❌ BAD DESCRIPTION:
"name": "query_documents"
"description": "Queries documents"

→ Claude doesn't know WHICH documents, WHAT kind of query, or HOW results are formatted

✅ GOOD DESCRIPTION:
"name": "search_research_papers"
"description": "Searches the research papers database for academic content. 
Returns matching paper titles, authors, abstracts, and DOI links.
Use this for queries about published research findings, not for internal reports."

→ Claude knows exactly what it does, what it returns, and when to use it vs other tools
```

**Key rules for descriptions:**
1. State what the tool does (action + object)
2. State what it returns (output format)
3. Clarify when to use it — especially when another tool might be confused with it
4. State when NOT to use it if disambiguation is important

---

### Parameter Design — Enum vs String

Choosing between an enum and a free-form string depends on whether the set of valid values is finite and known:

| Parameter Type | Use When | Example |
|---|---|---|
| **Enum** | Fixed, known set of values | Database selection: `["research_papers", "internal_reports", "technical_specs"]` |
| **String** | Open-ended, user-provided text | Search query: `"machine learning transformer architecture"` |
| **Integer** | Numeric counts or IDs | `page_size: 10`, `order_id: 4921` |
| **Boolean** | On/off flags | `include_archived: false` |
| **Array of strings** | Multiple values from open set | `["python", "django", "postgresql"]` |
| **Object** | Grouped related fields | `{"start_date": "2026-01-01", "end_date": "2026-06-01"}` |

**The Enum Advantage:** When users express intent in natural language ("search the research database", "check technical documents"), Claude maps the natural language to the correct enum value. This mapping is reliable because the model understands intent. You get structured, validated values without parsing logic.

---

### Required vs Optional Parameters

Always mark parameters as required when the tool cannot function without them:

```json
{
  "name": "update_user_profile",
  "input_schema": {
    "type": "object",
    "properties": {
      "user_id": {
        "type": "string",
        "description": "The unique identifier of the user to update. Required."
      },
      "fields_to_update": {
        "type": "object",
        "description": "An object containing the profile fields to update. 
                        Each key is a field name, each value is the new value.",
        "properties": {
          "email": { "type": "string" },
          "display_name": { "type": "string" },
          "timezone": { "type": "string" }
        }
      }
    },
    "required": ["user_id"]
  }
}
```

When `user_id` is not in `required`, Claude may omit it — causing the tool to fail or update the wrong record. **Always list required parameters in the `required` array.**

---

### Avoiding Common Parameter Mistakes

```
MISTAKE 1: Ambiguous parameter names
❌ "id" → Which ID? Order? Customer? Product?
✅ "order_id", "customer_id", "product_sku"

MISTAKE 2: Missing parameter descriptions
❌ "start_date": { "type": "string" }
✅ "start_date": { "type": "string", 
                   "description": "Start date in ISO 8601 format (YYYY-MM-DD). 
                                   Example: '2026-01-01'" }

MISTAKE 3: Overlapping tool purposes
❌ get_document_info and get_document_details (Claude can't tell them apart)
✅ get_document_metadata (returns title, author, date, file size) 
   get_document_content (returns full text content of the document)

MISTAKE 4: Too many tools with similar names
❌ search_v1, search_v2, search_advanced, search_basic
✅ Consolidate into one well-designed search tool with optional parameters
```

---

### Tool Decomposition — When to Split vs Consolidate

```
SPLIT when:
• Tools have fundamentally different purposes
• Different tools require different permissions
• You want Claude to be explicit about which action it's taking

CONSOLIDATE when:
• Two tools always get called together (unnecessary round-trip)
• One tool's output is always the input to another tool
• Splitting creates unnecessary latency and failure coupling

EXAMPLE — Unnecessary Coupling (should consolidate):
get_property_details(property_id) → returns address
get_neighborhood_info(address) → requires address from above tool

FIX: Change get_neighborhood_info to accept property_id directly
→ Eliminates the forced sequential call, reduces failure points
```

---

## 🇮🇳 Indian Real-Life Example

**Think of tool design like designing forms at a government office:**

A well-designed form (tool) has:
- A clear title explaining what it's for (tool description)
- Fields with clear labels, not just "ID" but "Aadhaar Number" (parameter names)
- Dropdown options where applicable, not blank text fields (enums)
- Asterisks marking required fields so nothing important is left blank (required schema)
- Instructions clarifying what format to use ("Date as DD/MM/YYYY") (parameter descriptions)

A poorly designed form confuses applicants and gets filled incorrectly — just like a poorly designed tool confuses Claude and causes incorrect parameter values.

---

## 🔑 Exam-Focused Points

- ✅ Claude selects tools **primarily based on the tool description** — write clear, specific, distinct descriptions
- ✅ **Enums** for fixed value sets; **strings** for open-ended user input
- ✅ Mark all essential parameters in the **`required` array** — Claude may omit optional ones
- ✅ **Eliminating unnecessary dependencies** between tools reduces latency and failure risk
- ✅ When two tools are always called together, consider consolidating them
- ✅ Add "when NOT to use" guidance in descriptions when tools might be confused with each other

---

## 🧩 Scenario-Based Thinking

**Scenario:** An MCP server has two tools: `archive_file(file_id)` and `delete_file(file_id)`. Company policy requires backup files to always be archived, never deleted. Production logs show the agent sometimes calls `delete_file` when users ask to "remove" backup files.

**What change most directly addresses this?**

- A) Remove `delete_file` from the available tools entirely
- B) Add a confirmation step before every delete operation
- C) Expand the tool descriptions — add "Do not use for backup files" to `delete_file`, and add "Use this for backup files instead of delete_file" to `archive_file`
- D) Rename `delete_file` to `permanently_remove_file` to make the consequences clearer

**Answer:** C — Improving the tool descriptions is the most direct fix. By explicitly stating which tool to use for backups and which not to use, Claude's tool selection will improve. Removing the tool entirely (A) prevents legitimate non-backup deletions. Confirmation steps (B) catch the error after selection, not during.

---

## 💡 Memory Tricks

**Description = GPS Direction:** A good GPS gives you specific turn-by-turn instructions (use this tool when, avoid when). A bad GPS just says "drive north" — you can still get lost.

**Enum = Multiple Choice Exam:** A student answering a multiple-choice question can only pick from listed options — they can't write a free-form answer. Enum parameters work the same way — Claude must pick from your defined list.

---

## ❓ Chapter Practice Questions

**Q1.** A tool called `get_data` has the description "Gets data from the system." After deployment, Claude frequently calls it for the wrong use cases. What is the most effective fix?

- A) Rename the tool to a more descriptive name
- B) Rewrite the description to specify exactly what data is retrieved, the format of results, and when this tool should (and should not) be used
- C) Add more parameters to make the tool more powerful
- D) Remove the tool and split it into separate tools for each data type

**Answer:** B

**Explanation:** The tool description is Claude's primary guide for tool selection. A vague description like "Gets data from the system" gives Claude nothing to distinguish this tool from others. A clear description explaining exactly what data is returned and when to use it will dramatically improve selection accuracy.

---

**Q2.** A tool searches one of three internal databases. Users refer to them naturally in conversation ("check the technical specs", "look in the research papers"). How should the database selection parameter be designed?

- A) A string field where Claude passes the user's exact phrase
- B) An enum with values like `research_papers`, `internal_reports`, `technical_specs`
- C) An integer index (1, 2, 3) mapping to each database
- D) Three separate boolean flags, one per database

**Answer:** B

**Explanation:** An enum is ideal when there is a fixed, known set of valid values and users express the same intent in varied natural language. Claude is excellent at mapping "check the technical docs" → `technical_specs`. The enum ensures the tool always receives a valid, structured database identifier, regardless of how the user phrased the request.

---

**Q3.** A `remove_team_member` tool requires a `member_id` parameter (required) and accepts an optional `reason` string. During testing, the agent sometimes omits `member_id`, causing failures. What is the most critical fix?

- A) Add an example of a valid member_id to the tool description
- B) Ensure `member_id` is listed in the `required` array of the input_schema and add type constraints
- C) Make `reason` also required so Claude fills in both fields
- D) Add few-shot examples demonstrating tool calls with member_id

**Answer:** B

**Explanation:** The `required` array in the JSON Schema tells Claude which parameters must always be provided. If `member_id` is not listed there, Claude may treat it as optional and omit it. Listing it as required with a string type constraint is the most direct, reliable fix — schema constraints work regardless of how the user phrases the request.

---

## 📌 Quick Revision Summary

- Tool descriptions are Claude's primary guide for tool selection — write them clearly and specifically
- Include "when to use" AND "when NOT to use" guidance if tools might be confused
- Enums for fixed value sets (databases, categories); strings for open-ended user input
- Mark all essential parameters in the `required` array — Claude may omit optional ones
- Avoid unnecessary tool dependencies — consolidate tools that are always called together
- Avoid overlapping tool names and vague descriptions — they cause incorrect tool selection

---

## 📎 References

- [Anthropic Documentation — Tool Use Best Practices](https://docs.anthropic.com/en/docs/tool-use-best-practices)

---

*Notes by certification-study-hub. Chapter 03 — Designing Good Tools.*
