---
title: ModelMint — Validation Plan & MVP
tags: [modelmint, validation, mvp, roadmap]
---

## Validate Before Writing a Line of Code

Three questions determine if this is a real business:

**Demand-side:**
- [ ] Will SMBs pay for a private model vs just using ChatGPT? What's the switching trigger?
- [ ] Is the frustration with generic AI sharp enough to create urgency — or is it tolerable today?
- [ ] Who inside an SMB makes this buying decision? IT? The owner? Operations?

**Supply-side:**
- [ ] What is the real cost of one fine-tuning run end-to-end at production quality?
- [ ] What adapter quality can be achieved with fully automated synthetic data — no human curation?
- [ ] How many tenants can realistically share one A100 node before latency degrades?

**Market-side:**
- [ ] Why hasn't Predibase / Together AI / HuggingFace already moved here? What's the actual blocker — technical, GTM, or just attention?
- [ ] Is the vernacular wedge (Indian languages) a durable differentiator or a 12-month window?

---

## 4-Week Validation Sprint (Before MVP)

**Week 1-2: Demand validation**
- 20 cold outreach messages to SMB owners (legal, healthcare, D2C) asking: *"If you could have an AI trained only on your business documents, what would you use it for — and what would you pay?"*
- No product mention. Just discovery.

**Week 3: Supply validation**
- Run the full loop manually once: take a real domain corpus → generate synthetic Q&A with Claude/Gemini → QLoRA fine-tune Gemma 2B with Unsloth → serve with vLLM → benchmark quality
- Measure: cost, time, quality score vs base model on domain questions
- This is the technical proof-of-concept. One weekend of GPU time.

**Week 4: Pricing signal**
- Show a Loom demo of the manual loop to 5 warm contacts
- Present three pricing tiers
- Ask which tier they'd pay for *today* if it existed
- No code needed. Wizard-of-Oz the demo if necessary.

**Decision gate:** If 3 of 20 outreach contacts express willingness to pay AND the manual loop produces measurably better domain answers than GPT-4o → build the MVP.

---

## MVP Scope (Ruthless Cut)

**One vertical. One pipeline. One benchmark.**

**Chosen vertical: D2C / E-commerce support bots**
Reasons: fastest sales cycle, clearest ROI (deflected tickets = saved cost), corpus is simple (catalog + policy docs), no regulatory risk, high volume.

**MVP deliverables — 6 weeks:**

| Week | Deliverable |
|---|---|
| 1 | Teacher (Flash) → Synthetic Q&A → User Validation UI (approve/edit/reject) |
| 2 | Judge (Sonnet) implementation — automated scoring vs ground truth |
| 3 | Three-layer eval: visible benchmark + hidden holdout + adversarial set |
| 4 | The Router — confidence-based switching between SLM (Gemma 2B) and frontier |
| 5 | Vernacular pre-processing (handling Tanglish/code-switched Telugu) |
| 6 | Pilot launch — 3 paying pilots onboarded manually (Wizard-of-Oz the UI) |

**No full platform. No multi-vertical. No self-serve yet.**

**Week 6 Go/No-Go:** If blended accuracy (SLM + Router) is not within 2% of GPT-4o for the D2C pilot → do not launch dashboard, iterate on retrieval quality first.

---

## Go / No-Go Criteria (Binary — No Partial Credit)

1. **Quality gate:** Fine-tuned / RAG-optimized model scores ≥2x better than base model on domain Q&A benchmark for at least 2 of 3 pilots
2. **Economics gate:** Total serving cost per customer <30% of what they'd pay calling GPT-4o directly for equivalent query volume
3. **Demand gate:** At least 1 of 3 pilots converts to paid before full platform is built

**If quality gate fails** → automation pipeline needs human curation in the loop. Redesign before scaling.
**If economics gate fails** → multi-tenant LoRA model needs rework. Don't proceed to self-serve.
**If demand gate fails** → the vertical is wrong. Not the product. Pivot to legal assistants or vernacular SME.

---

## Pivot Options If Gates Fail

- If fine-tuning economics don't work → pivot to RAG optimization platform instead
- If D2C vertical fails → try legal (highest WTP) or vernacular SME (strategic moat)
- If quality can't be automated → add human curation step in the loop

---

## Key Takeaways

- Validate demand before any engineering — 20 outreach messages cost nothing
- The manual loop proof-of-concept (one weekend of GPU time) answers the supply-side question
- Wizard-of-Oz the UI for the first 3 pilots — don't build before you know it sells
- The go/no-go criteria are binary; no partial credit means no self-deception

**Related:** [[concept-evolution]] · [[business-model]] · [[evaluation-layer]]
