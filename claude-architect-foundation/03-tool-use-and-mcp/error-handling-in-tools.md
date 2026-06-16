# Error Handling in Tools — Structured Errors & Recovery

> **Exam:** Claude Architect Foundation
> **Chapter:** 03 — Tool Use & MCP
> **Difficulty:** Intermediate
> **Last Updated:** 2026-06

---

## 📖 What Is This Topic About?

Tools fail. APIs go down. Records don't exist. Permissions are denied. How your tool communicates failures to Claude determines whether the agent can recover gracefully or collapses into unhelpful responses. This chapter covers the `isError` pattern, structured error metadata, the difference between transient and permanent errors, and what NOT to do.

---

## 🧠 Key Concepts

- **isError** — A flag in the tool result that signals to Claude that an error occurred; Claude can then decide how to respond
- **Transient Error** — An error that is temporary and may succeed if retried (e.g., network timeout, 503 Service Unavailable)
- **Permanent Error** — An error that will not be resolved by retrying (e.g., 403 Permission Denied, record not found)
- **errorCategory** — A structured field in the error response that classifies the error type (transient/validation/permission)
- **isRetryable** — A boolean field indicating whether the operation is worth retrying
- **Retry Logic** — Automatically attempting the same operation again after a transient failure

---

## 📘 Detailed Explanation

### The Problem with Unstructured Errors

Without structured error information, Claude receives the same signal for all types of failures:

```
UNSTRUCTURED ERROR RESPONSE (bad):
{
  "isError": true,
  "content": [{"type": "text", "text": "Operation failed"}]
}

What does Claude know from this?
• ❌ Was it a temporary network issue?
• ❌ Did the record not exist?
• ❌ Was the permission denied?
• ❌ Should it retry?
• ❌ Should it ask the user?
• ❌ Should it escalate?

Claude cannot distinguish, so it behaves inconsistently.
```

### The Structured Error Pattern (Correct Approach)

```
STRUCTURED ERROR RESPONSE (good):
{
  "isError": true,
  "errorCategory": "transient",
  "isRetryable": true,
  "description": "Database temporarily unavailable due to maintenance window. 
                  Expected to recover within 5 minutes.",
  "httpStatus": 503,
  "suggestedAction": "Wait briefly and retry the operation."
}
```

Now Claude can make an informed decision:
- `errorCategory: "transient"` + `isRetryable: true` → retry after a brief wait
- `errorCategory: "permission"` + `isRetryable: false` → inform the user they lack access; do not retry
- `errorCategory: "validation"` + `isRetryable: false` → the input was invalid; ask for correction

---

### Error Category Decision Tree

```
┌──────────────────────────────────────────────────────────────────┐
│                    ERROR HANDLING DECISION TREE                  │
│                                                                  │
│  Tool returns error                                              │
│         │                                                        │
│         ▼                                                        │
│  What category?                                                  │
│         │                                                        │
│    ┌────┴────────────────────────┬─────────────────────────┐    │
│    ▼                             ▼                          ▼    │
│  TRANSIENT                    VALIDATION               PERMISSION│
│  (network, 503)               (bad input)              (403)     │
│         │                         │                        │     │
│         ▼                         ▼                        ▼     │
│  isRetryable: true          isRetryable: false      isRetryable: │
│  → Retry with               → Ask user to           false        │
│    backoff                    correct input         → Inform user │
│                                                      they can't  │
│                                                      do this;    │
│                                                      escalate    │
└──────────────────────────────────────────────────────────────────┘
```

---

### Transient vs Permanent Errors — Where to Handle

| Error Type | Examples | Handle Where | How |
|---|---|---|---|
| **Transient** | Network timeout, 503 Service Unavailable | **Inside the tool** | Automatic retry with exponential backoff; return to Claude only if all retries exhausted |
| **Permanent (validation)** | Invalid parameter format, missing required field | **Surface to Claude** | Return structured error; Claude can ask user to provide correct input |
| **Permanent (permission)** | 403 Forbidden, unauthorised access | **Surface to Claude** | Return structured error; Claude informs user and may escalate |
| **Permanent (not found)** | Record doesn't exist | **Surface to Claude** | Return structured error; Claude informs user or asks for correct identifier |
| **Ambiguous (timeout)** | Request timed out — may or may not have executed | **Surface to Claude** | Return `isError: true` with uncertainty message — warn against retry to prevent duplicates |

---

### The Dangerous Ambiguous Case — Timeouts

When a tool times out before receiving a response, the operation may or may not have executed:

```
SCENARIO: Send notification → 30s timeout → No response received

DID THE NOTIFICATION GET SENT? 
  ┌─────────────────┐
  │  UNKNOWN        │  The server may have received and processed the
  │                 │  request but timed out before sending a response.
  │  DO NOT RETRY   │  Retrying risks sending the SAME notification TWICE.
  └─────────────────┘

CORRECT ERROR RESPONSE:
{
  "isError": true,
  "errorCategory": "ambiguous",
  "isRetryable": false,
  "description": "Request timed out. The operation may or may not have 
                  completed. Do not retry automatically — manual verification 
                  is recommended before attempting again."
}
```

This communicates uncertainty to Claude, preventing duplicate operations.

---

### What NOT to Do

```
❌ Throw an exception from the tool handler
   → The agent framework crashes; Claude gets no actionable information

❌ Return empty result with no error flag
   → Claude thinks the tool succeeded but returned no data; 
     may fabricate a response based on missing data

❌ Log the error server-side and return success
   → Claude thinks everything worked; downstream failures are mysterious

❌ Uniform error messages for all error types
   → Claude cannot distinguish transient from permanent errors; 
     may retry permanently failed operations or fail to retry transient ones
```

