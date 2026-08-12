# Manaslu — Build Plan Audit: From Toy to Production Service

**Date:** August 2026 | **Scope:** Critical review of manaslu's MVP build plan  
**Source:** `docs/architecture/11-mvp-build-plan.md` + all 11 architecture docs  
**Question:** Will following this plan produce a unique, saleable production service — or a toy?

---

## Verdict: The Plan Builds a Toy. Here's Why.

The current plan delivers **one working form (Form 80) with a vault that can't be demoed**. The entire value proposition — "scan once, every later form starts ~done" — requires two forms to demonstrate. But Form 1221 is deferred to M4, which is a "pilot" milestone, not part of the core build.

**Result:** At M3 (the "MVP" milestone), you have a Form 80 filler that:
- Is slower than Instafill (AI extraction vs manual typing)
- Costs money to run (Claude credits vs Instafill's free tier)
- Only speaks Nepali (Instafill serves 150+ countries in English)
- Has a vault that stores data but can't prove its value

This is not a product. It's a component that needs two forms to demonstrate its reason for existing.

---

## 1. The Fundamental Flaw: MVP Scope Can't Demo The Value Prop

### What the plan says:

> "The demo that sells it: user fills Form 80 once; weeks later opens Form 1221 and watches it arrive ~80% pre-filled."

### What the plan delivers at M3 (MVP):

| Has | Doesn't Have |
|-----|-------------|
| Form 80 extraction + fill | Form 1221 manifest |
| Vault stores facts | Second form to recall from vault |
| SSE API for review | Any way to demonstrate vault-reuse |
| Bilingual NP explanations | A user journey that proves the vault saves time |

### The vault is the product. The vault can't be demoed with one form.

**Fix:** Merge Form 1221 into M2. The plan already acknowledges this — "Form 1221 is the vault-reuse demo" — but treats it as M4 (pilot) rather than core MVP. If the vault is the moat, proving the vault works is NOT a pilot activity. It's the core milestone.

**Revised milestone:** M2 delivers BOTH Form 80 + Form 1221 fills. The demo is: fill Form 80 → log out → come back → fill Form 1221 → 80% pre-filled from vault. That's the "wow" moment. Without it, you're showing a single-form filler that's worse than free competitors.

---

## 2. The Plan Has No User — Only A Consumer

Every milestone is measured by API behavior ("SSE events fire, confirmation returns, PDF downloads"). Not a single milestone measures:

- Can a real Nepali-speaking user complete a fill without help?
- How many fields does a first-time user need to correct?
- Does the NP explanation actually help them understand what to enter?
- Would they recommend it to a friend?

**The concierge fills in M4 are too late.** By M4, the architecture is locked. If users find the extraction inaccurate or the review UX confusing, you're patching a shipped product.

**Fix:** Run a lightweight validation sprint BEFORE M1, not parallel with M1-M2. Specifically:

| Week | Activity | Cost |
|------|----------|------|
| Week 0 | Build a one-page Nepali landing page explaining the value prop | $0 (time only) |
| Week 0 | Post in 3 Nepali-in-Australia Facebook groups, offer 5 free "concierge fills" | $0 |
| Week 1 | Manually extract data from 5 people's documents (you do it, not Claude) | 2-3 hours |
| Week 1 | Fill Form 80 by hand, send them the PDF, ask: "Would you pay $5 for this?" | $0 |
| Week 1 | Count: how many said yes? That's your real market signal — before a single line of code | |

If 0/5 say yes, the pivot is a week of your time, not 8 weeks. If 5/5 say yes, you have validation AND 5 real document samples for your test set.

---

## 3. The Plan Competes On Features Competitors Can't Copy — But Ships None Of Them At MVP

| Competitive moat (from §1) | Shipped at M3 MVP? |
|----------------------------|-------------------|
| Persistent profile vault | ⚠️ Built but can't be demoed (only 1 form) |
| Bilingual explain-while-fill | ⚠️ Only for Form 80 fields, not Form 1221 |
| AU-migration depth | ⚠️ MRZ checksum + date validation only — useful but not visible to user |
| Trust posture (fill-only, provenance, MARN handoff) | ✅ Structural — this one actually ships |

**Three of four competitive advantages are invisible or undemoable at M3.** The one that ships (trust posture) is the one users can't see — they trust you or they don't, but they won't know about the provenance chain.

**Fix:** Make the moat visible:
- Show the vault working (needs Form 1221 in M2)
- Show the NP explanation at confirmation time (already planned, needs content written)
- Show the audit annex as a feature ("Every value in your form traces back to your own document — here's the proof")
- Show the MARN handoff as trust signal, not just a disclaimer

---

## 4. Missing: The Service Layer (What Makes This A Product, Not Code)

Every SaaS API product has layers that the manaslu plan doesn't address:

### 4.1 Pricing & Metering

| Question | Current Plan |
|----------|-------------|
| How much does manaslu cost per fill? | "~$0.10-0.15/session" (Claude cost only) |
| What's the pricing model? | Not defined |
| Free tier? | Not defined |
| Per-form pricing or subscription? | Not defined |
| How does saathi get billed? | Not defined |
| Usage dashboard for consumers? | Not defined |

**Fix:** Add a pricing model to the plan. Even if it's simple: "Free for first 5 forms/user/month, $2/form after." The API needs usage tracking (already in schema: `sessions.spend_tokens`) AND a billing integration — even if it's manual invoicing at MVP.

### 4.2 Consumer Onboarding

What does saathi need to do to start using manaslu?
1. Get a Cloud Run IAM grant (one-time, manual)
2. Configure Identity Platform JWKS URL (one-time)
3. Implement consent flow (manaslu returns `consent/required`, saathi renders it)
4. Integrate SSE + confirmation endpoints

**None of this is documented as a consumer guide.** The architecture docs are internal — they don't tell a consumer "here's how to integrate in 30 minutes."

**Fix:** Write `docs/consumer-onboarding.md` — a step-by-step guide for saathi (the first consumer). If saathi can't integrate in < 1 day, the API contract is too complex.

### 4.3 SLA & Reliability

| Question | Current Plan |
|----------|-------------|
| What uptime does manaslu guarantee? | Not defined |
| What's the deployment strategy? | Not defined (doc 09 mentions traffic-split canary as optional) |
| How are breaking changes communicated? | Not defined |
| What's the incident response process? | Not defined |

**Fix:** Define an SLA — even if it's "best effort at MVP, 99.5% when paid." Define the deployment strategy (at minimum: health check before traffic shift). Define a changelog format.

### 4.4 Developer Experience

| Missing | Why It Matters |
|---------|---------------|
| Generated TypeScript client | Saathi is Next.js — needs typed API calls, not raw fetch |
| OpenAPI spec (published, not just generated) | Consumer can read the API without reading 11 architecture docs |
| Sandbox/test environment | Saathi needs to test against manaslu without burning real Claude credits |
| Webhook/callback alternative to SSE | What if saathi wants async batch processing? |
| Status page | Consumers need to know if manaslu is down |

**Fix:** Add to M3: generated TS client published to npm (or at least committed to manaslu repo). Add to M4: public OpenAPI spec at `api.manaslu.saathi.app/docs`.

---

## 5. Missing: The Full User Journey

The plan stops at "PDF downloaded." What happens next?

### 5.1 Post-Fill

| Question | Current Plan |
|----------|-------------|
| User downloads PDF. Now what? | Plan ends at `fill.completed` |
| Can they re-download later? | `GET /artifacts/{id}` exists — yes |
| Can they edit one field and re-fill? | Not designed |
| Can they share the filled PDF with their agent? | Not designed (signed URLs expire) |
| What if they lose the PDF? | Re-download from artifacts |
| How long are artifacts kept? | D4 retention not decided |

**Fix:** Add "update one field" flow. Add "share with agent" (generate a longer-lived signed URL or a share token). Decide D4 retention NOW — not at M4.

### 5.2 Multi-Session

The vault survives across sessions. But the plan doesn't address:

- User fills Form 80 in January, comes back in June to fill Form 1221. Do their documents still exist? (D4 retention)
- Their passport expires and they upload a new one. Does the vault auto-detect the conflict? (Currently: `profile_facts.status ∈ active|stale|conflicted|deleted` — good schema, but no conflict resolution logic)
- They change their address. How do they update it in the vault without re-filling every form?

**Fix:** Add vault management endpoints: `GET /v1/vault` (list all facts), `PATCH /v1/vault/{key}` (user-updated value, provenance = user_entry), `DELETE /v1/vault/{key}` (remove stale fact).

### 5.3 Error Recovery

User uploads 3 documents, reviews 20 fields, confirms, and... `fill_pdf()` fails because the AcroForm field name changed. What happens?

Current plan: `session.error` SSE event. User loses all their review work. Must start over.

**Fix:** The gap-resolution engine recomputes state — so `resolve_form()` can be called again on the same session. The review confirmations are already persisted in `user_entries`. The only thing that fails is the PDF write. Re-tryable: fix the manifest, call `fill_pdf()` again with the same confirmed values. This works IF the engine is designed for partial-failure recovery.

---

## 6. Missing: Scale & Multi-Tenancy

The plan is single-consumer (saathi) and single-region. That's fine for MVP. But the architecture claims to support multiple consumers:

> "consumer_id recorded on sessions for quota + audit"

Yet there's no:
- Per-consumer rate limiting
- Per-consumer spend tracking
- Per-consumer API key or auth
- Consumer-specific manifest overrides (what if saathi wants Form 80 + another consumer wants Form 47PA?)

**Fix:** If multi-tenancy is the endgame, design the consumer boundary now. At minimum:
- `consumer_id` is required on session creation
- Per-consumer quota (max sessions/day, max spend/month) in a `consumers` table
- Consumer-specific config (enabled forms, custom manifests)

If manaslu is always single-consumer (just saathi), remove the multi-tenant code and simplify. The in-between state (schema supports it but nothing enforces it) is the worst of both worlds.

---

## 7. Missing: Testing Strategy Beyond Golden Files

Golden-file tests verify `fill_pdf()` output. They don't verify:

| Test Type | What It Catches | Currently? |
|-----------|----------------|------------|
| Golden file | PDF output correctness | ✅ Planned (M2) |
| Unit (resolve_form) | Gap computation logic | ✅ Planned (M3) |
| Integration (manaslu → saathi) | Full user journey | ❌ Not planned |
| Chaos (kill Cloud SQL mid-session) | Recovery behavior | ❌ Not planned |
| Load (50 concurrent sessions) | Performance under load | ❌ Not planned |
| Accuracy regression (50-doc set) | Extraction quality over time | ❌ Not planned |
| Guardrail eval (red-team prompts) | Suggestion leakage | ✅ Planned (M3) |
| Migration rollback | Schema change safety | ❌ Not planned |

**Fix:** Add integration tests with saathi in CI. Add a simple load test (10 concurrent sessions, measure p95 latency). Add accuracy regression tests on the golden doc set — CI fails if extraction accuracy drops.

---

## 8. Missing: Content Strategy

The plan says "NP explanations for ~140 Form 80 questions" and calls it "the biggest schedule risk." But there's no content plan:

| Question | Current Plan |
|----------|-------------|
| Who writes the NP explanations? | Not assigned |
| Who reviews them for accuracy? | Not assigned |
| What's the quality bar? | Not defined |
| How are they maintained when forms change? | Not defined |
| What about Form 1221 explanations? | Not mentioned |
| Can the community contribute? | "Consider community review" — not committed |

**Fix:** Content is a first-class deliverable, not a side note. Write a content plan:

1. Create a spreadsheet: field_name | EN_label | NP_label | EN_explanation | NP_explanation | common_mistakes | source_url
2. Populate EN labels from the form manifest (D3 dump)
3. Write NP explanations for top 40 most-confusing fields first (name formats, address formats, employment history, travel history)
4. Get a Nepali speaker to review
5. Remaining 100 fields: shorter, simpler explanations

**Timeline:** 2-3 days full-time, can be done in parallel with M1 extraction code since it's pure writing.

---

## 9. Missing: Go-To-Market

The plan ends at M4 (pilot with 10-20 community users). There's no path from pilot to launch.

| Question | Current Plan |
|----------|-------------|
| How do users discover manaslu? | Via saathi (consumer) |
| How does saathi market itself? | Not manaslu's concern — but the product depends on it |
| What's the conversion funnel? | Not defined |
| How is success measured post-pilot? | Not defined |
| When does manaslu start charging? | Not defined |

**Fix:** The build plan is for manaslu as a backend service, but the product is manaslu + saathi together. Add a joint GTM milestone:

**M5 — Public Launch (Week 10-12):**
- Saathi landing page live (saathi.app)
- 100 waitlist signups from NP Facebook groups
- 20 beta users, 2-week trial
- Metric: ≥70% complete at least 1 fill
- Metric: ≥50% complete a second fill (vault-reuse proven)
- Publish pricing: free for 2 forms/month, $4.99/month unlimited
- Publish first accuracy benchmark from the 50-doc test set

---

## 10. The Revised Plan: What A Production-Ready Milestones Look Like

| M | Weeks | What | Why This Order |
|---|-------|------|---------------|
| **V0 — Validation** | 0-1 | NP landing page, FB posts, 5 concierge fills, 5 document samples collected | Kill or commit before building |
| **M0 — Foundation** | 1 | Repo scaffold, Alembic schema (all tables), 50-doc golden test set collection started, Anthropic DPA inquiry sent, Form 80 D3 dump, ontology v0, NP explanations spreadsheet created | Unblocks everything |
| **M1 — Extraction + Vault** | 1-3 | classify + extract_schema + validate + save_fact + recall_facts. 10-sample accuracy test on passport photos. Vault working (write + read). | Core engine — everything depends on this |
| **M2 — Form 80 + Form 1221 + Fill** | 3-6 | **Both manifests.** Form 80 fill + Form 1221 fill. map_to_form + fill_pdf + annex + post-fill verify. Golden-file tests for both forms. NP explanations for Form 80 (top 40 fields). THE VAULT-REUSE DEMO WORKS. | THE milestone. If this doesn't demo, nothing else matters. |
| **M3 — API + Hardening** | 6-8 | SSE API v1 + confirmations + pending-state recovery. Consent gate + spend caps. Error recovery (retry + Document AI fallback). Rate limiting. Idempotency. Guardrail evals in CI. Generated TS client. | Production-grade API |
| **M4 — Pilot + Content** | 8-10 | 10-20 community users concierge-supported. NP explanations for ALL Form 80 + Form 1221 fields. Vault management endpoints. Audit sampling. Fix issues from pilot. | Real users find real bugs |
| **M5 — Launch** | 10-12 | Landing page live. Public OpenAPI docs. Pricing announced. Accuracy benchmark published. D4/D5 decisions finalized. 100+ beta users. | Public launch |

**Key changes from the original plan:**
1. Validation sprint BEFORE any code (V0)
2. Form 1221 merged into M2 (the vault-reuse demo is core MVP, not pilot)
3. Content (NP explanations) is a tracked deliverable, not a side note
4. Error recovery, rate limiting, idempotency in M3 (not skipped)
5. TS client in M3 (developer experience)
6. GTM milestone added (M5 — the plan previously stopped at pilot)
7. Total timeline: 12 weeks (original was 8 — the extra 4 weeks are validation, Form 1221, hardening, and launch)

---

## 11. What Makes This A Unique, Saleable Service (Not A Toy)

A toy Form 80 filler:
- Works for one form
- Requires 3+ document uploads
- Takes 30+ seconds
- Only the builder has tested it
- No pricing model
- No users

A production Form 80 filler:
- Works for Form 80 AND Form 1221 (proves the vault)
- Recalls from vault on second form (the moat in action)
- Has accuracy numbers (≥95% field-level on 50 docs)
- Has NP explanations for every field
- 10+ real users have completed fills
- Has a pricing model (free tier + paid)
- Has error recovery (doesn't lose user work on failure)
- Has a TS client so saathi integrates in < 1 day
- Has DPA sorted with Anthropic (legal compliance)
- Has a documented accuracy benchmark you can publish

**The difference between toy and product is not the architecture. It's everything around the architecture — the validation, the second form, the content, the users, the pricing, the error recovery. The architecture is sound. The plan around it is what needs work.**

---

*Plan audit compiled: August 2026 | Based on: manaslu docs/architecture/11-mvp-build-plan.md + ARCHITECTURE.md + docs 01-10*
