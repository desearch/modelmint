---
title: ModelMint — Concept Evolution (v0.1 → v0.2)
tags: [modelmint, strategy, positioning]
---

## What ModelMint Is

An **accuracy measurement and routing layer** that sits above AI models — measuring whether a business's AI is answering correctly in their domain, improving it when it isn't, and monitoring it continuously so it stays correct.

The core question ModelMint is now answering: *"Can we measure something businesses trust enough to pay for?"*

The user-facing product is not a model. It is a **benchmark** — a score that tells a business exactly where their AI is failing, what it costs them, and what an improved version scores instead.

---

## v0.1 — "Domain SLMs as a Service" (Original Framing)

**Core pitch:** "Upload your documents. Get your own AI model. No ML team needed."

**The insight that made it viable:**
- Open SLMs (Gemma 2B, Phi-3 Mini, Qwen 1.5B) are now production-grade on domain tasks
- Multi-LoRA serving is solved — vLLM hot-swaps per-tenant adapters at sub-100ms
- Synthetic data generation is cheap — Gemini Flash / Claude Haiku generate 10K Q&A pairs for ~$2-5

**The core loop:**
```
Upload docs → teacher LLM generates synthetic Q&A → QLoRA fine-tune runs →
LoRA adapter deployed on shared base → private API endpoint issued → dashboard
```

**What the user sees:** A form. Their documents. A progress bar. An API endpoint. Done. No epochs, no learning rates, no GPU provisioning.

---

## The Critique of v0.1

1. **Wrong core assumption** — Businesses don't want a "custom model." They want *reliability on their specific tasks*. They don't care if it's fine-tuning, RAG, or prompt engineering — they care about outcome quality per unit cost.
2. **Buyer trigger is unclear** — "ChatGPT is sometimes wrong" is not sharp enough to drive onboarding + payment
3. **Fine-tuning is not always better** — RAG + good prompting can match or beat fine-tuning for many use cases at zero retraining cost
4. **Evaluation layer missing** — No way to measure improvement means no credibility

**Conclusion:** v0.1 was a feature. It needed to become a business.

---

## v0.2 — "Accuracy as a Service" (Post-Critique Rewrite)

**Core pivot:** From selling the drill (fine-tuning) to selling the hole (accuracy).

**Repositioned pitch:** *"Reduce wrong AI answers in your business workflows from ~30% to under 5% — benchmarked and monitored."*

("Guaranteed" removed — see [[strategic-positioning]] for why this is a legal and trust liability.)

**Key changes:**

| v0.1 | v0.2 |
|---|---|
| Fine-tuning as the product | Domain accuracy as the product |
| Model factory positioning | Reliability guarantee positioning |
| Fixed fine-tuning pipeline | Dynamic strategy selection (RAG / fine-tune / hybrid) |
| No evaluation layer | Evaluation is the core primitive |
| Upload → endpoint | Upload → accuracy benchmark → improvement loop |

**The strategy selector (internal, invisible to user):**
```
Corpus size < 500 chunks → RAG likely sufficient
Corpus stable + style-heavy → fine-tune
Both needed → hybrid (RAG retrieval + fine-tuned reader)
```

User never chooses between RAG/fine-tune/hybrid. They see one number: **domain accuracy score.**

---

## v0.3 — "Measurement Layer as Identity" (Current Direction)

The strategic question has shifted again — from "can we build this?" to **"can we measure something businesses trust enough to pay for?"**

This is the harder problem, and the right one. It means:
- The **accuracy audit tool is the first product** — not a lead magnet, not a feature, the product
- The model pipeline (fine-tune / RAG / hybrid) is infrastructure *behind* the measurement layer
- ModelMint's long-term identity is **"Datadog for AI accuracy"** — model-agnostic, recurring, compounding
- The moat is the evaluation dataset, not the adapter

See [[validation-plan]] for the corrected MVP sequence and [[strategic-positioning]] for how this identity survives Cohere's Tiny Aya Fire.

---

## Key Takeaways

- The pivot from "model factory" → "accuracy guarantee" (v0.2) was correct but incomplete
- The current identity (v0.3): accuracy measurement and routing layer — model-agnostic, outcome-driven
- "Guaranteed" is removed from the pitch — replaced with "benchmarked and monitored"
- The real asset is ground truth QA benchmarks per domain per language — not adapters, not infra
- Vernacular wedge window: 6-9 months for language fluency (Tiny Aya Fire), 12-18 months for domain accuracy benchmarks — build those datasets now

**Related:** [[evaluation-layer]] · [[tech-stack]] · [[business-model]] · [[validation-plan]] · [[moat-analysis]] · [[strategic-positioning]]
