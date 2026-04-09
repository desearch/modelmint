---
title: Knowledge Distillation — LLM to SLM
tags: [ai-techniques, distillation, slm, fine-tuning]
---

## What It Is

Using a large LLM (teacher) to generate domain-rich Q&A pairs, then fine-tuning a small language model (student) on those pairs.

```
Large LLM (teacher)
    ↓  generates domain-rich Q&A pairs
Domain-curated synthetic dataset
    ↓  fine-tune / distill on
Small Language Model (student)
    ↓
Lean, domain-expert SLM
```

This is how **Phi-series (Microsoft)**, **Gemma**, and **Mistral small variants** were built — textbook quality data over massive parameter count.

---

## Why It Works

A general LLM carries enormous weight because it knows everything loosely. A domain SLM only needs to know one thing *deeply*. The parameter budget goes entirely toward that domain's reasoning patterns, vocabulary, and answer structure.

**Key insight:** You don't need the teacher at inference time — just at dataset generation time. The teacher is a one-time cost.

---

## Compounding Advantages

**1. Size collapses legitimately**
A 1-3B SLM trained on 50K high-quality domain Q&A pairs will outperform a 70B general model on that specific domain. Proven with Phi-2, Orca, and Gemma fine-tunes.

**2. Fine-tuning simplifies dramatically**
Narrow domain = no catastrophic forgetting risk across general knowledge. Fewer epochs, smaller dataset, lower GPU cost, faster convergence.

**3. RAG becomes optional or minimal**
If domain corpus is stable (land records schema, aquaculture disease patterns, court procedure logic), distill it in. RAG only needed for live/dynamic facts on top.

**4. Inference cost drops to the floor**
A 2-3B model runs on a single T4 or even CPU with quantization. Critical for unit economics on private inference APIs.

---

## Dataset Generation Strategy

Quality of synthetic Q&A determines everything:

```
Domain corpus (PDFs, docs, structured data)
    ↓
RAG over corpus → LLM teacher generates Q&A
    ↓
Filter: diversity, difficulty gradient, no hallucinations
    ↓
Format: instruction-following pairs
    ↓
Fine-tune SLM (Gemma 2B / Phi-3 mini / Qwen 1.5B)
```

**Quality risks:**
- Synthetic Q&A overfits style of teacher model
- Misses real-world edge cases
- Introduces subtle errors that become "confident but wrong" outputs

**Mitigations:** Three-layer evaluation (visible benchmark + hidden holdout + live sampling), rejection + retraining loop.

---

## LoRA Makes It Even Leaner

You don't even need full fine-tuning. **QLoRA** trains only a small adapter (~1% of parameters) on top of a frozen base SLM:

```
Gemma 2B base (frozen, shared across verticals)
    + LoRA adapter: domain A
    + LoRA adapter: domain B
    + LoRA adapter: domain C
```

Swap adapters at runtime. Entire system runs on modest hardware.

---

## Two Datasets — Two Different Strategic Weights

Distillation produces two distinct datasets. They are not equal in strategic value:

**Training dataset** — the synthetic Q&A pairs used to fine-tune the SLM. Useful, but increasingly commoditized as teacher LLMs get cheaper and better.

**Evaluation dataset** — the user-blessed ground truth QA benchmarks that define what "correct" means in a domain. This is the moat.

> The training dataset builds the model. The evaluation dataset *defines correctness* — and whoever owns that definition wins long-term.

The SLM is a runtime artifact. The evaluation dataset is the compounding asset. This distinction is why ModelMint's identity is the measurement layer, not the model factory. See [[evaluation-layer]] and [[moat-analysis]].

---

## Key Takeaways

- Teacher LLM is a one-time generation cost — cheapest options: Gemini Flash (~$2-5 per 10K pairs), Claude Haiku
- Domain SLMs can outperform 70B+ general models on their specific domain
- The synthetic data quality ceiling is the main risk — requires an evaluation layer to catch failures
- LoRA adapters enable one shared base model serving many domain-specific tenants

**Related:** [[rag-vs-finetuning]] · [[multi-lora-serving]] · [[evaluation-layer]] · [[tech-stack]]
