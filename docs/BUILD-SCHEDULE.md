# Saathi — Build Schedule & Development Plan

**Version:** 2.0 | **Date:** August 8, 2026

Complete implementation plan with phases, task breakdown, estimates, and dependencies.

> **Supersedes v1.0.** Stack corrected from Vercel/Railway/Supabase to GCP-native
> (matching manaslu's infra pattern), and Phase 4 rewritten: saathi no longer
> builds the scan/extract/fill pipeline from scratch — it integrates with
> `manaslu`, a separate headless backend that owns that skill. See
> `docs/architecture/f4-manaslu-integration.md` for the API contract.

---

## Dependency: manaslu

F4 (Form Helper) has two halves with very different dependency profiles:

- **F4a (field explanation / knowledge-service)** is saathi's own build — no
  dependency on manaslu. Can start any time after Phase 0.
- **F4b (scan → extract → validate → transliterate → fill)** is no longer
  built here. It's `manaslu`'s responsibility, shipped on **manaslu's own
  M0–M4 timeline, in its own repo** (`manaslu/docs/architecture/11-mvp-build-plan.md`).
  Saathi's F4b integration work (review UI, SSE handling, confirmations,
  transliteration picker) can only be built meaningfully once manaslu's
  **M3 — agent loop + API** milestone is reachable (full `/v1/sessions`
  REST+SSE surface live). Until then, F4b tasks below are blocked on an
  external repo, not on anything in this schedule.

**This does not gate the rest of saathi.** F1 (Tracker), F2 (Calculator), and
F3 (Checklist) have zero manaslu dependency and proceed on their own schedule
regardless of manaslu's progress. Only the F4b sub-phase waits.

---

## Timeline Overview

```
Phase 0: Foundation        Weeks 1-2    → GCP infra, schema, auth, CI/CD
Phase 1: F3 Checklist      Weeks 2-3    → First shippable feature
Phase 2: F2 Calculator     Weeks 3-4    → Second feature
Phase 3: F1 Visa Tracker   Weeks 4-6    → Third feature + reminders
Phase 4a: F4 Knowledge Svc Weeks 6-7    → RAG / field explanations (saathi's own build)
Phase 4b: F4 Manaslu Integ Weeks 7-9*   → Review UI + SSE + fill flow (*gated on manaslu M3, see above)
Phase 5: Polish            Weeks 9-11   → Testing, legal, beta launch
Phase 6: F5 News & Events  post-beta    → Traction-gated; designs done, ~8.5 dev-days when triggered
Phase 7: F6 Agent Connect  post-beta    → Traction + legal-gated; designs done, ~9 dev-days + agent recruitment

Total: ~11 weeks to beta-ready product, assuming manaslu M3 lands inside this window.
If manaslu is not yet at M3 when saathi reaches Phase 4b, ship Phase 5 for
F1/F2/F3/F4a first and slot F4b in as a fast-follow once manaslu is ready.
```

## Resource Assumptions
- **1 full-time developer** (solo/small team)
- **Part-time content curator** for Nepali translations + knowledge base maintenance
- **Legal review** (migration lawyer opinion) — outsourced, budget AU$500-1500
- GCP project(s) for saathi (dev/prod) provisioned via Terraform in
  `karki-labs-infra`, same pattern already used for manaslu
- Estimates assume focused engineering; multiply by 1.5-2x if also doing PM/design/QA

---

## Phase 0 — Foundation (Weeks 1-2)

**Goal:** Bootable dev environment, database schema, auth working — GCP-native, same deploy pattern as manaslu.

