---
title: ModelMint — Validation Plan & MVP
tags: [modelmint, validation, mvp, roadmap]
date: 2026-04-09
---

## The Constraint Has Shifted

The engineering questions (LoRA efficiency, vLLM scaling, synthetic data generation) are already answered. Those are known.

**What actually needs to be proven:**
1. Businesses care about *measured* accuracy — not just "better AI"
2. They trust *your* measurement enough to act on it
3. They are willing to pay when the measurement reveals the gap

Everything else follows from these three. If these fail, no pipeline architecture saves the business.

---

## Validate Before Writing a Line of Code

**Demand-side:**
- [ ] Will SMBs pay for measured accuracy vs just using ChatGPT? What's the switching trigger?
- [ ] Is the frustration with generic AI sharp enough to create urgency — or tolerable today?
- [ ] Who inside an SMB makes this buying decision? IT? The owner? Operations?

**Supply-side:**
- [ ] What is the real quality of automated corpus ingestion on messy SMB documents (scanned PDFs, inconsistent SOPs)?
- [ ] What adapter quality can be achieved with fully automated synthetic data — no human curation?
- [ ] At what failure rate does automated ingestion require human intervention?

**Market-side:**
- [ ] Does the free accuracy audit tool produce scores that feel intuitively believable to business owners?
- [ ] Do users look at their score and say "this explains our problem" or "this doesn't match my experience"?
- [ ] Is the vernacular domain accuracy gap (not language fluency) still real after Tiny Aya Fire?

---

## Corrected MVP Sequence

The original plan built the model pipeline first, then the evaluation tool. **This is wrong.** The audit tool is the first product, not a lead magnet.

**Why the audit tool comes first:**
- Creates pain visibility — without it, no one buys
- Generates ground truth datasets at zero additional cost
- Exposes corpus quality reality before infra investment
- If it fails to produce believable scores, the rest of the business is invalid

### Phase 1: Accuracy Audit Tool (Zero model infra required)

**What it does:**
1. User uploads domain corpus
2. System generates 20-30 candidate Q&A pairs from the corpus (teacher LLM)
3. User approves / edits / rejects pairs (the ground truth capture)
4. System tests their *existing AI* (ChatGPT, Gemini, etc.) against the approved pairs
5. Output: accuracy score + failure category breakdown + estimated dollar cost of wrong answers

**What you learn:**
- Whether users complete the flow without heavy drop-off
- Whether the accuracy scores feel believable to them
- What the real corpus looks like in the wild (OCR failures, sparse coverage, inconsistent terminology)
- Whether any users react with: "this explains our problem — how do we fix it?"

**That last reaction is your conversion trigger.** Without it, do not proceed.

**Target:** 30 D2C SMBs + 10 Telugu regional businesses. Outreach message: *"Free AI accuracy audit — we'll show you exactly where your AI is getting your domain wrong, in 20 minutes."*

---

### Phase 2: Manual Improvement (1-2 customers, no automation)

Take the best corpus from Phase 1 — the one where the audit revealed a real, measurable gap.

**Manually execute the full pipeline:**
1. Clean the corpus (fix OCR, normalize formats, remove noise — this is where the real learning happens)
2. Generate curated Q&A dataset (human-reviewed, not just automated)
3. Test RAG vs fine-tune vs hybrid on this specific corpus
4. Deploy on RunPod, measure real accuracy delta, real cost, real latency

**Do not automate anything in Phase 2.** The manual process is how you discover what breaks, what needs human judgment, and what is safe to automate later.

**The real pipeline (not the pitch version):**
```
Documents → Cleaning → Structuring → Evaluation → Strategy Selection → Model
```
The first three steps are not trivial, not automated initially, and not visible in the current pitch. They must be understood before any automation is built.

**Deliverable:** One real case study with before/after accuracy scores and a cost-per-correct-answer number.

---

### Phase 3: Controlled Automation (Months 2-3)

Automate only the steps that worked cleanly and consistently in Phase 2. Keep edge cases manual.

**Typically safe to automate first:**
- Document chunking and indexing
- Q&A pair generation (with human review step retained)
- Baseline accuracy scoring

**Keep manual for longer:**
- Corpus cleaning and OCR correction
- Strategy selection (RAG vs fine-tune vs hybrid)
- Edge case query handling

---

### Phase 4: First Paid Engagement

**Structure:** Project-based, not subscription. Fixed scope, fixed corpus, fixed evaluation set, fixed endpoint for 60 days.

**Pricing:** $500-1,500 as a project fee. Do not offer recurring subscription until the customer demonstrates production dependency on the endpoint.

**Conversion signal:** If the customer asks "what happens if I stop paying — do I lose the endpoint?" — that is the subscription trigger.

---

## Go / No-Go Criteria (Binary — No Partial Credit)

1. **Trust gate:** Accuracy audit scores are considered believable by at least 4 of 5 users who complete the audit. If scores don't feel real, the measurement system is broken.
2. **Quality gate:** Manual pipeline improvement produces ≥2x accuracy gain over baseline for at least 2 of 3 pilot customers.
3. **Economics gate:** Total serving cost per customer <30% of equivalent frontier API cost, *after accounting for corpus cleaning and evaluation overhead.*
4. **Demand gate:** At least 1 of 3 pilots converts to paid before any automation is built.

**If trust gate fails** → the evaluation methodology is wrong. Fix scoring before everything else.
**If quality gate fails** → corpus quality is the bottleneck. Add human curation to the loop permanently.
**If economics gate fails** → the cost model is broken. Do not proceed to self-serve.
**If demand gate fails** → the vertical is wrong. Not the product. Move to legal (highest WTP) or vernacular domain (strategic moat).

---

## Corpus Quality Is the Hidden Constraint

The real failure mode for Phase 1 and 2 is not model quality — it is corpus quality. Most SMB document repositories contain:
- Scanned PDFs with poor OCR
- Inconsistent terminology across documents
- Outdated or contradictory policies
- Knowledge that lives in people's heads, not in documents

Estimated failure rate on real SMB corpora with fully automated ingestion: **40-60%.**

**Required adjustment:** For the first 20 customers, treat corpus ingestion as a service layer. Manually clean, normalize, and structure the corpus. This is where the automation roadmap comes from — you cannot automate what you have not mapped manually.

---

## What "Accuracy Guarantee" Must Become

Remove "guaranteed" from all communications immediately.

**Replace with:**
- "benchmarked accuracy"
- "continuously monitored performance"
- "measurable improvement over your current baseline"

The system communicates **transparency**, not certainty. In domains like healthcare or legal, even a 1% failure rate can be unacceptable. The claim of 30% → 5% across diverse SMB corpora with automated pipelines is not defensible until you have production evidence across at least 10 customers.

---

## Key Takeaways

- The audit tool is the first product, not a lead magnet — if it fails, everything else is invalid
- Phase 2 must be entirely manual — do not automate before you understand the failure modes
- The trust gate is the most important of the four go/no-go criteria
- "Cost per correct answer" is the metric to track internally and communicate externally
- Corpus cleaning is a service layer for the first 20 customers — that is where the real learning lives

**Related:** [[concept-evolution]] · [[business-model]] · [[evaluation-layer]] · [[strategic-positioning]]
