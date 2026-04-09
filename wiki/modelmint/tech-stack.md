---
title: ModelMint — Technology Stack
tags: [modelmint, tech, infrastructure, architecture]
---

## Full Stack (All Available Today)

| Layer | Tool | Role |
|---|---|---|
| **Base models** | Gemma 2B, Phi-3 Mini, Qwen 1.5B | Student model — open, production-grade, small footprint |
| **Fine-tuning** | Unsloth + QLoRA | 2-5x faster, single GPU, 4-bit quantization |
| **Synthetic data gen** | Gemini Flash / Claude Haiku API | Teacher LLM — ~$2-5 per 10K Q&A pairs |
| **Data filtering** | Sentence-transformers + dedup | Quality and diversity guardrails |
| **Multi-tenant serving** | vLLM with LoRA hot-swap | Native multi-adapter support, sub-100ms adapter swap |
| **Evaluation scoring** | Sentence-transformers + LLM-as-judge | The accuracy number the user sees |
| **Quantization** | AWQ / GGUF | 2B model fits in ~1.5GB VRAM |
| **API gateway + metering** | SupaKey / Kong + Stripe | Credential issuance, rate limiting, usage billing |
| **Corpus ingestion** | LangChain / LlamaIndex | PDF, DOCX, URL, structured data parsing |
| **Storage** | S3 + Supabase | Adapter versioning, dataset storage, tenant isolation |
| **Orchestration** | Celery + Redis or Modal | Async fine-tuning job queue |
| **Fallback routing** | Custom router layer | Hard queries → frontier API, easy → SLM |
| **Drift monitoring** | Scheduled eval jobs + alerting | Accuracy SLA enforcement |
| **Frontend** | Next.js + Tailwind | Simple upload → monitor → deploy UI |
| **Infra** | Lambda Labs / RunPod / Vast.ai | Cheap GPU compute vs AWS |

> No breakthrough technology needed. This is an **integration and abstraction play** over tools that already exist and work.

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

Generic models fail here because training data is inconsistent. ModelMint's advantage: clean, domain-specific language distributions built from curated corpora.

---

## Key Takeaways

- The router is more important than the model — it's the core reliability engine
- Multi-LoRA is viable but has real limits: VRAM fragmentation, cache invalidation, latency under concurrency
- Vernacular preprocessing pipelines are required — not just fine-tuning
- The full distillation loop (corpus → synthetic data → adapter → endpoint) is what nobody has automated end-to-end

**Related:** [[concept-evolution]] · [[evaluation-layer]] · [[knowledge-distillation]] · [[multi-lora-serving]]
