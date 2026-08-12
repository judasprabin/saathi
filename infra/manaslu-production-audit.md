# Manaslu — Production Readiness Audit

**Date:** August 2026 | **Author:** Architecture Review  
**Scope:** Critical analysis of `judasprabin/manaslu` — what's good, what's missing, what must change before this is saleable.

---

## Executive Summary

Manaslu's architecture is **thoughtful and well-reasoned**. The gap-resolution engine over an agent loop, the provenance chain, and the vault-first design are genuinely good decisions. But **zero code exists** — all 11 docs describe a system that hasn't been built. Between the docs and a working product lies ~6-8 weeks of implementation plus several architectural gaps that need addressing before this is production-ready.

**Overall rating: Architecture B+ / Implementation F / Production Readiness D**

---

## 1. What Manaslu Gets Right

### 1.1 Gap-Resolution Engine Over Agent Loop (Strong)

The D8 reversal — replacing Opus-decides-every-turn with a deterministic Python loop — is the single best decision in the architecture. Reasons:

- **Testable:** The `resolve_form()` function can be unit-tested without mocking an LLM. You can verify "given vault state X and manifest Y, does the engine correctly compute gaps Z?" — this is priceless for reliability.
- **Cheaper:** No Opus reasoning tokens per turn. The cost drops from ~$0.10-0.25/session to ~$0.02-0.05/session.
- **Auditable:** Every decision is in code, not in an LLM's black-box reasoning.
- **Guardrail-enforceable:** The `fill_pdf` provenance check is structural, not prompt-based.

**Recommendation:** Ship this. Do not revert to an agent loop. This is the right architecture.

### 1.2 Provenance Chain (Strong)

```
documents → extractions → profile_facts → fill_values
                ↑ user_entries can also write to both
```

Every value in a filled PDF has a FK back to its source document or user entry. `fill_pdf` enforces this. This is what makes "transcribe, never suggest" structural — not aspirational. Solid design.

**Recommendation:** Keep this. Consider adding a `provenance_strength` field to `fill_values` — values from MRZ-verified passport extractions should carry more weight than values from a low-confidence payslip field.

### 1.3 Vault-First Architecture (Strong)

The vault (`profile_facts`) surviving across sessions is the product moat. Form 80 fills → vault populated → Form 1221 pre-filled weeks later. Competitors (Instafill, FormMate80) are transactional one-shots — they can't do this without rebuilding their entire data model.

**Recommendation:** Build the vault demo (Form 80 → wait → Form 1221 pre-filled) as the MVP's north star. Nothing else matters if this doesn't work.

### 1.4 Tiered LLM Strategy (Good)

Haiku for classification, Sonnet for extraction, Opus reserved for the (now-removed) agent loop. Cost-conscious and well-reasoned.

### 1.5 Bilingual Explain-While-Fill (Good Concept)

Per-field NP explanations at confirmation time. Not yet written (see §3.10), but the concept is strong and aligns with Saathi's positioning.

---

## 2. Critical Gaps — What Must Be Fixed Before Production

### 2.1 ⚠️ CRITICAL: Zero Implementation

| What exists | What doesn't |
|-------------|-------------|
| 11 architecture docs | Zero lines of service code |
| Schema on paper | No Alembic migrations |
| API contract documented | No FastAPI routes |
| pypdf recommended | No `fill_pdf()` function |
| Extraction pipeline diagram | No `classify()` or `extract_schema()` |
| M0 claims "CI/CD skeleton done" | The CI/CD is in `karki-labs-infra`, not manaslu |

**Impact:** Every estimate in the build plan (M1-M4, 8 weeks) assumes the architecture is correct and implementation is straightforward. Without a single function written, all estimates are speculative.

**Fix:** Build the extraction spine first (M1) — `classify()` + `extract_schema()` + MRZ validation — and validate against 10 real passport photos. Only then can you calibrate the rest of the timeline.

### 2.2 ⚠️ CRITICAL: No Golden Test Set

Doc 02 open Q3: "Sample-set accuracy target before launch (proposal: ≥95% field accuracy on a 50-doc labeled set)" — this set **does not exist**. Without it:

- Extraction accuracy is vibes-based
- Prompt changes can't be measured
- Model tiering (Sonnet vs Haiku for which field) is guesswork
- You can't tell a competitor or investor your accuracy numbers

