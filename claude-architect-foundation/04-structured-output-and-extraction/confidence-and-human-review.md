# Confidence & Human Review — Quality Control for Extraction Pipelines

> **Exam:** Claude Architect Foundation
> **Chapter:** 04 — Structured Output & Extraction
> **Difficulty:** Intermediate
> **Last Updated:** 2026-06

---

## 📖 What Is This Topic About?

No extraction system is perfect. Some extractions will contain errors — even when the JSON is valid and all fields are present. The question is: how do you find and fix errors efficiently when you cannot review every single extraction? This chapter covers confidence scoring, routing to human review, stratified sampling, and measuring error rates over time.

---

## 🧠 Key Concepts

- **Confidence Score** — A numerical measure (typically 0–1 or 0–100) indicating how certain the model is about a particular extraction
- **Field-Level Confidence** — Confidence scores for each individual field, rather than just the overall extraction
- **Human Review Queue** — A workflow where low-confidence or flagged extractions are sent for manual verification
- **Stratified Sampling** — Reviewing a fixed percentage of both low-confidence AND high-confidence extractions to catch errors that the confidence score misses
- **Semantic Error** — An extraction error where the JSON is valid but the value is placed in the wrong field or is factually incorrect
- **Calibration** — The process of comparing confidence scores to actual accuracy to determine the optimal routing threshold

---

## 📘 Detailed Explanation

### Why Confidence Scoring?

Schema validation confirms that extracted JSON is structurally correct — but it cannot detect semantic errors:

```
SCHEMA VALIDATION CATCHES:
❌ Missing required field
❌ Wrong type (string where integer expected)
❌ Value not in enum

SCHEMA VALIDATION MISSES (semantic errors):
✅ duration_minutes = 500 (placed in ingredient_quantity_grams field)
✅ price = "New York" (a location value placed in a price field)
✅ Correct field, wrong value ("123 Main St" → "124 Main St" due to OCR noise)
```

Field-level confidence scores help identify which extractions are most likely to contain these silent errors.

---

### Adding Confidence Scores to the Schema

Confidence can be added as companion fields alongside each extracted value:

```json
{
  "name": "extract_meeting_data",
  "input_schema": {
    "type": "object",
    "properties": {
      "meeting_date": {
        "type": ["string", "null"],
        "description": "Date of the meeting in YYYY-MM-DD format"
      },
      "meeting_date_confidence": {
        "type": "number",
        "minimum": 0,
        "maximum": 1,
        "description": "Confidence score for meeting_date extraction. 
                        1.0 = clearly stated in document. 
                        0.5 = inferred from context. 
                        0.0 = no direct evidence."
      },
      "attendee_count": {
        "type": ["integer", "null"],
        "description": "Number of attendees"
      },
      "attendee_count_confidence": {
        "type": "number",
        "minimum": 0,
        "maximum": 1,
        "description": "Confidence score for attendee_count extraction."
      }
    }
  }
}
```

---

### Routing Strategy — When to Send for Human Review

Not all extractions need human review. The goal is to allocate limited reviewer capacity where it matters most:

```
┌──────────────────────────────────────────────────────────────────┐
│                   REVIEW ROUTING DECISION TREE                   │
│                                                                  │
│  Extraction complete                                             │
│        │                                                         │
│        ▼                                                         │
│  Any field confidence below threshold (e.g., < 0.75)?           │
│        │                                                         │
│    ┌───┴───┐                                                     │
│    YES     NO                                                    │
│    │       │                                                     │
│    ▼       ▼                                                     │
│  Route   Was source document flagged as ambiguous or             │
│  to      containing conflicting information?                     │
│  Human   │                                                       │
│  Review  ┌───┴───┐                                              │
│          YES     NO                                              │
│          │       │                                               │
│          ▼       ▼                                               │
│        Route   Add to stratified sample pool                    │
│        to      (reviewed at fixed rate regardless               │
│        Human   of confidence score)                             │
│        Review                                                    │
└──────────────────────────────────────────────────────────────────┘
```

---

### Stratified Sampling — Catching High-Confidence Errors

Even at high confidence, some extractions contain errors. Without sampling, these are never caught:

```
PROBLEM:
• 12% of high-confidence extractions contain errors (discovered in audit)
• With 10,000 daily extractions, 12% = 1,200 undetected errors per day
• These errors are never caught because only low-confidence extractions are reviewed

SOLUTION: Stratified Random Sampling
• Low-confidence extractions (< 0.75): route ALL to human review
• High-confidence extractions (≥ 0.75): route RANDOM 5% to human review

BENEFITS:
✅ Catches errors that confidence scores miss
✅ Measures the true error rate of high-confidence extractions
✅ Detects novel failure patterns (new document formats, new edge cases)
✅ Provides data to calibrate confidence thresholds over time
```

---

### Calibrating Review Thresholds

After collecting human review results, you can calibrate the threshold:

```
CALIBRATION DATA:
• Extractions with confidence 0.70–0.80: 15% error rate → threshold too low
• Extractions with confidence 0.80–0.90: 8% error rate
• Extractions with confidence 0.90–1.00: 3% error rate

INSIGHT: The threshold should be higher (e.g., 0.85) to catch more errors
          while keeping review volume manageable.

RECALIBRATE quarterly as:
• Document types change
• Extraction patterns improve
• New failure modes are discovered from sampling
```

---

## 🇮🇳 Indian Real-Life Example

**Think of confidence scoring + sampling like quality control at a manufacturing plant:**

