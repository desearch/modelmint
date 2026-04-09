---
title: Multi-Tenant LoRA Serving
tags: [ai-techniques, lora, vllm, infrastructure, multi-tenancy]
---

## What It Is

Serving many tenants from a single shared base model, with per-tenant LoRA adapters hot-swapped at request time.

```
Shared GPU Node (e.g., single A100 80GB)
└── Gemma 2B base — loaded once, frozen
    ├── Tenant A LoRA: domain A
    ├── Tenant B LoRA: domain B
    └── Tenant N LoRA: ...
```

---

## Why It Matters for Unit Economics

- Base model loaded once — all tenants share its VRAM footprint
- Adapter swap at request time: **<100ms** with vLLM
- Marginal cost per additional tenant approaches **near zero** after base model is loaded
- This is SaaS margin structure applied to AI inference — the economic breakthrough that makes ModelMint viable

---

## The Technology

**vLLM** (as of late 2024): native multi-LoRA serving with hot-swap support, production-ready.

**Quantization:** AWQ / GGUF allows a 2B model to fit in ~1.5GB VRAM, making multi-tenant density much higher.

**Unsloth + QLoRA:** Training adapters 2-5x faster on consumer hardware.

---

## Real Constraints

Hot-swapping LoRA at scale introduces:
- **VRAM fragmentation** — adapters compete for GPU memory
- **Cache invalidation** — KV cache must be cleared between adapter swaps
- **Latency spikes under concurrency** — if too many adapters are loading simultaneously

**You cannot load unlimited adapters per node.** Do not overpromise "infinite multi-tenancy" early.

---

## Warm-Pool Clustering Strategy

Instead of trying to host all adapters equally:

1. **Cluster tenants by vertical + language** (e.g., D2C/Telugu, Legal/English)
   - Tenants in the same cluster share base model weights; adapters are similar in structure
2. **Keep top-N active adapters "hot"** in VRAM at all times
3. **Cold-load long-tail adapters asynchronously** when request arrives
4. **Router handles cold-start gracefully** — if adapter is cold, route query to frontier API for ~30 seconds while LoRA loads in background. Zero latency impact for end-user.

---

## Scaling Considerations

- One A100 80GB can realistically serve dozens of tenants with common base model
- As tenants grow, add GPU nodes — cluster by vertical to maximize adapter cache hits
- Adapter versioning (via S3 + Supabase) allows rollbacks and A/B testing per tenant
- Long-tail tenants (low traffic) can share cold-loading slots without impacting active tenants

---

## Key Takeaways

- Multi-LoRA serving is solved at the infrastructure level (vLLM, late 2024) but requires careful operational design
- The warm-pool clustering + cold-load-with-fallback pattern is the practical mitigation for the cold-start problem
- This is the unit economics engine of the entire ModelMint business model — get this right before scaling
- Never promise unlimited adapters per node; be honest about clustering requirements upfront

**Related:** [[knowledge-distillation]] · [[tech-stack]] · [[rag-vs-finetuning]]