**Fix:** Priority #1 — collect 50 real documents (passports, payslips, bank statements, visa grants, birth certs, transcripts, English test results) and manually label every expected field. Redact PII. Build a pytest harness that runs extraction and reports per-field, per-document-type accuracy. This gates everything.

### 2.3 ⚠️ CRITICAL: Single Point of Failure (Anthropic)

The entire extraction pipeline depends on Claude. Doc 02 acknowledges "vendor concentration" but the mitigation is "Document AI as documented fallback" — not implemented, not tested, not even configured.

**What happens when:**
- Anthropic has an outage (happens ~1-2x/year)
- Claude API returns garbage JSON (model regression)
- Rate limit hit mid-session
- Sonnet 5 deprecated for Sonnet 6 with different behavior

**Current state:** Manaslu is dead until Anthropic recovers. No fallback is wired.

**Fix:** Implement a **dual-provider extraction strategy**:
1. Primary: Claude Sonnet (as designed)
2. Fallback: Google Document AI (same GCP project, already GCP-native)
3. Implement a `try_extract(provider)` function that auto-falls-back on failure
4. Track per-provider accuracy in the audit log so you know when to switch

```python
def extract_with_fallback(doc, doc_type, schema):
    try:
        return extract_claude(doc, doc_type, schema)
    except (ClaudeTimeout, ClaudeRateLimit, ClaudeMalformedResponse):
        logger.warning("Claude extraction failed — falling back to Document AI")
        return extract_document_ai(doc, doc_type, schema)
```

### 2.4 ⚠️ CRITICAL: No Error Recovery For Extraction Failures

The gap-resolution engine's pseudocode has no error handling paths:

```python
# Current pseudocode (simplified)
extracted = extract_schema(doc, classification.type)   # What if this fails?
validated = validate(extracted)                          # What if validate finds everything invalid?
vault.save_facts(uid, validated, source=doc)             # What if DB is down?
```

