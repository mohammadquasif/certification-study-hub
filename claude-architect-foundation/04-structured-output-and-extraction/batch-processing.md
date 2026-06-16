# Batch Processing — Message Batches API & Cost-Efficient Extraction

> **Exam:** Claude Architect Foundation
> **Chapter:** 04 — Structured Output & Extraction
> **Difficulty:** Intermediate
> **Last Updated:** 2026-06

---

## 📖 What Is This Topic About?

Processing thousands of documents through Claude's API can be expensive if each document is processed with a real-time API call. The **Message Batches API** offers 50% cost savings in exchange for asynchronous processing with up to a 24-hour window. Understanding when to use batch vs real-time processing — and how to design reliable batch pipelines — is a key exam topic.

---

## 🧠 Key Concepts

- **Message Batches API** — An Anthropic API for submitting large numbers of requests asynchronously; results are available when processing completes (up to 24 hours)
- **Real-Time API (Messages API)** — The standard API for immediate, synchronous responses; results available within seconds
- **SLA (Service Level Agreement)** — A commitment about how quickly results will be delivered after input is received
- **Batch Window** — The time interval at which you submit accumulated documents as a batch
- **Throughput** — The number of documents processed per unit of time
- **Cost per Extraction** — The API cost to process one document; reduced by 50% with the Batches API

---

## 📘 Detailed Explanation

### Real-Time API vs Message Batches API

| Dimension | Real-Time (Messages API) | Batch (Message Batches API) |
|---|---|---|
| **Response time** | Seconds | Up to 24 hours |
| **Cost** | Standard pricing | 50% discount |
| **Use case** | Latency-sensitive, interactive | Asynchronous, high-volume |
| **Tool calls** | Supported (iterative loops) | NOT supported during processing |
| **Best for** | Urgent/interactive queries | Document processing, overnight jobs |
| **Reliability** | Real-time feedback | Results file when complete |

---

### Decision Framework: Batch vs Real-Time

```
┌───────────────────────────────────────────────────────────────────┐
│             BATCH vs REAL-TIME DECISION FRAMEWORK                 │
│                                                                   │
│  Is a response needed immediately (< 60 seconds)?                 │
│         │                                                         │
│    ┌────┴────┐                                                    │
│    YES       NO                                                   │
│    │         │                                                    │
│    ▼         ▼                                                    │
│  Use       Does the workflow require tool calls during processing? │
│  Real-     │                                                      │
│  Time      ┌────┴────┐                                           │
│  API       YES       NO                                           │
│            │         │                                            │
│            ▼         ▼                                            │
│          Use       Can you tolerate up to 24-hour processing?     │
│          Real-     │                                              │
│          Time      ┌────┴────┐                                   │
│          API       YES       NO                                   │
│                    │         │                                    │
│                    ▼         ▼                                    │
│                  Use       Use Real-Time API                      │
│                  Batches                                          │
│                  API                                              │
└───────────────────────────────────────────────────────────────────┘
```

---

### Designing a Reliable Batch Pipeline

The most common mistake: submitting all documents in a single giant batch at midnight. If that batch fails or is delayed, all documents are late.

#### Batching Strategy — Use Rolling Windows

Instead of one batch per day, submit batches at regular intervals throughout the day:

```
┌─────────────────────────────────────────────────────────────────┐
│                  ROLLING BATCH STRATEGY                         │
│                                                                 │
│  SLA: Results within 30 hours of document arrival              │
│  Batch API maximum processing: 24 hours                         │
│  Therefore: Must submit within 30 - 24 = 6 hours of arrival     │
│                                                                 │
│  SOLUTION: Submit batches every 6 hours                         │
│                                                                 │
│  Document arrives at 2:00 AM →                                  │
│  Included in 6:00 AM batch →                                    │
│  Batch completes by 6:00 AM + 24h = 6:00 AM next day →         │
│  Total time: 28 hours < 30-hour SLA ✅                          │
│                                                                 │
│  Worst case: Document arrives just after 6:00 AM batch submits →│
│  Included in 12:00 PM batch →                                   │
│  Completes by 12:00 PM + 24h = 12:00 PM next day →             │
│  Total time: ~24 hours < 30-hour SLA ✅                         │
└─────────────────────────────────────────────────────────────────┘
```