A saree weaving factory inspects its output:
- Every saree below the minimum quality threshold → full inspection (low-confidence review)
- 5% of sarees above the threshold → random spot inspection (stratified sampling)

Why 5% of the good ones? Because even the "good" batch occasionally has hidden defects. The spot inspection:
- Catches defects that passed the initial threshold
- Gives data to refine the threshold (maybe the threshold is set too low)
- Detects new defect patterns from new looms or materials

---

## 🔑 Exam-Focused Points

- ✅ Schema validation catches structural errors; **confidence scores** help find semantic errors
- ✅ Route extractions to human review when: **low confidence** OR **source document was ambiguous/conflicting**
- ✅ **Stratified random sampling** reviews a fixed percentage of high-confidence extractions — catches errors that confidence scores miss
- ✅ Sampling also enables **error rate measurement** over time — essential for tracking whether improvements work
- ✅ Calibrate thresholds using labeled validation set data — not guesswork
- ✅ Field-level confidence (per field) is more useful than extraction-level confidence (one score for the whole output)

---

## 🧩 Scenario-Based Thinking

**Scenario:** A quarterly audit reveals that 12% of high-confidence extractions (confidence ≥ 0.85) still contain errors. With 20% reviewer capacity, the team needs a sustainable strategy to catch these errors and measure improvement over time.

**Which approach is most effective?**

- A) Lower the confidence threshold from 0.85 to 0.70 so more extractions are reviewed
- B) Review only the 1% of extractions with confidence below 0.70
- C) Implement stratified random sampling — review all low-confidence extractions plus a fixed 5% of high-confidence ones weekly, enabling error rate measurement and novel failure discovery
- D) Hire more reviewers to achieve 100% review coverage

**Answer:** C

**Explanation:** Stratified sampling catches high-confidence errors that the threshold misses, and enables ongoing measurement of the high-confidence error rate. Lowering the threshold (A) increases review volume without necessarily catching the right errors. Reviewing only the lowest-confidence (B) ignores the 12% high-confidence error problem. 100% coverage (D) is unsustainable.

---

## 💡 Memory Tricks

**Confidence Score = Doctor's Blood Test:** A normal result doesn't guarantee perfect health. The doctor still schedules periodic check-ups (stratified sampling) for "healthy" patients — because some problems don't show up in routine tests.

**Calibration = Adjusting the Weighing Scale:** If the scale consistently shows 5kg when the true weight is 4.5kg, you recalibrate. Same with confidence thresholds — if 0.85+ confidence still has 12% errors, recalibrate upward.

---

## ❓ Chapter Practice Questions

**Q1.** A deployment analysis finds that 12% of extractions with confidence scores above 0.90 still contain semantic errors. The extraction pipeline only routes extractions with confidence below 0.85 to human review. What is the primary problem with this strategy?

- A) The confidence threshold is set too high, causing too many reviews
- B) High-confidence extractions are never reviewed, allowing a 12% error rate to persist undetected among those extractions
- C) Semantic errors cannot be detected by any review system
- D) Confidence scoring should not be used for high-volume pipelines

**Answer:** B

**Explanation:** A threshold-only strategy creates a blind spot — extractions above the threshold are assumed to be correct and never reviewed. The 12% finding proves this assumption is wrong. Stratified random sampling of high-confidence extractions would catch these errors, measure the true error rate, and provide data for threshold recalibration.

---

**Q2.** What is the most significant advantage of **field-level** confidence scores over a single extraction-level confidence score?

- A) Field-level scores reduce token consumption
- B) Field-level scores allow reviewers to focus attention on specific fields most likely to contain errors, rather than reviewing the entire extraction
- C) Field-level scores are required for tool use compatibility
- D) Extraction-level scores are not supported by Claude's API

**Answer:** B

**Explanation:** A single extraction-level score tells you "this extraction may have problems" but not where. Field-level scores (e.g., `meeting_date_confidence: 0.4, attendee_count_confidence: 0.9`) tell reviewers exactly which fields to focus on, making review faster and more targeted. This is particularly valuable when extractions have many fields but only one or two are uncertain.

---

**Q3.** An extraction pipeline processes 500 documents daily. With 20% human reviewer capacity (~100 reviews per day), which routing strategy best allocates reviewer attention?

- A) Review the first 100 documents received each day
- B) Review all extractions where any field confidence is below 0.80 (estimated ~60/day) AND a random 5% sample of high-confidence extractions (~20/day) for quality monitoring
- C) Review only the 5 extractions with the lowest confidence scores
- D) Review all 500 extractions by lowering confidence threshold to 0.50

**Answer:** B

**Explanation:** This strategy allocates reviewer capacity effectively: low-confidence extractions (most likely to have errors) get systematic review; a small random sample of high-confidence extractions catches silent errors and measures the true error rate. The combined volume (~80/day) fits within the 100-review daily capacity. Option A (first 100) introduces selection bias; Option C (only bottom 5) is too narrow.

---

## 📌 Quick Revision Summary

- Confidence scores help identify semantic errors that schema validation cannot detect
- Field-level confidence: per-field scores are more actionable than one score for the whole extraction
- Route to human review: low confidence OR ambiguous/conflicting source document
- Stratified sampling: review fixed % of high-confidence extractions to catch silent errors
- Stratified sampling also measures true error rate and discovers novel failure patterns
- Calibrate thresholds regularly using labeled validation data — recalibrate as document types evolve

---

## 📎 References

- [Anthropic Documentation — Evaluating Structured Outputs](https://docs.anthropic.com/en/docs/evaluations)

---

*Notes by certification-study-hub. Chapter 04 — Confidence & Human Review.*