---

## 🇮🇳 Indian Real-Life Example

**Think of structured error handling like a delivery status system:**

When a courier company can't deliver your parcel, a good company doesn't just say "Delivery failed." They tell you:
- **Why** it failed: "Recipient not available" (permanent — don't try same time)
- **Or:** "Vehicle breakdown" (transient — will retry today)
- **Or:** "Address not found" (validation — please provide correct address)
- **What to do next:** "Call to reschedule" / "We'll retry tomorrow" / "Update address"

A bad courier company just says "Unable to deliver" — you don't know what to do next. That's what unstructured errors do to Claude.

---

## 🔑 Exam-Focused Points

- ✅ Always return errors as **tool results with `isError: true`** — never throw exceptions or return empty results
- ✅ Include **errorCategory** (transient/validation/permission/ambiguous) so Claude can act appropriately
- ✅ Include **isRetryable** boolean — Claude should not retry permanent failures
- ✅ Handle **transient errors** (503, timeouts) inside the tool with automatic retry; surface to Claude only after exhausting retries
- ✅ **Ambiguous outcomes** (timeout on write operations) — warn against retry to prevent duplicates
- ✅ Surface **permanent errors** to Claude with descriptive messages so it can explain to the user or escalate

---

## 🧩 Scenario-Based Thinking

**Scenario:** An agent's tool calls an external logistics API that sometimes returns 503 errors. The current implementation raises a Python exception when a 503 is received, causing the agent to produce unhelpful responses.

**What is the most effective improvement?**

- A) Add retry logic with exponential backoff inside the tool for 503 errors; if retries are exhausted, return a structured error result with `isError: true` and a description that the service is temporarily unavailable
- B) Surface the 503 directly to Claude as a tool result with `isError: true`
- C) Ignore the 503 and return empty results to avoid confusing Claude
- D) Add a try-except block that swallows the exception and retries 10 times with no delay

**Answer:** A

**Explanation:** Transient errors like 503 should be handled inside the tool with automatic retry. If all retries fail, return a structured error result to Claude with `isError: true` and clear description. This prevents cascading failures while giving Claude actionable information if the service is truly down.

---

## 💡 Memory Tricks

**isError = Warning Light on Car Dashboard:** The light (isError) tells you something is wrong. But you need more info: is it fuel (transient — fill up and go) or engine failure (permanent — needs the garage)? errorCategory is the specific warning type.

**Transient = Retry Inside. Permanent = Tell Claude.** Handle transient errors yourself inside the tool. Tell Claude about permanent errors so it can communicate to the user.

---

## ❓ Chapter Practice Questions

**Q1.** A tool returns the following error response. What is the most significant problem with it?

```json
{"isError": true, "content": [{"type": "text", "text": "Operation failed"}]}
```

- A) The `isError` field is not a valid JSON type
- B) The error message is too short and should be longer
- C) The response lacks structured metadata (errorCategory, isRetryable, description), so Claude cannot determine whether to retry, escalate, or ask for user input
- D) Tool errors should always raise exceptions, not return results

**Answer:** C

**Explanation:** A uniform "Operation failed" message provides no information about the type of failure. Claude cannot determine whether this is a transient error (retry), a validation error (ask user), or a permission error (escalate). Structured metadata (errorCategory, isRetryable, description) enables Claude to respond appropriately.

---

**Q2.** A billing tool sends payment confirmation emails. During testing, it occasionally times out before receiving a response. What should the tool return in this situation?

- A) An empty success response, since the email may have been sent
- B) An `isError: true` response noting the uncertain state and recommending against automatic retry to prevent duplicate emails
- C) An `isRetryable: true` response prompting Claude to retry immediately
- D) Raise an exception and let the framework handle the timeout

**Answer:** B

**Explanation:** Timeouts on write operations create an ambiguous state — the operation may or may not have completed. Returning `isRetryable: false` with a message explaining the uncertainty prevents Claude from automatically retrying (which would risk sending duplicate emails). The agent or human can investigate the actual state before retrying.

---

**Q3.** When should retry logic with exponential backoff be implemented **inside the tool** rather than being left to Claude to decide?

- A) Always — Claude should never decide when to retry
- B) Never — the agent should control all retry decisions
- C) For transient errors (network timeouts, 503 responses) — these are temporary conditions the tool can handle internally without burdening Claude with retry decisions
- D) Only when the tool has a dedicated retry configuration parameter

**Answer:** C

**Explanation:** Transient errors are predictable, temporary conditions that the tool is well-positioned to handle internally. Implementing retry with exponential backoff inside the tool for 503s and timeouts keeps this implementation detail away from Claude. Claude should only see the error if all retries are exhausted — at that point, it needs to inform the user or escalate.

---

## 📌 Quick Revision Summary

- Always return errors as tool results with `isError: true` — never throw exceptions
- Include errorCategory (transient/validation/permission/ambiguous) and isRetryable
- Transient errors (503, timeout): handle inside tool with retry; surface only if all retries fail
- Permanent errors (403, not found): surface to Claude immediately with clear description
- Ambiguous timeouts: `isRetryable: false` + warn against retry to prevent duplicates
- Empty result with no error flag = silent failure — Claude may fabricate responses on missing data

---

## 📎 References

- [Anthropic Documentation — Error Handling in Tool Use](https://docs.anthropic.com/en/docs/tool-use)
- [Anthropic Documentation — MCP Error Handling](https://docs.anthropic.com/en/docs/mcp)

---

*Notes by certification-study-hub. Chapter 03 — Error Handling in Tools.*