---

### Handling Batch Failures — Resubmitting Failed Documents

In a large batch (10,000 documents), some documents may fail — typically due to `context_length_exceeded` errors for very large documents.

```
BATCH RESULTS FILE contains:
• 9,700 successful extractions ✅
• 300 failures with error type "context_length_exceeded" ❌

MOST COST-EFFECTIVE RECOVERY:
Step 1: Identify only the 300 failed documents from the results file
Step 2: Chunk each failed document into smaller pieces
         (e.g., 200K token document → four 50K chunks)
Step 3: Resubmit only the 300 failed documents (as chunks) as a new batch
Step 4: Merge the partial extractions from each chunk into a single result

DO NOT:
• Resubmit all 10,000 documents → wasteful, re-charges for 9,700 successful ones
• Skip the failures → 3% of your data is missing
• Process failures with real-time API → fine for small numbers, but inefficient for 300
```

---

### Why Batch Processing Cannot Use Tool Calls

The Batch API is **asynchronous** — your request is submitted and results are retrieved later. This architecture is incompatible with iterative tool-use workflows:

```
ITERATIVE TOOL USE REQUIRES:
Request → Claude thinks → Tool call → App executes → Result returned → Claude continues

BATCH API ARCHITECTURE:
Request submitted → (batch processing, no interactive loop) → Results file ready

The "App executes tool → Result returned to Claude → Claude continues" step
CANNOT happen in a batch because there is no live connection during processing.

→ Batch is appropriate for SINGLE-PASS tasks (read document, extract fields, done)
→ Batch is NOT appropriate for tasks requiring tool calls during processing
```

---

### Routing Different Document Types

In many pipelines, different documents have different urgency:

```
EXAMPLE: Two document types, one pipeline

Standard Monthly Reports:
• Arrive continuously throughout the day
• No urgency — team reviews them weekly
• High volume (500/day)
→ BATCH API: 50% cost savings, processed overnight

Urgent Exception Reports:
• Arrive infrequently (2–5/day)
• Must trigger business actions within 30 minutes
• Time-sensitive
→ REAL-TIME API: immediate processing, standard cost
```

This routing strategy maximises cost savings without sacrificing the SLA for urgent documents.

---

## 🇮🇳 Indian Real-Life Example

**Think of Batch vs Real-Time like IMPS vs NEFT bank transfers:**

- **IMPS (Immediate Payment Service)** = Real-Time API. Transfers money in seconds. Available 24/7. Higher fees.
- **NEFT (National Electronic Funds Transfer)** = Batch API. Processed in batches (every 30 minutes during banking hours). Lower cost. Not instant.

If you need to pay someone right now (emergency, time-sensitive), use IMPS. If you're processing your company's monthly salary disbursements on a Friday, use NEFT — lower fees, batch processing, acceptable delay.

Same logic for Claude API: interactive, urgent queries → real-time. High-volume, overnight document processing → batch.

---

## 🔑 Exam-Focused Points

- ✅ Batches API: 50% cost discount, up to 24-hour processing window
- ✅ Batches API does NOT support tool calls during processing — single-pass tasks only
- ✅ **Rolling batch windows** (e.g., every 6 hours) are more reliable than one giant batch per day
- ✅ Batch window interval = SLA latency requirement − 24h maximum batch processing time
- ✅ Handle batch failures by resubmitting only failed documents after chunking — NOT the whole batch
- ✅ Routing: urgent documents → real-time API; cost-sensitive high-volume → batch API

---

## 🧩 Scenario-Based Thinking

**Scenario:** Documents arrive continuously. The SLA requires extraction results within 30 hours of document arrival with 99.9% reliability. You want to use the Batch API (50% cost savings, up to 24-hour processing window).

**Which batching strategy is most appropriate?**

- A) Submit one batch at midnight containing all documents from the day
- B) Submit a new batch for every document as it arrives
- C) Submit batches every 6 hours containing documents from that window
- D) Submit batches every 24 hours at the start of business

