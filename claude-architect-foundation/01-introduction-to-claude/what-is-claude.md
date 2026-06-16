# What is Claude? — Introduction

> **Exam:** Claude Architect Foundation
> **Chapter:** 01 — Introduction to Claude
> **Difficulty:** Beginner
> **Last Updated:** 2026-06

---

## 📖 What Is This Topic About?

Claude is an AI assistant created by **Anthropic**. It is built to be helpful, harmless, and honest — making safety a design principle, not an afterthought.

Unlike a search engine (which finds documents) or a calculator (which runs fixed logic), Claude understands natural language, reasons across long documents, and generates coherent responses. It is a **Large Language Model (LLM)** trained to be a thoughtful, capable, and safe AI system.

---

## 🧠 Key Concepts

- **Claude** — An AI assistant by Anthropic; not affiliated with OpenAI, Google, or Amazon
- **Anthropic** — The AI safety company that created Claude; its core mission is the responsible development and maintenance of advanced AI for the long-term benefit of humanity
- **Constitutional AI (CAI)** — Anthropic's training method that teaches Claude to evaluate its own outputs against a set of guiding principles, rather than purely optimising for human approval
- **HHH Principles** — Helpful, Harmless, Honest; the three core values that guide all of Claude's responses
- **Context Window** — The maximum amount of text (measured in tokens) that Claude can read and consider in a single conversation

---

## 📘 Detailed Explanation

### What Makes Claude Different?

Many AI models are trained simply to generate text that humans rate highly. The problem: humans sometimes rate confident-sounding but wrong answers more favourably than honest, uncertain ones.

Anthropic took a different approach. They designed a training method — **Constitutional AI** — where Claude learns to evaluate its own outputs against a set of principles before finalising a response. This makes Claude more honest, more cautious about harmful content, and more willing to say "I don't know."

### The Three H's — HHH Principles

| Principle | What It Means | Example |
|---|---|---|
| **Helpful** | Genuinely useful, addressing what the user actually needs | Answering the real question, not just the literal one |
| **Harmless** | Avoiding content that causes physical, emotional, or social harm | Refusing to provide instructions for dangerous activities |
| **Honest** | Truthful; expressing uncertainty rather than fabricating confident answers | Saying "I'm not certain, please verify" instead of guessing |

### Claude vs Other AI Systems

| Aspect | Claude | Traditional Chatbot |
|---|---|---|
| Training focus | Safety + capability | Task completion |
| Uncertainty handling | Explicitly states uncertainty | May hallucinate |
| Harm avoidance | Designed in at training | Usually rule-based filters |
| Context window | Up to 200K tokens | Often much smaller |

### Claude's Context Window

Claude can process up to **200,000 tokens** in a single conversation (approximately 150,000 words — longer than most novels). This means:
- You can paste in entire codebases for analysis
- Long legal or medical documents can be reviewed in one go
- Multi-turn conversations can maintain context without losing earlier details

> **Important for the exam:** The context window includes everything — system prompts, conversation history, documents you paste in, and tool definitions. All of these consume tokens.

---

## 🇮🇳 Indian Real-Life Example

**Think of Claude like a very knowledgeable professor at IIT:**

Imagine a professor who:
- Genuinely wants you to understand the subject, not just pass the exam (**Helpful**)
- Will not help you cheat or do something unethical, even if you ask nicely (**Harmless**)
- Honestly tells you "I am not sure about this specific regulation, please check the official gazette" instead of making up an answer (**Honest**)
- Can hold in memory an entire semester's worth of notes and refer back to any part during the conversation (**Long Context Window**)

---

## 🔑 Exam-Focused Points

- ✅ Claude is created by **Anthropic** — not Google, OpenAI, Microsoft, or Amazon (though Claude is available through Amazon Bedrock)
- ✅ The three core principles are **Helpful, Harmless, Honest** (HHH)
- ✅ **Constitutional AI** is how Anthropic trains Claude — self-evaluation against principles
- ✅ Claude's context window can be up to **200,000 tokens** — everything in a conversation counts toward this limit
- ✅ Claude is designed **safety-first** — safety is baked into training, not bolted on after
- ✅ Claude will express **uncertainty** rather than fabricate confident but wrong answers

---

## 🧩 Scenario-Based Thinking

**Scenario:** A company wants to deploy an AI assistant for customer support. The support team is worried the assistant might give confident but incorrect answers about refund policies, misleading customers.

**Which Claude principle most directly addresses this concern?**

**Think it through:**
1. The concern is about confident but *wrong* answers — this is a truth/accuracy problem
2. "Helpful" is about usefulness — not directly about accuracy
3. "Harmless" is about avoiding harm — closer, but the key issue is truth
4. "Honest" is exactly this — Claude is trained to express uncertainty rather than state incorrect information with false confidence

**Answer: Honesty** — Claude acknowledges when it is uncertain rather than generating plausible-but-wrong responses.

---

## 💡 Memory Tricks

**The 3H Rule:** *"A great friend gives Helpful advice, never causes Harm, and is always Honest."* That is Claude.

**Context Window = Hotel Room:** Everything in your hotel room (your bags, your belongings, the furniture) takes up space. Similarly, everything in a conversation — your message, the system prompt, uploaded documents, tool definitions — all takes up context window space.

---

## ❓ Chapter Practice Questions

**Q1.** Which company created Claude?

- A) OpenAI
- B) Google DeepMind
- C) Anthropic
- D) Amazon

**Answer:** C

**Explanation:** Claude is created by Anthropic, an AI safety company. It is *available* through Amazon Bedrock, but that does not mean Amazon created it. OpenAI makes ChatGPT; Google DeepMind makes Gemini.

---

**Q2.** What are the three core principles that guide Claude's design and training?

- A) Fast, Scalable, Secure
- B) Helpful, Harmless, Honest
- C) Intelligent, Responsive, Efficient
- D) Creative, Accurate, Consistent

**Answer:** B

**Explanation:** The HHH principles — Helpful, Harmless, Honest — are the foundation of Claude's design. These principles are instilled during training through Constitutional AI, not enforced through external filters alone.

---

**Q3.** A developer reports that Claude sometimes says "I am not certain about this — please verify with the official documentation" instead of giving a direct answer. This is an example of which principle?

- A) Helpful — Claude is being helpful by admitting its limits
- B) Harmless — Claude is avoiding causing harm through bad information
- C) Honest — Claude is expressing genuine uncertainty rather than fabricating a confident answer
- D) Constitutional AI — Claude is following its training rules

**Answer:** C

**Explanation:** Expressing uncertainty is a direct manifestation of the Honesty principle. Claude is trained to acknowledge what it does not know rather than hallucinate a confident response.

---

## 📌 Quick Revision Summary

- Claude = AI assistant by Anthropic (not Google, not OpenAI)
- HHH Principles: Helpful, Harmless, Honest
- Constitutional AI = Anthropic's training method; Claude evaluates its own outputs against guiding principles
- Context window = up to 200K tokens; everything in a conversation counts toward this limit
- Claude expresses uncertainty rather than fabricating confident answers
- Safety is baked into training — it is a core design goal, not a filter

---

## 📎 References

- [Anthropic Official Website — anthropic.com](https://www.anthropic.com)
- [Claude Official Page — claude.ai](https://claude.ai)
- [Anthropic's Model Card — anthropic.com/model-card](https://www.anthropic.com/model-card)

---

*Notes by certification-study-hub. Chapter 01 — Introduction to Claude.*