| ID | Task | Est. | Depends On | Deliverable |
|----|------|------|------------|-------------|
| P0-1 | Monorepo init: Next.js (App Router) + lightweight API layer (FastAPI) skeleton, both Cloud Run-shaped from day one | 1 day | — | `npm run dev` boots web + api locally |
| P0-2 | Terraform in `karki-labs-infra`: Cloud Run services (web, api), Cloud SQL Postgres instance (pgvector extension enabled), GCS bucket, VPC connector — dev project | 1.5 days | P0-1 | `terraform apply` provisions dev resources |
| P0-3 | Schema migration (Alembic): `users`, `visas`, `checklist_sessions` tables against Cloud SQL, run via Cloud SQL Auth Proxy locally | 1 day | P0-2 | DB running locally + in dev, migrations applied |
| P0-4 | shadcn/ui + Tailwind + i18n setup (next-intl, EN/NP) | 1 day | P0-1 | Bilingual shell rendering |
| P0-5 | GCP Identity Platform: email + Google OAuth sign-in, ID token verification middleware in the API layer (resource-server model — same shape as manaslu's own auth, see manaslu doc 07) | 1 day | P0-2 | Login/register working, ID token verified server-side |
| P0-6 | API layer skeleton with health endpoint, Cloud SQL client | 0.5 day | P0-3 | `/api/health` returns 200 |
| P0-7 | CI/CD: GitHub Actions + Workload Identity Federation (no stored keys), Docker build → Artifact Registry (australia-southeast1) → deploy Cloud Run dev (auto) → manual approval → prod — mirrors manaslu's pipeline exactly (manaslu doc 09) | 1.5 days | P0-1 | Push to main → auto-deploy to dev |
| P0-8 | Design system tokens, bilingual components (Disclaimer, LanguageToggle, ConfidenceBadge) | 1 day | P0-4 | Shared component library |
| P0-9 | Observability: Cloud Logging (structured JSON) + Error Reporting wired for both Cloud Run services | 0.5 day | P0-1 | Errors visible in Cloud Console |

**Phase 0 exit criteria:**
- Fresh clone + `npm install` + Cloud SQL Auth Proxy boots everything locally
- Auth working via GCP Identity Platform, ID tokens verified server-side
- CI/CD deploys to Cloud Run dev on push to main, same pattern as manaslu

---

## Phase 1 — F3 Document Checklist (Weeks 2-3)

| ID | Task | Est. | Depends On |
|----|------|------|------------|
| P1-1 | Checklist DB schema: `checklist_sessions` table (already scaffolded in P0-3), row-level user scoping | 1 day | P0-3 |
| P1-2 | API endpoint `/checklist`: session CRUD | 1 day | P1-1 |
| P1-3 | Visa type selection UI (card grid, 6 types with EN/NP labels) | 1 day | P0-8 |
| P1-4 | Branching questionnaire component (step-by-step, progress indicator) | 1 day | P1-3 |
| P1-5 | Decision tree engine: JSON rules → checklist generator | 1 day | P1-4 |
| P1-6 | Checklist renderer (categorized accordion, status badges, progress bar) | 1 day | P1-5 |
| P1-7 | Checklist item detail modal (expanded view: what/how/common mistakes) | 0.5 day | P1-6 |
| P1-8 | Print/export checklist as PDF | 0.5 day | P1-7 |
| P1-9 | Populate knowledge base: all checklist items for 6 visa types (EN+NP) | 2 days | P1-4 |
| P1-10 | Test all branching paths for all 6 visa types | 1 day | P1-9 |

**Exit:** User selects visa type → answers 3-5 questions → gets correct, bilingual checklist → prints it

---

## Phase 2 — F2 Points Calculator (Weeks 3-4)

| ID | Task | Est. | Depends On |
|----|------|------|------------|
| P2-1 | Points rules JSON schema + all 12 criteria defined in `knowledge/points-criteria/` | 1 day | — |
| P2-2 | Points calculation engine (Python, deterministic, with test suite) | 1 day | P2-1 |
| P2-3 | API endpoint `/calculator`: validate + compute | 0.5 day | P2-2 |
| P2-4 | Step-by-step wizard UI (progress bar, back/next, 12 questions) | 2 days | P0-8 |
| P2-5 | Question components: radio groups, dropdowns with points shown inline | 1 day | P2-4 |
| P2-6 | English test score mapper (IELTS/PTE/TOEFL → points auto-lookup) | 0.5 day | P2-5 |
| P2-7 | Results screen: score visualization, breakdown, SkillSelect comparison | 1 day | P2-4 |
| P2-8 | Save/load calculations (Cloud SQL, user-scoped) | 0.5 day | P2-7 |
| P2-9 | "How to improve" suggestions (logic: identify max gainable points) | 0.5 day | P2-2 |
| P2-10 | Test suite: 10 known profiles verified for 100% correctness | 1 day | P2-2 |

**Exit:** Calculator produces correct score for all 10 test profiles. Passes CI on every push.

---

## Phase 3 — F1 Visa Tracker (Weeks 4-6)

| ID | Task | Est. | Depends On |
|----|------|------|------------|
| P3-1 | Visas DB schema: `visas` table (already in P0-3, verify row-level scoping) | 0.5 day | P0-3 |
| P3-2 | API endpoint `/tracker`: visa CRUD | 1 day | P3-1 |
| P3-3 | Add visa wizard (3 steps: subclass → dates → conditions) | 1.5 days | P0-8 |
| P3-4 | Visa subclass UI (card grid with icons, auto-populated conditions) | 1 day | P3-3 |
| P3-5 | Dashboard: countdown timer component, condition cards, next-steps | 2 days | P3-3 |
| P3-6 | Expiry warning state (< 30 days): red-amber theme, actionable steps | 0.5 day | P3-5 |
| P3-7 | Multi-visa support: list view + individual cards | 0.5 day | P3-5 |
| P3-8 | Cloud Scheduler reminder job: daily HTTP trigger → API endpoint checks 180/90/30/7 day thresholds | 1 day | P3-2 |
| P3-9 | FCM push integration: register, send, handle click (replaces the Web Push API / OneSignal debate — one ADR, "start direct, add only when it hurts") | 1.5 days | P3-8 |
| P3-10 | Service Worker: cache visa data for offline viewing | 1 day | P3-5 |
| P3-11 | Notification preferences UI (enable/disable per threshold) | 0.5 day | P3-9 |

**Exit:** User can track multiple visas, receive push reminders, view tracker offline.

---

## Phase 4a — F4 Knowledge Service (RAG / field explanations) (Weeks 6-7)

This is still **saathi's own build** — manaslu is scoped to scan+fill, not
explain. No manaslu dependency; can start any time after Phase 0, in parallel
with Phase 1-3.

| ID | Task | Est. | Depends On |
|----|------|------|------------|
| P4A-1 | Knowledge ingestion scraper: fetch Home Affairs pages, change detection | 2 days | — |
| P4A-2 | Chunking + embedding pipeline (EN+NP), store in Cloud SQL pgvector table (extension already enabled in P0-2) | 1.5 days | P0-2 |
| P4A-3 | RAG retrieval endpoint: query → embedding → pgvector search → top-3 chunks | 1 day | P4A-2 |
| P4A-4 | Claude explanation service: RAG context + form field → bilingual explanation | 1 day | P4A-3 |
| P4A-5 | Form selection UI (card grid with field counts, descriptions) | 1 day | P0-8 |
| P4A-6 | Field explainer UI (chat-like, citation footer, feedback buttons) | 2 days | P4A-4, P4A-5 |

**Exit:** User gets a bilingual, cited explanation for any form field, independent of whether F4b (fill) is available yet.

---

## Phase 4b — F4 Manaslu Integration (scan → review → fill) (Weeks 7-9, gated on manaslu M3)

**This replaces the from-scratch scan/classify/extract/validate/fill build.**
The hardest engineering here — document classification, schema + open
extraction, MRZ/date/ABN/BSB validation, transliteration, AcroForm fill —
moved to `manaslu`'s own independent build and is documented there, not
duplicated here. See `docs/architecture/f4-manaslu-integration.md` for the
API contract and `manaslu/docs/architecture/` (docs 02, 03, 04, 06, 11) for
the pipeline itself.

Saathi's job is materially smaller: consume manaslu's API, render the review
UI its events are designed for, and forward identity correctly.

| ID | Task | Est. | Depends On |
|----|------|------|------------|
| P4B-1 | Session bootstrap: API layer creates a manaslu session (`POST /v1/sessions`), attaching the Cloud Run service-to-service ID token (IAM invoker) and forwarding the end-user's Identity Platform JWT | 1 day | P0-5, manaslu M3 |
| P4B-2 | SSE consumption: relay/subscribe to `POST /v1/sessions/{id}/messages` stream — handle `message.delta`, `tool.started`/`tool.finished`, `session.done`/`session.error` | 1 day | P4B-1 |
| P4B-3 | Upload UX: signed-URL handshake to `POST /v1/sessions/{id}/documents`, multi-file, progress | 1 day | P4B-1 |
| P4B-4 | Review UI: renders `extraction.ready` — side-by-side value ↔ source-crop, confidence badges, bilingual labels from the field manifest payload | 2 days | P4B-2 |
| P4B-5 | Confirmations flow: renders `review.required`, posts to `/v1/sessions/{id}/confirmations`, handles pending-state recovery via `GET /v1/sessions/{id}` on reconnect | 1 day | P4B-4 |
| P4B-6 | Transliteration picker sub-screen: candidate Latin spellings for Devanagari values, user selects/edits, submits as a confirmation | 1 day | P4B-5 |
| P4B-7 | Artifact download UI: renders `fill.completed`, fetches filled PDF + audit annex from `/v1/sessions/{id}/artifacts/{artifact_id}` | 0.5 day | P4B-2 |
| P4B-8 | Error/guardrail handling: RFC 7807 problem+json rendering, distinct messaging for `guardrail/advice-boundary` declines (MARN handoff prompt) | 0.5 day | P4B-2 |
| P4B-9 | End-to-end integration test against manaslu's dev environment: upload passport → review in NP → confirm → download filled Form 80 | 1 day | P4B-1 to P4B-8 |

**Exit:** User can upload documents to manaslu, review/confirm extractions (including transliteration choices) in saathi's UI, and download a filled PDF — with saathi never re-implementing extraction or fill logic itself.

**Not blocked on Phase 4a** — 4a and 4b can be built in either order; they only share the "Form Helper" umbrella, not code.

---

## Phase 5 — Polish & Launch (Weeks 9-11)

| ID | Task | Est. | Depends On |
|----|------|------|------------|
| P5-1 | Accessibility audit: WCAG 2.1 AA, axe DevTools scan, keyboard navigation | 1 day | All features |
| P5-2 | Performance optimization: Lighthouse audit, bundle size, lazy loading | 1 day | All features |
| P5-3 | PWA installability: manifest.json, service worker, offline fallbacks | 0.5 day | P3-10 |
| P5-4 | Error state hardening: every screen tested with network offline, API down, manaslu unavailable/timeout | 1 day | All features |
| P5-5 | Content freshness alert: flag knowledge base items > 90 days stale | 0.5 day | P4A-2 |
| P5-6 | Privacy policy, T&Cs, age gate (16+), data deletion flow | 1 day | — |
| P5-7 | Legal review: migration lawyer confirms F4 does not constitute immigration assistance | External | P4B-9 |
| P5-8 | Beta testing: 10-20 Nepalese migrants, feedback collection, bug fixes | 1 week elapsed | P5-1 to P5-6 |
| P5-9 | Landing page (saathi.app) with waitlist, Nepali + English | 1 day | — |
| P5-10 | Launch: Product Hunt, Nepalese Facebook groups, Reddit r/Nepal | 1 day | P5-8 |

**Note:** if manaslu M3 hasn't landed by Week 9, ship Phase 5 for F1/F2/F3/F4a
and treat P4B + its dependents (P5-4's manaslu-timeout case, P5-7's F4 scope)
as a fast-follow release once manaslu is ready, rather than delaying the
whole beta.

---

## Phase 6 — F5 News, Seminars & Opportunities (post-beta, traction-gated)

**Not scheduled into the 11 weeks.** F5 (PRD §4: Home digest, news feed with AI
Nepali summaries, MARN-verified seminar listings, student corner) begins only
once the core four features show traction per PRD §9's discipline. Designs are
already complete (`diagrams/saathi-screen-designs.html` §F5, `ui-ux-flows.md`
§11), so this phase is engineering + ops, not design.

| ID | Task | Est. | Depends On |
|----|------|------|------------|
| P6-1 | `news_items` + `events` schema; extend the P4A-1 ingest worker with the news-source allowlist | 1 day | P4A-1 |
| P6-2 | Haiku Batch summarisation job (headline + ≤2-sentence NP/EN summary, labelled) | 1 day | P6-1 |
| P6-3 | Home tab: digest card stack (tracker → resume-session → news → event); tab bar 4→5 | 1.5 days | P3-5 |
| P6-4 | News feed + detail screens, category chips, follow-topic push tags | 1.5 days | P6-2 |
| P6-5 | Events: admin curation sheet → table, list/detail screens, MARN-verification field (required for migration topics), .ics + FCM reminders | 2 days | P6-1 |
| P6-6 | Student corner (deadline-first listing view, reuses F3 content contract) | 1 day | P6-5 |
| P6-7 | Ops runbook: weekly curation checklist, MARN check procedure, copyright posture sign-off from the P5-7 legal reviewer | 0.5 day + external | P5-7 |

**Exit:** feed ships ≥5 fresh items/week; every migration seminar shows a verifiable MARN; zero full-text reproductions; curation time ≤2 hrs/week.

---

## Phase 7 — F6 Connect to an Agent (post-beta, traction-gated, EN-only)

**Not scheduled into the 11 weeks.** The PRD §7 referral surface (F6, PRD §4).
Gated on: core traction + P5-7's legal review extended to cover the APP 6
third-party-disclosure consent framework and the referral-fee arrangement
(ARCHITECTURE.md risk #11). Designs complete
(`diagrams/saathi-screen-designs.html` §F6, `ui-ux-flows.md` §12).

| ID | Task | Est. | Depends On |
|----|------|------|------------|
| P7-1 | Schema: `agents`, `enquiries`, `enquiry_consents` + audit-log wiring | 1 day | P0-2 |
| P7-2 | MARA verification job: onboarding check + quarterly re-verify, auto-delist on lapse | 1 day | P7-1 |
| P7-3 | Directory + profile screens (filters, verify-link block, disclosure copy) | 1.5 days | P7-1 |
| P7-4 | Enquiry flow: form → itemised consent review → send; consent recording | 2 days | P7-3 |
| P7-5 | Agent-side minimal surface: transactional email + token-authenticated response page (view, mark responded, propose call time) | 2 days | P7-4 |
| P7-6 | My Enquiries: status timeline, accept call time (.ics + FCM), revoke round-trip (agent notification + closure + audit) | 1.5 days | P7-5 |
| P7-7 | Agent onboarding ops: terms (use-once, delete-on-revoke, no re-marketing), recruitment of first 5-10 agents | External | P7-2 |

**Exit:** every listed MARN independently verifiable; zero shares without an itemised consent record; revoke round-trip works end-to-end; referral disclosure on profile + send; first 5 agents live.

---

## Critical Path

```
P0-1 → P0-2 → P0-5 (Auth)
              → P1-1 → P1-2 (F3 Checklist API)
              → P3-1 → P3-2 (F1 Tracker API)
              → P4A-2 → P4A-3 (RAG vector store)

The longest in-repo chain is P0-1 → P4A-2 → P4A-3 → P4A-4 → P4A-6 (F4a knowledge service)
P4b's chain (P4B-1 → … → P4B-9) is short in engineering terms but has an
**external gate**: it cannot start in earnest before manaslu reaches M3 in
its own repo, regardless of how far ahead saathi's other phases are.
```

## Parallelization Opportunities

| Can run in parallel | When |
|--------------------|------|
| F3 Checklist + F2 Calculator | Both are independent after Phase 0 |
| Knowledge base population (P1-9) | Start immediately, runs alongside coding |
| F4a knowledge service (P4A-1 to P4A-6) | Can start after Phase 0, no dependency on F1-F3 or on manaslu |
| Landing page (P5-9) | Anytime — no dependency on product features |
| Legal review (P5-7) | Start as early as possible — has external lead time |
| F4b (P4B-*) | Whenever manaslu M3 is reachable — may land before, during, or after Phase 3 depending on manaslu's own schedule |

## Cost Tracking

GCP pricing shape, replacing the old Vercel/Railway/Supabase table. The one
material difference from the old plan: **Cloud SQL has no meaningful free
tier** (unlike Supabase) — it's a small always-on cost even at zero traffic.
Everything else (Cloud Run, GCS, Identity Platform, Artifact Registry) scales
to near-zero at low volume the same way the old stack did.

| Phase | External Cost (Monthly) |
|-------|--------------------------|
| Phase 0-2 | Cloud Run: $0 (scale-to-zero, generous free tier) · Cloud SQL: **~$25-70/mo** (smallest always-on instance — see flag below) · GCS: <$1 · Identity Platform: $0 (free tier to 50K MAU) |
| Phase 3-4a | + Claude API ~$5/mo for testing F4a explanations (RAG). F4b's scan/extract/fill Claude usage is billed inside **manaslu's own GCP project**, not saathi's — a real cost-boundary benefit of not rebuilding that pipeline here. |
| Phase 5 | ~$10-15/mo beta users' Claude API (F4a explanations only) |
| Post-launch | ~$60-100/mo at 1K MAU — Cloud SQL's fixed cost is now the dominant line item, not Claude API (which scales with usage but stays cheap per call) |

**⚠️ Flag, not a firm number:** the old "under $100 total POC cost" claim
assumed Supabase's free-tier Postgres. Cloud SQL's cheapest always-on
instance alone is plausibly $25-70/month depending on tier/region/HA
settings — get an exact quote from the GCP pricing calculator before
committing to a budget line. One lever worth considering: share a single
Cloud SQL instance (multiple databases) between saathi and manaslu's
dev/staging environments in `karki-labs-infra`, rather than provisioning two.
If that's viable, total POC cost over the build window is still plausibly
under $150-200; if not, budget closer to $250-300 for the 11 weeks.

---

*Schedule compiled: August 8, 2026. Rewritten from v1.0 per the saathi/manaslu unification brief (D-B, D-C).*
