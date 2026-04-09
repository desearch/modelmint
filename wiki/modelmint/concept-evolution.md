---
title: ModelMint — Concept Evolution (v0.1 → v0.2)
tags: [modelmint, strategy, positioning]
---

## What ModelMint Is

A no-code platform where any business uploads their domain corpus and receives a private, fine-tuned SLM endpoint — trained on their knowledge, deployed instantly, billed like cloud.

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

**Repositioned pitch:** *"Reduce wrong AI answers in your business workflows from ~30% to under 5% — guaranteed by benchmark."*

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

## Key Takeaways

- The pivot from "model factory" to "accuracy guarantee" is the single most important reframe
- v0.2 competes with internal AI teams; v0.1 competed with tooling platforms
- The evaluation layer is the actual product — everything else is infrastructure behind it
- The real asset being built is ground truth QA benchmarks per domain per language — not adapters, not infra
- Vernacular wedge (Telugu, Tamil, Marathi) is a 12-18 month window before frontier labs close it

**Related:** [[tech-stack]] · [[evaluation-layer]] · [[business-model]] · [[validation-plan]] · [[moat-analysis]]