**Missing error paths:**
- Claude returns malformed JSON → no retry, no fallback, session dead
- Validation rejects every field → user gets a blank review screen with no explanation
- DB write fails mid-session → uploaded documents orphaned in GCS
- SSE connection drops mid-stream → consumer has no way to resume (GET /sessions works but the consumer code doesn't exist)

**Fix:** Every LLM call needs: retry (3× exponential backoff) → fallback provider → graceful degradation. Every DB write needs: transaction rollback → orphan cleanup. The SSE stream needs: heartbeat pings every 15s → consumer can detect dead connections.

### 2.5 ⚠️ HIGH: No Observability Beyond Logging

Doc 09 mentions Cloud Logging + Monitoring. But what metrics actually matter for manaslu?

| Metric | Currently Tracked | Why It Matters |
|--------|------------------|----------------|
| Extraction accuracy per document type | ❌ | Model quality degradation detection |
| Field-level confidence distribution | ❌ | Tune confidence thresholds |
| Vault fill rate (% fields from vault vs extraction) | ❌ | Product moat metric — is the vault actually saving time? |
| Per-session Claude spend | ❌ | Cost tracking, user quotas |
| Session completion rate | ❌ | How many sessions reach `fill.completed`? |
| Average review.confirmations per session | ❌ | Friction metric — how much is the user correcting? |
| p95 extraction latency | ❌ | Performance regression detection |
| SSE connection drop rate | ❌ | Network reliability |

**Fix:** Instrument every stage transition in `resolve_form()` with a custom metric. Build a Cloud Monitoring dashboard with these 8 metrics. Alert on accuracy degradation and spend anomalies.

### 2.6 ⚠️ HIGH: Form Revision Drift Is Detection-Only

The manifest CI job detects when Form 80 field names change. But:

- CI runs on push, not continuously
- Home Affairs can change PDFs any time
- Production keeps serving the old manifest until someone notices and deploys
- No automated notification when drift is detected

**Fix:** Add a **runtime drift check** on session start:
1. `GET /v1/sessions` → engine checks `manifest.last_verified` timestamp
2. If > 30 days old, trigger an async re-dump + diff (non-blocking)
3. If drift detected, flag the session as `manifest_stale=true` and surface in the API response
4. Consumer (Saathi) renders a warning: "This form may have been updated. Some fields may not fill correctly."

### 2.7 ⚠️ HIGH: Transliteration Deferred Post-MVP

Doc 04's Aksharamukha pipeline is labeled "post-MVP." This means Devanagari names from Nepali birth certificates **cannot be auto-filled at launch** — users must type them manually.

For a **Nepali-first product**, this is a major gap. The entire value prop is "fill forms in your language, from your documents." If a user uploads their Nepali birth certificate and gets "please type your name in English characters manually," that's a broken onboarding experience.

**Fix:** Move transliteration to M1, not post-MVP. At minimum:
- MRZ-first: if passport exists, use passport spelling (done, no new code)
- Aksharamukha candidate generation: 1-2 days of implementation
- The engine is well-designed — just build it earlier

### 2.8 ⚠️ HIGH: No Idempotency

| Action | What happens if retried? |
|--------|-------------------------|
| `POST /documents` (same file uploaded twice) | Two GCS objects, two `documents` rows |
| `POST /confirmations` (same answers sent twice) | Duplicate `user_entries`, duplicate facts |
| `POST /sessions` (client retries) | Multiple sessions for same intent |

**Fix:** 
- Documents: client generates upload hash; server deduplicates by `(uid, sha256)` 
- Confirmations: `request_id` is already in the schema — enforce idempotency key
- Sessions: client provides `idempotency_key`; server returns existing session_id on duplicate

### 2.9 ⚠️ HIGH: No Rate Limiting

The API has zero rate limiting. Attack surface:
- Scripted uploads burning Claude credits ($0.03/doc × 10,000 docs = $300)
- Session spam (create 1,000 sessions)
- Confirmation brute-force

**Fix:** Per-user rate limits enforced in FastAPI middleware:
- 10 sessions/day per user (free tier)
- 20 documents/session max
- 50 confirmations/session max
- 3 failed sessions/hour → cooldown

Use Cloud Memorystore (Redis) for rate limit counters, or an in-memory store if single-instance.

### 2.10 ⚠️ HIGH: No Session Cleanup

```python
# What happens when:
session = create_session()          # DB row created
upload_documents(session, [f1, f2]) # GCS objects created
# User closes browser tab — never sends /confirmations
# Session sits forever: DB row + GCS objects + no cleanup
```

**Fix:** 
- Session TTL: `sessions.status = 'abandoned'` after 24h of inactivity
- GCS lifecycle: auto-delete abandoned session documents after 30 days
- CronJob: `DELETE FROM sessions WHERE status='abandoned' AND created_at < NOW() - INTERVAL '30 days'`
- User-initiated `DELETE /v1/sessions/{id}` — cascade to GCS + DB

### 2.11 ⚠️ MEDIUM: No Caching Layer

Every `vault.recall(uid, required_keys)` hits Cloud SQL. For a new user with 0 facts: fine. For a returning user with 6 months of accumulated facts across 20+ documents: ~20 DB queries per session.

**Fix:** For MVP this is acceptable. For production:
- In-memory LRU cache per process (vault facts are small, immutable-after-write)
- Redis at 1K+ MAU
- Cache invalidation: `save_fact()` busts the per-user cache entry

### 2.12 ⚠️ MEDIUM: No Multi-Page Document Handling

Payslips and bank statements are frequently multi-page PDFs. Doc 02 has an open question about page-by-page vs stitched extraction. Neither is implemented.

**Fix:** 
- Client responsibility: split multi-page PDFs before upload (saathi-web does this)
- Server: accept PDF → extract page count → classify each page → merge extractions
- Bank statements: only extract the summary page (page 1), not every transaction page

### 2.13 ⚠️ MEDIUM: No HEIC Handling

iPhones default to HEIC. Doc 02 asks about server-side vs client-side conversion. Neither exists.

**Fix:** Client-side conversion (saathi-web converts HEIC → JPEG before upload). This keeps the manaslu API simpler (no image processing dependency). Add `pillow-heif` to saathi-web's dependencies.

### 2.14 ⚠️ MEDIUM: Spend Caps Not Implemented

Doc 05 describes per-user spend caps. No code exists.

**Fix:** Track `sessions.spend_tokens` (already in schema). Before each Claude call, check: `user_total_spend_this_month + estimated_call_cost > user_cap`. If so, return `quota/exceeded` — consumer renders "Free tier limit reached."

---

## 3. What The Docs Miss — Gaps In The Architecture Itself

### 3.1 No Load/Performance Targets

How many concurrent sessions can `resolve_form()` handle? What's the p95 latency of a full Form 80 fill? The docs specify SSE timeouts but not throughput.

**Add:** 
- Target: 50 concurrent sessions per instance before horizontal scaling
- Target: p95 session latency < 60s (upload → fill.completed)
- Load test: 10 concurrent sessions with 5 documents each before public launch

### 3.2 No Backward Compatibility Strategy

The API is versioned (`/v1`) but there's no deprecation policy, no migration path for consumers, no changelog format.

**Add:**
- Semantic versioning for the API contract
- 30-day deprecation notice for breaking changes
- Generated OpenAPI spec + changelog in repo

### 3.3 No Database Migration Testing

Alembic migrations run in CI before deploy. But there's no test that migrations don't break existing data.

**Add:** Migration test: restore a production backup → run new migration → verify data integrity.

### 3.4 No Disaster Recovery Plan

What happens when Cloud SQL goes down? What's the RTO (Recovery Time Objective)? The docs say "backups exist" but don't say how to restore them.

**Add:** 
- DR runbook: restore from backup → verify → update DNS → done
- RTO target: < 1 hour
- Rehearse quarterly

### 3.5 Missing: How To Handle Partially-Filled Forms

If a user fills Form 80, downloads the PDF, and then wants to edit one field and re-download — what happens?

Current design: each fill is a new `filled_forms` row. This is correct (immutable). But there's no "update one field and re-fill" flow.

**Add:** `POST /v1/sessions/{id}/fills` with `override_fields: {field_name: new_value}`. The engine re-runs `fill_pdf()` with overrides applied.

### 3.6 Missing: Form 80 + 1221 Are The Only Forms — What About Form 1085? 956? 47PA?

The architecture claims the manifest format supports arbitrary forms. But only 2 manifests are planned (Form 80, Form 1221). The `understand_form` tool for arbitrary forms is "stage 2."

For a saleable product, you need at least the top 5 most-used forms by your target audience:

| Form | Purpose | Used by |
|------|---------|---------|
| Form 80 | Character assessment | Almost all visa types |
| Form 1221 | Additional particulars | Commonly required |
| Form 1085 | Health undertaking | Health-related conditions |
| Form 956 | Appointment of migration agent | Anyone using an agent |
| Form 47PA | Partner visa application | Partner visa applicants |

Add at least Form 1085 and Form 956 to the post-MVP roadmap. They're simpler forms (fewer fields) and add real utility.

---

## 4. What Makes This Saleable — The Missing Product Layer

### 4.1 The Demo That Sells It

Doc 11 §1 defines the demo: "User fills Form 80 once; weeks later opens Form 1221 and watches it arrive ~80% pre-filled from their own vault."

**This demo requires:** Form 80 manifest + Form 1221 manifest + working extraction + working vault + working fill + a real user's documents. Currently: 0 of these exist.

**Milestone before anyone sees this:** Build the demo end-to-end. Screenshare it. If the vault-reuse moment doesn't elicit "wow," the architecture is correct but the product isn't.

### 4.2 Competitive Positioning Reality Check

Doc 11 positions against Instafill and FormMate80 with "vault + bilingual explain + AU depth + trust posture." But:

- **Instafill fills Form 80 today, for free, in English.** The user types their data into a web form and gets a filled PDF. It works. It's fast. It doesn't need Claude credits.
- **FormMate80 does the same, also free.**
- **The vault value prop only kicks in on the SECOND form.** Most users may only ever fill one form. If their first experience is "upload 3 documents, wait 30 seconds, confirm fields, download" vs Instafill's "type 8 fields, download instantly" — which wins?

**The vault only beats manual entry if:**
1. The user fills multiple forms (proven when they do)
2. The per-form time savings from the vault demonstrably exceed manual typing time
3. The extraction accuracy is high enough that review time is minimal

**Recommendation:** The first form's UX must be faster than typing — or the vault value prop never materializes. Time it. If a Form 80 takes > 3 minutes for a first-time user, you're losing to Instafill on the first impression. The vault might win them back on form 2, but they need to reach form 2.

### 4.3 The "Explain-While-Fill" Content Is The Product

Doc 11 F5: "NP explanations for ~140 Form 80 questions is days of careful bilingual writing." This is the single biggest schedule risk in the entire project — not code, content.

**140 fields × 5 minutes per explanation (write + translate + review) = 11.6 hours of focused bilingual writing.** This is 2-3 days of full-time work. It hasn't started.

**Recommendation:** Start writing NP explanations NOW, in parallel with M1 extraction. Use a spreadsheet: field_name | EN_label | NP_label | EN_explanation | NP_explanation | common_mistakes. Get a Nepali speaker to review. This gates the "explain-while-fill" feature which gates the demo which gates the sale.

### 4.4 D5 Anthropic DPA Is The Real Launch Blocker

Doc 10 §"The exception": "Before public launch: confirm Anthropic's data-retention terms for API traffic, whether a DPA/zero-retention arrangement applies, and disclose the processor in the privacy policy. **This is a launch blocker, not a build blocker.**"

This is not a technical problem, but it's the one that could kill the product. If Anthropic won't sign a DPA or won't commit to zero-retention for API traffic, you cannot legally process passport images through their API under Australian Privacy Act (APP 8 — cross-border disclosure).

**Recommendation:** Send the DPA inquiry to Anthropic THIS WEEK. It has the longest lead time of anything in the project and you can't work around a "no."

---

## 5. Implementation Priority (What To Build First)

Current build plan (doc 11): M0 → M1 → M2 → M3 → M4.

**Revised priority based on this audit:**

| # | Task | Why First | Current Status |
|---|------|-----------|---------------|
| 1 | **Collect 50-doc golden test set** | Everything else is unmeasurable without it | ❌ Not started |
| 2 | **Send Anthropic DPA inquiry** | Launch blocker with longest lead time | ❌ Not started |
| 3 | **D3 dump: extract Form 80 field names** | Gates manifest creation → gates fill | ❌ Not started |
| 4 | **Write NP explanations for Form 80 fields** | Gates explain-while-fill → gates demo | ❌ Not started |
| 5 | **Build extraction spine: classify + extract + validate** (M1) | Core engine — everything depends on it | ❌ Not started |
| 6 | **Build vault + profile_facts** (M1) | The moat — gates the demo | ❌ Not started |
| 7 | **Build Form 80 manifest + fill + annex** (M2) | First working end-to-end | ❌ Not started |
| 8 | **Build API + SSE + confirmations (v1)** | Consumer contract | ❌ Not started |
| 9 | **Build Form 1221 manifest** (M4) | The vault-reuse demo | ❌ Not started |
| 10 | **Implement error recovery + idempotency + rate limiting** (§2) | Production hardening | ❌ Not started |
| 11 | **Implement Document AI fallback** (§2.3) | Vendor risk mitigation | ❌ Not started |
| 12 | **Load test + DR rehearsal** | Production readiness | ❌ Not started |

---

## 6. Summary: What Must Change

| Category | Current State | Required State |
|----------|--------------|----------------|
| **Code** | Zero lines | Working extraction + fill for Form 80 |
| **Testing** | No test set | 50-doc golden set + per-field accuracy CI |
| **Error handling** | Happy path only | Retry, fallback provider, graceful degradation |
| **Observability** | Logging only | 8 custom metrics + Cloud Monitoring dashboard |
| **Vendor risk** | Anthropic-only | Document AI fallback implemented + tested |
| **Content** | Not started | 140 NP field explanations written + reviewed |
| **Compliance** | DPA not sent | Anthropic DPA confirmed |
| **Idempotency** | Not designed | Per-endpoint idempotency keys |
| **Rate limiting** | None | Per-user quotas + spend caps |
| **Session cleanup** | Not designed | TTL + cron cleanup + user-initiated delete |
| **Transliteration** | Post-MVP | M1 — required for Nepali birth certificates |
| **Multi-page docs** | Open question | Client-side splitting + server page handling |
| **HEIC** | Open question | Client-side conversion before upload |
| **DR** | No plan | Restore-from-backup runbook, rehearsed |

**Timeline impact:** The current M1-M4 plan (8 weeks) assumes the architecture is correct and implementation is straightforward. With the gaps above addressed, a realistic timeline is **10-12 weeks** to production-ready — assuming solo development.

---

*Audit compiled: August 2026 | Based on: manaslu/docs/architecture/* (11 docs) + ARCHITECTURE.md + README.md*  
*Commit: 2162c94 (gap-resolution engine refactor)*