**Answer:** C

**Explanation:** With a 30-hour SLA and a 24-hour maximum batch window, documents must be submitted within 6 hours of arrival (30 − 24 = 6). Submitting every 6 hours ensures every document is submitted within this window. A midnight batch (A) means documents arriving at 1 AM wait 23 hours before submission, violating the 30-hour SLA.

---

## 💡 Memory Tricks

**Batch = Overnight Bus:** Cheaper ticket, leaves at set times, arrives the next morning. Good for non-urgent travel. Real-time = Ola/Uber: more expensive, arrives in minutes, available on demand.

**Batch Window Formula:** Maximum wait before submission = SLA − 24h (max batch time). If SLA is 30h, max wait = 6h → submit every 6 hours.

---

## ❓ Chapter Practice Questions

**Q1.** A data team processes 10,000 product description documents daily using the Message Batches API. After the batch completes, the results file shows 300 documents failed with `context_length_exceeded` errors. What is the most cost-effective way to handle these failures?

- A) Resubmit all 10,000 documents with reduced document lengths
- B) Skip the failed documents and note the 3% failure rate as acceptable
- C) Process the 300 failed documents using the real-time API individually
- D) Identify the 300 failed documents, chunk each into smaller pieces, resubmit only those 300 as a new batch, and merge the resulting partial extractions

**Answer:** D

**Explanation:** Resubmitting all 10,000 (A) wastes money re-processing 9,700 successful extractions. Skipping (B) leaves 3% of data missing. Processing individually with real-time API (C) works but costs more than batch for 300 documents. Resubmitting only the failed 300 as a new batch — chunked to fit the context window — is the most cost-effective approach.

---

**Q2.** An analysis system reviews code files using iterative tool calls: it reads files, identifies imports, reads referenced files, and continues until the full dependency tree is mapped. A developer suggests switching to the Message Batches API for cost savings. Why is the Batch API NOT appropriate for this workflow?

- A) The Batch API does not support code-related tasks
- B) The Batch API's asynchronous model prevents executing tool calls mid-request and returning results for Claude to continue analysis — iterative tool-use workflows require a live request-response loop
- C) The Batch API cannot handle large code files
- D) The Batch API only supports text extraction, not code analysis

**Answer:** B

**Explanation:** The Batch API is a fire-and-forget asynchronous model. There is no live connection during processing, so there is no mechanism for: Claude calls tool → App executes → Result returned to Claude → Claude continues. This iterative loop requires the real-time Messages API. Batch is appropriate only for single-pass tasks.

---

**Q3.** A team processes two types of documents through a single Claude extraction pipeline: standard quarterly reports (500/day, reviewed weekly) and urgent incident reports (3–5/day, must trigger alerts within 30 minutes). How should the pipeline route these document types?

- A) Both through real-time API to ensure reliability
- B) Both through batch API for cost savings
- C) Standard reports through real-time API; urgent reports through batch API
- D) Standard reports through batch API for cost savings; urgent reports through real-time API to meet the 30-minute requirement

**Answer:** D

**Explanation:** Standard reports have no latency requirement and high volume — perfect for the Batch API's 50% cost discount. Urgent reports must produce results within 30 minutes — far tighter than the Batch API's 24-hour window, so they require the real-time API. Routing by urgency optimises cost without compromising the SLA for critical documents.

---

## 📌 Quick Revision Summary

- Batches API: 50% cost discount, up to 24h processing window, no tool call support during processing
- Use Batch for: high-volume, non-urgent, single-pass document processing
- Use Real-Time for: urgent documents, interactive workflows, iterative tool-use
- Rolling batch windows: interval = SLA − 24h; submit every N hours, not once per day
- Handle failures: resubmit only failed documents (chunked) — do not resubmit the whole batch
- Route by urgency: urgent → real-time API; cost-sensitive high-volume → batch API

---

## 📎 References

- [Anthropic Documentation — Message Batches API](https://docs.anthropic.com/en/docs/message-batches)

---

*Notes by certification-study-hub. Chapter 04 — Batch Processing.*
