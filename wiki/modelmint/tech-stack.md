---
title: ModelMint — Technology Stack
tags: [modelmint, tech, infrastructure, architecture]
---

## Stack Priority by Build Phase

The stack is documented here in full, but **not all of it gets built at once.** Build sequence follows the measurement-first strategy — the evaluation and corpus layers come before the model training layers.

### Phase 1 — Audit Tool Only (build this first)

| Layer | Tool | Role |
|---|---|---|
| **Corpus ingestion** | LangChain / LlamaIndex | PDF, DOCX, URL parsing — first point of corpus quality failure |
| **Corpus cleaning** | Manual + custom scripts (Phase 1 is a service layer) | OCR correction, format normalization, noise removal |
| **Q&A generation** | Gemini Flash / Claude Haiku API | Teacher LLM — generates candidate Q&A pairs from corpus |
| **Evaluation scoring** | Sentence-transformers + LLM-as-judge | Scores *existing* AI against user-approved ground truth |
| **Ground truth UI** | Next.js + Tailwind | Approve / Edit / Reject flow |
| **Storage** | Supabase | Ground truth dataset storage per tenant |

### Phase 2-3 — Model Improvement Pipeline (build after audit proves demand)

| Layer | Tool | Role |
|---|---|---|
| **Base models** | Gemma 2B, Phi-3 Mini, Qwen 1.5B | Student model — open, production-grade, small footprint |
| **Fine-tuning** | Unsloth + QLoRA | 2-5x faster, single GPU, 4-bit quantization |
| **Data filtering** | Sentence-transformers + dedup | Quality and diversity guardrails on synthetic Q&A |
| **Multi-tenant serving** | vLLM with LoRA hot-swap | Native multi-adapter support, sub-100ms adapter swap |
| **Quantization** | AWQ / GGUF | 2B model fits in ~1.5GB VRAM |
| **API gateway + metering** | SupaKey / Kong + Stripe | Credential issuance, rate limiting, usage billing |
| **Orchestration** | Celery + Redis or Modal | Async fine-tuning job queue |
| **Fallback routing** | Custom router layer | Hard queries → frontier API, easy → SLM |
| **Drift monitoring** | Scheduled eval jobs + alerting | Accuracy SLA enforcement |
| **Infra** | Lambda Labs / RunPod / Vast.ai | Cheap GPU compute vs AWS |

> No breakthrough technology needed. This is an **integration and abstraction play** over tools that already exist and work. The sequencing is the strategy.

---

## Multi-Tenant Architecture

```
Shared GPU Node (e.g., single A100 80GB)
└── Gemma 2B base — loaded once, frozen
    ├── Tenant A LoRA: aquaculture diagnosis
    ├── Tenant B LoRA: HR policy Q&A
    ├── Tenant C LoRA: legal clause interpretation
    ├── Tenant D LoRA: product catalog assistant
    └── Tenant N LoRA: ...
```

- One GPU node serves dozens of tenants simultaneously
- Adapter swap at request time: **<100ms**
- Marginal cost per additional tenant approaches zero after base model is loaded
- This is the SaaS margin structure applied to AI inference

**LoRA clustering strategy (warm-pool):**
- Cluster tenants by vertical + language (e.g., D2C/Telugu, D2C/Marathi)
- Keep top-N active adapters hot in VRAM
- Cold-load long-tail adapters asynchronously
- If adapter is cold → router sends query to frontier API while adapter loads in background (zero latency impact for user)
- Do not overpromise "infinite multi-tenancy" early

---

## The Fallback Router (Core Reliability Engine)

```
Query arrives
    ↓
Complexity classifier (fast, lightweight)
    ├── High confidence + strong retrieval → SLM (cheap, fast)
    └── Low confidence OR high complexity → frontier API (accurate)
        ↓
Response returned — same endpoint, same UX
```

**User gets:** SLM economics on ~80% of queries. Frontier accuracy on ~20% that need it. Cost blended. Quality uncompromised.

**Minimum viable router logic requires:**
- Confidence score (logits / entropy proxy)
- Retrieval coverage score (how well docs support answer)
- Query complexity classifier

Must be measurable and tunable per customer.

**Poor routing leads to:**
- Cost explosion (over-calling frontier APIs)
- Quality degradation (under-calling them)

---

## Vernacular Pre-processing Layer

For Indian language markets, the edge doesn't come from fine-tuning alone — it requires:
- **Terminology normalization** — local domain vocabulary mapping
- **Code-mixed language handling** — Telugu + English ("Tanglish"), Marathi + English
- **Spelling variance tolerance** — inconsistent romanization

Generic models fail here because training data is inconsistent. ModelMint's advantage is not language generation capability — Cohere's Tiny Aya Fire closes that gap. The advantage is **domain accuracy benchmarks in these languages** — knowing what a correct answer looks like in Telugu agritech or Marathi legal contexts. The preprocessing layer enables the evaluation layer to function correctly in code-mixed inputs.

---

## Key Takeaways

- Build Phase 1 (audit tool) before touching Phase 2-3 (model pipeline) — the sequence is the strategy
- Corpus cleaning is a manual service layer for the first 20 customers, not an automated feature
- The evaluation scoring layer (Phase 1) is the product; the model training layer (Phase 2-3) is the delivery mechanism
- The router is more important than the model — it is model-agnostic and the proof ModelMint sits above the model wars
- Multi-LoRA is viable but has real operational limits — do not build it before demand is proven

**Related:** [[concept-evolution]] · [[evaluation-layer]] · [[knowledge-distillation]] · [[multi-lora-serving]] · [[validation-plan]]
