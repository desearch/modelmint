---
title: ModelMint — Evaluation Layer
tags: [modelmint, product, evaluation, accuracy]
---

## Why Evaluation Is the Core Product

> "If the accuracy metric is doubted once, the product collapses."

ModelMint is not building model pipelines or infra abstraction. It is building **a measurement system that customers trust.** This is what every competitor has avoided because it's hard.

The evaluation layer has two forms:

**Form 1 — The Audit Tool (first product, standalone)**
A free tool that tests a business's *existing* AI against their own domain. No model training involved. Outputs an accuracy score + dollar cost of failures. This is the top-of-funnel, the trust-builder, and the ground truth data generator — all in one. See [[validation-plan]] for why this must come before any model pipeline.

**Form 2 — The Continuous Benchmark (post-improvement)**
The ongoing measurement layer after ModelMint has improved the model. Three-layer architecture below.

---

## Three-Layer Evaluation Architecture

**Layer 1 — User-visible benchmark (20 questions)**
- Creates buy-in (user invests before product is built)
- Gives immediate before/after feedback
- Effective psychology: forces "investment" in the platform

**Layer 2 — Hidden holdout set (system-controlled)**
- Prevents gaming / Goodhart's Law overfitting
- Never exposed to user
- Teacher LLM generates 10 adversarial variations of the approved 20
- Countermeasure: paraphrase test questions, introduce adversarial phrasing, mix unseen document fragments

**Layer 3 — Live production sampling**
- Captures real queries in production
- Scored weekly against benchmark
- If accuracy drops below threshold → retrain triggered automatically
- Without this layer, accuracy becomes stale within weeks

---

## Ground Truth Capture — Assisted, Not Manual

**Don't do:** "Answer 20 questions" (high cognitive load, low completion rate)

**Do instead:**
1. System generates 50 candidate Q&A pairs from corpus (via teacher LLM)
2. User performs: **Approve / Edit / Reject**
3. Target: 20 approved pairs become the visible benchmark
4. System auto-generates 10 adversarial variants for the hidden holdout

This converts *creation → selection* and *cognitive load → validation task*. Completion rate and dataset quality both increase.

---

## Automated Scoring Primitives

- **Embedding similarity** — semantic match between model output and ground truth
- **LLM-as-judge** — rubric scoring via teacher LLM (Claude Sonnet recommended for nuance detection; less prone to sycophancy than smaller judges)
- **Rejection threshold** — configurable per vertical
- **Consistency tracking** — answer consistency across rephrased queries, not just correctness on fixed inputs

---

## Teacher / Judge Architecture

| Role | Model | Why |
|---|---|---|
| **Teacher (generation)** | Gemini 1.5 Flash | 2M token context window — ingests entire catalog/policy in one pass; best speed/cost for high-volume synthetic Q&A |
| **Judge (scoring)** | Claude 3.5 Sonnet | Best nuance detection and rubric-following; least sycophantic judge available |

---

## Goodhart's Law Countermeasures

If you optimize only for the visible benchmark: model becomes "benchmark-smart" but "production-dumb."

Required mitigations:
- Hidden holdout set user never sees
- Distribution shift introduced intentionally (paraphrasing, adversarial phrasing, unseen fragments)
- Track answer *consistency* across rephrased queries, not just fixed-input correctness

---

## Key Takeaways

- The evaluation dataset is the real asset — not the LoRA adapter
- Three-layer eval (visible + hidden + live) is non-negotiable; any single layer is insufficient
- Ground truth capture must be assisted (approve/edit/reject), not manual creation
- Cost per successful answer is the right internal metric, not cost per token
- Vertical-specific evaluation datasets accumulated over time are not replicable by later competitors

**Related:** [[concept-evolution]] · [[moat-analysis]] · [[validation-plan]] · [[strategic-positioning]]
