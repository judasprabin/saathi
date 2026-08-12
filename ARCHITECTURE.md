# Saathi — Master Architecture Document

**Project:** AI Settlement & Immigration Companion for Nepalese Diaspora in Australia
**Version:** 2.0 — Unified with manaslu (GCP-native, F4 delegated to a separate agent service)
**Date:** August 8, 2026
**Author:** Prabin Karki
**Status:** Architecture approved — ready for implementation

> This is the single source of truth for Saathi's architecture. Saathi is one product built across **two repos**: this repo (frontend + F1/F2/F3 + F4 integration) and [`manaslu`](../../manaslu) (the headless scan/extract/form-fill agent service that F4 consumes). See §1 for the boundary. Component-specific detail files live in `docs/architecture/`.

---

## Table of Contents

1. [System Overview](#1-system-overview)
2. [Architecture Principles](#2-architecture-principles)
3. [Repository Structure](#3-repository-structure)
4. [System Context Diagram](#4-system-context-diagram)
5. [Container Architecture](#5-container-architecture)
6. [Data Architecture](#6-data-architecture)
7. [Feature Architecture — F1 Visa Tracker](#7-f1-visa-tracker)
8. [Feature Architecture — F2 Points Calculator](#8-f2-points-calculator)
9. [Feature Architecture — F3 Document Checklist](#9-f3-document-checklist)
10. [Feature Architecture — F4 Form Helper](#10-f4-form-helper)
11. [RAG & Knowledge Architecture](#11-rag--knowledge-architecture)
12. [Infrastructure & Deployment](#12-infrastructure--deployment)
13. [Security Architecture](#13-security-architecture)
14. [Architecture Decision Records](#14-architecture-decision-records)
15. [Build Schedule](#15-build-schedule)
16. [Open Questions & Risks](#16-open-questions--risks)

---

## 1. System Overview

### What Saathi Is

A focused, bilingual (English/Nepali) PWA that helps Nepalese migrants in Australia navigate the immigration system. Positioning, corrected against the July 2026 competitive landscape (`docs/research/market-and-competitive-analysis.md`): mechanical form-filling is already commoditized (Instafill has a dedicated Form 80 page; FormMate80 does it free). Saathi's headline is **"understand and complete your visa forms in your own language, from your own documents"** — the moat is a persistent profile vault (fill once, every later form starts ~pre-filled), bilingual field-by-field explanation, AU-migration depth, and a fill-only trust posture, not raw fill speed.

| # | Feature | What It Does | Complexity | Owning repo |
|---|---------|-------------|------------|-------------|
| F1 | **Visa Tracker** | Track visa expiry, conditions, and deadlines | Low | saathi |
| F2 | **Points Calculator** | Calculate skilled migration points score | Medium | saathi |
| F3 | **Document Checklist** | Generate personalised visa document checklists | Medium | saathi |
| F4a | **Form Explainer** | Explain immigration form fields in Nepali (RAG) | Medium | saathi |
| F4b | **Scan & Form-Fill** | Scan documents → vault → auto-fill AcroForm PDF | High | **manaslu** (consumed via API) |
| F5 | **News, Seminars & Opportunities** *(Phase 2, traction-gated)* | Home digest + curated visa news (AI Nepali summaries, link-out only) + MARN-verified seminars + student opportunities | Low-Med | saathi |
| F6 | **Connect to an Agent** *(Phase 2, traction-gated, EN-only UI)* | MARA-verified agent directory + enquiry/request-a-call + itemised share-consent + revocation. The PRD §7 referral surface | Medium | saathi |

### The saathi / manaslu boundary

**manaslu is a separate, headless agent service** — no UI, no end-user identity, scoped to exactly one skill: scan a user's documents, maintain a persistent provenance-tracked profile vault, and fill Form 80 / Form 1221 AcroForm fields from it. It exposes a REST + SSE API. Saathi is its only consumer at MVP.

```
saathi (this repo)                          manaslu (separate repo)
─────────────────────                       ──────────────────────────
Next.js PWA (all 4 features' UI)            Headless FastAPI agent service
F1 Tracker, F2 Calculator,     ◄── owns ──  F4b: classify/extract/validate/
F3 Checklist — full stack                   transliterate/map/fill + vault
F4a Form Explainer (RAG,                    (profile_facts table, gap-
 own knowledge-service)                      resolution engine, audit log)
F4b UI: renders manaslu's                   Consumers forward the
 session/review/confirm events    ──API──►  end-user JWT; manaslu verifies
 (side-by-side value↔source-crop,           it, never issues its own
 transliteration picker)                    identity — see manaslu doc 07
```

**Why this split, not one rebuild:** manaslu already exists, is GCP-native, and was designed post-competitive-analysis specifically around the vault moat (see `manaslu/docs/architecture/11-mvp-build-plan.md`). Rebuilding scan/extract/fill inside saathi would duplicate ~4-6 weeks of already-solved, harder engineering (Devanagari transliteration, AcroForm provenance contracts, confidence tiering) for no benefit. Saathi's job for F4b is thin: call the API, render what it returns, forward auth.

### What Saathi Is NOT

- A migration agent — does not lodge applications or give advice
- An ImmiAccount integration — does not connect to Home Affairs systems
- A marketplace (initially) — no agent matching until traction is proven
- A community platform — focused purely on utility tools
- A generic/horizontal form filler — see competitive positioning above; this is explicitly not the wedge

### Regulatory Boundary (Non-Negotiable)

```
✅ ALLOWED: explaining form fields, showing required documents, calculating points,
            tracking user-entered dates, translating official content into Nepali,
            transcribing values from the user's own documents into forms (manaslu, fill-only)

❌ PROHIBITED: assessing eligibility, recommending visa pathways, lodging applications,
              preparing forms on a person's behalf, giving migration advice

🔄 HANDOFF: whenever a question crosses into advice territory → "Consult a registered
            migration agent (verify at mara.gov.au using their MARN)"
```

**Legal status — do not overstate:** `docs/legal/legal-memo.md`'s verdict is **"GO-WITH-CONDITIONS,"** not an unconditional clearance. Condition 4 (a written opinion from a MARN-registered migration agent confirming autofill doesn't constitute "immigration assistance" under Migration Act 1958 s.276) has **not yet been obtained**. This gates public launch of F4b, not development — see §16.

---

## 2. Architecture Principles

1. **Bilingual by default** — Every UI label, explanation, and error message exists in English AND Nepali. No English-only dead ends.

2. **Cite everything** — Every checklist item, calculator rule, and form explanation carries a source URL and "last verified" date. Stale information is more dangerous than no information.

3. **Fail safe, not silent** — Every AI call has a confidence threshold. Low-confidence outputs are flagged, not auto-accepted. The user always has a manual fallback path.

4. **Cheap until proven** — Use GCP's free/low tiers wherever possible (Cloud Run scale-to-zero, Cloud SQL smallest tier). Only pay when traction justifies it.

5. **Progressive disclosure** — Don't overwhelm new users with all 4 features. Onboard one feature at a time with clear value demonstration.

6. **Offline-capable where possible** — F1 (Tracker) and F2 (Calculator) work without internet. F3 (Checklist) and F4 (Form Helper) require connectivity for AI calls.

7. **RLS-equivalent everywhere** — Every user-scoped Cloud SQL table enforces row ownership (`owner_uid` scoping in every query, mirroring manaslu's model in its doc 07) from day one. Users can only access their own data.

8. **One cloud, one auth system** — Everything lives in GCP; the end-user identity token (GCP Identity Platform) is the same token forwarded to manaslu, verified there as a resource server. No auth bridging between clouds.

9. **Knowledge is always fresh** — All content (checklist items, calculator rules, form explanations, visa conditions) is sourced from the AU Visa Source Registry crawler, which verifies 19 government domains daily. Every explanation carries a "last verified" timestamp and source URL. Stale data (>90 days) is flagged automatically.

---

## 3. Repository Structure

```
saathi/                              # This repo — frontend + F1/F2/F3 + F4 integration
├── web/                             # Next.js PWA (frontend)
│   ├── app/                         # App Router pages
│   │   ├── (auth)/                  # Login/register (GCP Identity Platform)
│   │   ├── (dashboard)/             # Main app shell
│   │   │   ├── tracker/             # F1 — Visa Tracker
│   │   │   ├── calculator/          # F2 — Points Calculator
│   │   │   ├── checklist/           # F3 — Document Checklist
│   │   │   ├── form-helper/         # F4 — Form Explainer (own) + Scan/Fill (renders manaslu API)
│   │   │   └── settings/            # Profile, language toggle
│   │   └── api/                     # Next.js API routes (BFF — forwards JWT to manaslu, see §10)
│   ├── components/
│   │   ├── ui/                      # Design system (shadcn/ui based)
│   │   ├── features/                # Feature-specific components
│   │   │   ├── tracker/
│   │   │   ├── calculator/
│   │   │   ├── checklist/
│   │   │   └── form-helper/         # incl. manaslu-review/, transliteration-picker/
│   │   └── shared/                  # Shared: LanguageToggle, Disclaimer, etc.
│   ├── lib/
│   │   ├── auth.ts                  # GCP Identity Platform client
│   │   ├── manaslu-client.ts        # Generated TS client for manaslu's API (from its OpenAPI spec)
│   │   ├── i18n.ts                  # Internationalization (EN/NP)
│   │   └── constants.ts             # Visa types, form lists, etc.
│   ├── public/
│   │   └── locales/                 # EN/NP translation files
│   └── next.config.js
│
├── api/                             # FastAPI — F1/F2/F3 CRUD + F4a RAG (F4b lives in manaslu)
│   ├── app/
│   │   ├── routers/
│   │   │   ├── tracker.py           # F1 — visa CRUD + reminder scheduling
│   │   │   ├── calculator.py        # F2 — points calculation endpoint
│   │   │   ├── checklist.py         # F3 — checklist generation endpoint
│   │   │   ├── explain.py           # F4a — RAG field explanation
│   │   │   └── health.py            # Health check
│   │   ├── services/
│   │   │   ├── claude_client.py     # Claude text API wrapper (F4a explanations only — no vision here)
│   │   │   ├── rag_service.py       # Vector search + LLM grounding
│   │   │   └── knowledge_refresh.py # Scheduled ingestion of Home Affairs pages
│   │   ├── schemas/
│   │   │   ├── forms.py             # Form-field manifests (labels/explanations — NOT AcroForm mapping, that's manaslu's)
│   │   │   └── points.py            # Points calculator rules schema
│   │   └── core/
│   │       ├── config.py            # Settings from env vars
│   │       ├── db.py                # Cloud SQL client (via Cloud SQL connector)
│   │       └── errors.py            # Typed error responses
│   ├── tests/
│   └── requirements.txt
│
├── db/
│   └── migrations/                  # Alembic migrations — Cloud SQL Postgres
│       ├── 001_users.sql            # uid mirror + profile prefs (identity itself lives in Identity Platform)
│       ├── 002_visas.sql            # Visa tracker tables
│       ├── 003_checklists.sql       # Checklist tables
│       └── 004_knowledge_base.sql   # RAG knowledge base tables (pgvector)
│
├── knowledge/                       # Curated knowledge base (F4a explanations, F3 checklist content)
│   ├── visa-types/                  # Visa conditions + requirements
│   │   ├── 500-student.json
│   │   ├── 485-graduate.json
│   │   ├── 189-skilled-independent.json
│   │   └── ...
│   └── points-criteria/             # Points calculator rules
│       └── gsm-points.json
│
├── docs/
│   ├── README.md                    # Project overview
│   ├── PRD.md                       # Product requirements
│   ├── ARCHITECTURE.md              # This file
│   ├── BUILD-SCHEDULE.md            # Phase-by-phase build plan
│   ├── architecture/
│   │   ├── ui-ux-flows.md           # Canonical UI/UX flows (all features)
│   │   ├── f4-manaslu-integration.md# Saathi's side of the F4b contract (see manaslu/docs/architecture/ for the pipeline itself)
│   │   └── archive/                 # Superseded drafts, kept for history
│   ├── research/
│   │   ├── market-and-competitive-analysis.md  # Consolidated market + competitive research
│   │   └── archive/                 # Superseded drafts
│   └── legal/
│       ├── legal-memo.md            # Visa form-fill legality analysis
│       └── fallback-answer-sheet.md # Legal fallback architecture
│
├── diagrams/
│   └── saathi-ui-mockup.html        # Interactive HTML mockup, all 4 feature screens
│
├── .env.example                     # Environment variables template
└── README.md
```

**Related repo:** [`manaslu`](../../manaslu) — headless scan/extract/form-fill agent (F4b). Its own architecture docs (`manaslu/docs/architecture/`, 11 documents) are the source of truth for that pipeline; not duplicated here.

---

## 4. System Context Diagram

```
                         ┌───────────────────────────────────┐
                         │        SAATHI (this repo)          │
                         │                                    │
  ┌──────────┐           │  ┌──────────────────────────────┐  │
  │ Nepalese │           │  │   Next.js PWA                 │  │
  │ Migrant  │──HTTPS───▶│  │   (mobile-first, bilingual)   │  │
  │ in AU    │           │  │                               │  │
  └──────────┘           │  │ F1 Tracker   F2 Calculator    │  │
                         │  │ F3 Checklist F4a Explain      │  │
                         │  │ F4b Scan/Fill (renders manaslu│  │
                         │  │      session events)          │  │
                         │  └─────────────┬─────────────────┘  │
                         │                │                     │
                         │     ┌──────────▼─────────────────┐   │
                         │     │  Cloud Run — FastAPI        │   │
                         │     │  /tracker /calculator       │   │
                         │     │  /checklist /explain (RAG)  │   │
                         │     └──────────┬─────────────────┘   │
                         │                │                     │
                         │     ┌──────────▼─────────────────┐   │
                         │     │  Cloud SQL (Postgres        │   │
                         │     │  + pgvector) · GCS           │   │
                         │     │  · Identity Platform         │   │
                         │     └─────────────────────────────┘   │
                         └───────────────────┬─────────────────┘
                                              │ REST + SSE (v1)
                                              │ end-user JWT forwarded
                         ┌───────────────────▼─────────────────┐
                         │     MANASLU (separate repo)          │
                         │  Gap-resolution engine (Cloud Run)   │
                         │  scan → extract → validate →         │
                         │  transliterate → map → fill          │
                         │  profile_facts vault · audit log     │
                         │  Cloud SQL (own DB) · GCS (AU)       │
                         └───────────────────┬─────────────────┘
                                              │
                    ┌─────────────────────────┼──────────────────────┐
                    ▼                         ▼                      ▼
           ┌──────────────┐         ┌──────────────────┐   ┌──────────────┐
           │ Claude Opus/  │         │ Home Affairs      │   │ SkillSelect   │
           │ Sonnet/Haiku  │         │ (immi.gov.au)      │   │ (live data)   │
           │ (Anthropic)   │         │ Source of truth    │   │ Invitation    │
           │               │         │ for all content     │   │ rounds data   │
           └──────────────┘         └──────────────────┘   └──────────────┘
```

---

## 5. Container Architecture

### Service Boundaries

```
┌─────────────────────────────────────────────────────────────────────┐
│                      CLIENT (Next.js PWA)                            │
│                                                                      │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────────────┐   │
│  │ F1       │ │ F2       │ │ F3       │ │ F4                   │   │
│  │ Tracker  │ │ Calc     │ │ Checklist│ │ Form Helper          │   │
│  │          │ │          │ │          │ │ ┌────────┐ ┌───────┐ │   │
│  │ Offline  │ │ Offline  │ │ Online   │ │ │Explain │ │Scan   │ │   │
│  │ capable  │ │ capable  │ │ only     │ │ │(own RAG│ │+ Fill │ │   │
│  │          │ │          │ │          │ │ │ API)   │ │(manaslu)│  │
│  └────┬─────┘ └────┬─────┘ └────┬─────┘ │ └────┬───┘ └───┬───┘ │   │
│       │            │            │        └──────┼───────┼──────┘   │
│       │            │            │               │       │           │
│  ┌────▼────────────▼────────────▼───────────────▼───┐   │           │
│  │       Shared: Auth · i18n · Disclaimer · Design   │   │           │
│  └────────────────────────────────────────────────────┘   │           │
└──────────────────────────────┬───────────────────────────┼──────────┘
                               │ HTTPS + JWT                │ HTTPS + JWT
                               │ (saathi API)                │ (forwarded to manaslu)
                    ┌──────────▼──────────┐        ┌────────▼─────────────┐
                    │  CLOUD RUN           │        │  MANASLU (separate   │
                    │  FastAPI (saathi)    │        │  repo/service)       │
                    │                      │        │                      │
                    │  /tracker    F1      │───┐    │  /v1/sessions        │
                    │  /calculator F2      │   │    │  /v1/sessions/{id}/  │
                    │  /checklist  F3      │   │    │    messages (SSE)    │
                    │  /explain    F4a RAG │   │    │  /v1/.../documents   │
                    │                      │   │    │  /v1/.../confirmations│
                    └──────────┬───────────┘   │    │                      │
                               │               │    │  Gap-resolution engine│
                    ┌──────────▼──────────┐    │    │  scan/extract/fill/  │
                    │  CLOUD SQL           │    │    │  transliterate tools │
                    │  (saathi's own DB)    │    │    └──────────┬───────────┘
                    │  ├── visas            │    │               │
                    │  ├── checklists       │    │    ┌──────────▼───────────┐
                    │  └── knowledge_base   │    │    │  MANASLU'S OWN DB     │
                    │      (pgvector)       │    │    │  documents, extractions,
                    │                      │    │    │  profile_facts (vault),
                    │  GCS (F1-F3 assets)   │    │    │  filled_forms, audit_log
                    │  Identity Platform    │    └───▶│  (Cloud SQL + GCS, AU) │
                    │  (shared auth for     │         └───────────────────────┘
                    │   both services)      │
                    └───────────────────────┘
```

**Note on F4b:** saathi never talks to Claude Vision, pypdf, or an AcroForm manifest directly — all of that is manaslu's internal implementation. Saathi's Next.js BFF route forwards the end-user's Identity Platform JWT to manaslu, opens/resumes a session, and renders the SSE events manaslu emits (`extraction.ready`, `review.required`, `fill.completed`) into the review UI. Full contract: `manaslu/docs/architecture/06-service-api.md`.

---

## 6. Data Architecture

**Scope note:** this schema is saathi's own database only. Documents, extractions, the profile vault, filled PDFs, and their audit trail live in manaslu's database — saathi never stores a copy.

### Entity Relationship Diagram

```
users ─────────┬─────────────────────────────────────────────┐
│ uid (PK, from Identity Platform — no password/credential data here) │
│ name_np      │                                             │
│ name_en      │  visas ───────────────────┐                 │
│ language_pref│  │ id (PK)               │                 │
│ created_at   │  │ uid (FK → users)       │                 │
│              │  │ visa_subclass          │                 │
              │  │ visa_type              │                 │
              │  │ grant_date             │                 │
              │  │ expiry_date            │                 │
              │  │ conditions (JSONB)      │                 │
              ├──┤ status                  │                 │
              │  │ notes                   │                 │
              │  │ reminder_180d_sent      │                 │
              │  │ reminder_90d_sent       │                 │
              │  │ reminder_30d_sent       │                 │
              │  │ reminder_7d_sent        │                 │
              │  │ created_at              │                 │
              │  └─────────────────────────┘                 │
              │                                             │
              │  checklist_sessions ──────┐                  │
              │  │ id (PK)               │                  │
              │  │ uid (FK → users)       │                  │
              │  │ visa_type              │                  │
              │  │ answers (JSONB)         │                  │
              └──┤ generated_checklist     │                  │
                 │   (JSONB)              │                  │
                 │ status                  │                  │
                 │ created_at              │                  │
                 └─────────────────────────┘                  │
                                                              │
                 knowledge_base ───────────┐                 │
                 │ id (PK)                 │                 │
                 │ content_type             │                 │
                 │   (form_field|visa_cond|                 │
                 │    checklist_item|points_rule)             │
                 │ title_en                │                 │
                 │ title_np                │                 │
                 │ content_en              │                 │
                 │ content_np              │                 │
                 │ source_url              │                 │
                 │ last_verified           │                 │
                 │ embedding (vector(1024))│  ◄── Voyage AI multilingual-2 (see ADR-007), not OpenAI
                 └─────────────────────────┘
```

manaslu's schema (for reference — owned and versioned in that repo, `manaslu/docs/architecture/08-data-layer.md`): `consents`, `sessions`, `documents`, `extractions`, `user_entries`, `profile_facts` (the vault), `review_requests`, `filled_forms`, `fill_values`, `audit_log`.

### Key Indexes
```sql
CREATE INDEX idx_visas_user_expiry ON visas(uid, expiry_date);
CREATE INDEX idx_checklist_user ON checklist_sessions(uid, created_at DESC);
CREATE INDEX idx_knowledge_embedding ON knowledge_base
  USING ivfflat (embedding vector_cosine_ops);
```

### Row-level ownership (every table)
Cloud SQL Postgres RLS, mirroring manaslu's resource-ownership model:
```sql
ALTER TABLE visas ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Users can only access own visas"
  ON visas FOR ALL
  USING (current_setting('app.current_uid')::text = uid);
```
(`app.current_uid` set per-request from the verified Identity Platform JWT in FastAPI middleware — same pattern manaslu uses for its own tables.)

---

## 7. F1 — Visa Tracker

### Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    F1 — VISA TRACKER                         │
│                                                              │
│  ┌──────────────────┐     ┌─────────────────────────────┐   │
│  │  User Input Flow │     │  Dashboard                  │   │
│  │                  │     │                             │   │
│  │  Select visa     │     │  ┌─────────────────────┐    │   │
│  │  subclass ───────┼────▶│  │ Days remaining: 247 │    │   │
│  │  Enter grant     │     │  │ ████████░░░ 82%     │    │   │
│  │  date ───────────┤     │  └─────────────────────┘    │   │
│  │  Enter expiry ───┤     │                             │   │
│  │  (or auto-calc)  │     │  ┌─────────────────────┐    │   │
│  └──────────────────┘     │  │ Conditions           │    │   │
│                           │  │ • 40 hrs/fortnight   │    │   │
│                           │  │ • Must maintain OSHC │    │   │
│  ┌──────────────────┐     │  └─────────────────────┘    │   │
│  │  Reminder Engine │     │                             │   │
│  │                  │     │  ┌─────────────────────┐    │   │
│  │  Cloud Scheduler ┼──▶  │  │ Next Steps           │    │   │
│  │  triggers daily  │     │  │ → Apply for 485      │    │   │
│  │  Cloud Run job,  │     │  │   before 2027-03-15  │    │   │
│  │  checks expiry   │     │  └─────────────────────┘    │   │
│  │  fires at:       │     └─────────────────────────────┘   │
│  │  180d, 90d,      │                                       │
│  │  30d, 7d before  │                                       │
│  └──────────────────┘                                       │
│                                                              │
│  Data: visas table (Cloud SQL, row-owned)                    │
│  API: Cloud Run FastAPI /tracker (CRUD)                      │
│  Notifications: Firebase Cloud Messaging (FCM)                │
│  Offline: Service Worker caches visa data locally             │
└─────────────────────────────────────────────────────────────┘
```

### API Contracts
```
GET    /tracker          → List user's visas
POST   /tracker          → Add a new visa entry
PATCH  /tracker/:id      → Update visa details
DELETE /tracker/:id      → Remove a visa
```

### Reminder Architecture
- **Cloud Scheduler** triggers a Cloud Run job daily at 2am AEST
- Query: `SELECT * FROM visas WHERE expiry_date - CURRENT_DATE IN (180, 90, 30, 7) AND reminder_Nd_sent = false`
- Sends push notification via **FCM** (works uniformly across web push + any future native wrapper, one SDK instead of the Web Push/OneSignal split debated earlier — see ADR-005)
- Sets `reminder_Nd_sent = true` flag

### Offline Support
- Service Worker caches visa data from last successful fetch
- User can view tracker offline (read-only)
- Queues add/update actions and syncs when online

---

## 8. F2 — Points Calculator

### Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                   F2 — POINTS CALCULATOR                          │
│                                                                   │
│  ┌───────────────────────┐     ┌────────────────────────────┐    │
│  │  Step-by-Step Wizard  │     │  Results Screen            │    │
│  │                       │     │                            │    │
│  │  Step 1/12: Age       │     │  Your Points: 75/100       │    │
│  │  ┌─────────────────┐  │     │  ████████████████░░░░      │    │
│  │  │ Select age range │  │     │                            │    │
│  │  │ ○ 18-24 (25 pts) │  │     │  Breakdown:               │    │
│  │  │ ● 25-32 (30 pts) │  │     │  Age: 30 | English: 20    │    │
│  │  │ ○ 33-39 (25 pts) │  │     │  Work exp: 10 | Study: 5  │    │
│  │  │ ○ 40-44 (15 pts) │  │     │  Partner: 5 | State: 5    │    │
│  │  └─────────────────┘  │     │                            │    │
│  │                       │     │  Latest 189 invites: 85+   │    │
│  │  Progress: ●●○○○○○○○○  │     │  Latest 190 invites: 70+   │    │
│  │                       │     │                            │    │
│  │  [Back]     [Next]    │     │  How to improve:           │    │
│  └───────────────────────┘     │  • NAATI CCL: +5 pts       │    │
│                                │  • PY completion: +5 pts   │    │
│  ┌───────────────────────┐     │                            │    │
│  │  Rules Engine          │     │  [Save]  [Share]  [Reset] │    │
│  │                       │     │                            │    │
│  │  Static JSON config   │     │  ⚠️ This is an estimate    │    │
│  │  in knowledge/         │     │  only. Consult a MARA      │    │
│  │  points-criteria/      │     │  registered agent for     │    │
│  │  gsm-points.json       │     │  formal assessment.       │    │
│  │                       │     └────────────────────────────┘    │
│  │  Deterministic,       │                                       │
│  │  testable, no LLM      │                                       │
│  └───────────────────────┘                                       │
│                                                                   │
│  Data: Client-side calculation (no DB write required)             │
│  API: Cloud Run FastAPI /calculator (validate + log anonymously)  │
│  SkillSelect data: Fetched from knowledge/points-criteria/         │
│  Updated manually or via scheduled scraper                         │
└──────────────────────────────────────────────────────────────────┘
```

### Design Decision: Rules Engine, NOT LLM

The points calculator is **purely rule-based** — a static JSON configuration + a calculation function. Why not LLM:

- ❌ LLM hallucination risk in a high-stakes calculation
- ❌ LLM would need to be prompted with ALL rules every call (cost)
- ❌ LLM math errors are well-documented
- ✅ Rule engine is deterministic, testable, and always correct
- ✅ Updates require a JSON edit, not prompt engineering

### Rules Schema
```json
{
  "criteria": [
    {
      "id": "age",
      "label_en": "Age",
      "label_np": "उमेर",
      "options": [
        {"range": "18-24", "points": 25},
        {"range": "25-32", "points": 30},
        {"range": "33-39", "points": 25},
        {"range": "40-44", "points": 15}
      ],
      "max_points": 30
    }
  ],
  "meta": {
    "last_updated": "2026-06-01",
    "source": "https://immi.homeaffairs.gov.au/visas/working-in-australia/skillselect"
  }
}
```

### SkillSelect Invitation Round Data
- Stored as static JSON in `knowledge/points-criteria/invitation-rounds.json`
- Updated manually by reviewing published SkillSelect round results
- Future: scheduled scraper that checks for new rounds and alerts if data stale > 30 days

---

## 9. F3 — Document Checklist

### Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                F3 — DOCUMENT CHECKLIST                        │
│                                                               │
│  ┌─────────────────────┐     ┌──────────────────────────┐    │
│  │  Visa Selection     │     │  Branching Questions     │    │
│  │                     │     │                          │    │
│  │  ┌───────────────┐  │     │  For 485 visa:          │    │
│  │  │ 485 Temporary │  │     │                          │    │
│  │  │ Graduate      │  │     │  Q1: Which stream?      │    │
│  │  └───────────────┘  │     │   ● Graduate Work       │    │
│  │  ┌───────────────┐  │     │   ○ Post-Study Work     │    │
│  │  │ 500 Student   │  │     │                          │    │
│  │  └───────────────┘  │     │  Q2: Apply onshore?     │    │
│  │  ┌───────────────┐  │     │   ● Yes (in Australia)  │    │
│  │  │ 189 Skilled   │  │     │   ○ No (offshore)       │    │
│  │  │ Independent   │  │     │                          │    │
│  │  └───────────────┘  │     │  Q3: Include partner?   │    │
│  │  ┌───────────────┐  │     │   ○ Yes                 │    │
│  │  │ 190 Skilled   │  │     │   ● No                  │    │
│  │  │ Nominated     │  │     │                          │    │
│  │  └───────────────┘  │     │  Q4: Study in regional  │    │
│  │  ┌───────────────┐  │     │     Australia?          │    │
│  │  │ 491 Regional  │  │     │   ● Yes                 │    │
│  │  │ (Provisional) │  │     │   ○ No                  │    │
│  │  └───────────────┘  │     └──────────┬───────────────┘    │
│  │  ┌───────────────┐  │                │                    │
│  │  │ 482 TSS       │  │                ▼                    │
│  │  └───────────────┘  │     ┌──────────────────────────┐    │
│  └─────────────────────┘     │  Generated Checklist     │    │
│                              │                          │    │
│  ┌─────────────────────┐     │  📋 Identity Documents   │    │
│  │  Decision Tree      │     │  ├ ☐ Certified passport  │    │
│  │  Engine             │     │  ├ ☐ Birth certificate   │    │
│  │                     │     │  └ ☐ Passport photo      │    │
│  │  Stored as JSON:    │     │                          │    │
│  │  knowledge/visa-    │     │  📋 Study Documents      │    │
│  │  types/485.json     │     │  ├ ☐ Completion letter   │    │
│  │                     │     │  ├ ☐ Academic transcript │    │
│  │  Each visa type     │     │  └ ☐ AFP police check    │    │
│  │  has a decision     │     │                          │    │
│  │  tree that maps     │     │  📋 English (485 GW)     │    │
│  │  answers → items    │     │  ├ ☐ IELTS overall 6.0   │    │
│  └─────────────────────┘     │  └ ☐ Test within 3 years │    │
│                              │                          │    │
│  Each checklist item:        │  [Print] [Save] [Share]  │    │
│  {                           │                          │    │
│    "id": "cert-passport",    │  Last verified: June 2026│    │
│    "category": "identity",   │  Source: immi.gov.au     │    │
│    "title_en": "Certified    └──────────────────────────┘    │
│      copy of passport",                                     │
│    "title_np": "पासपोर्टको      │
│      प्रमाणित प्रतिलिपि",     │
│    "description_en": "...",  │
│    "description_np": "...",  │
│    "how_to_get_en": "...",   │
│    "how_to_get_np": "...",   │
│    "common_mistakes_en": ...,│
│    "common_mistakes_np": ...,│
│    "source_url": "...",      │
│    "last_verified": "..."    │
│  }                           │
└──────────────────────────────────────────────────────────────┘
```

### Design Decision: Decision Tree, NOT LLM

Same reasoning as F2 — checklist generation is deterministic. No LLM required for the POC version of F3. An LLM could enhance it later (e.g., "I have an unusual document — is it equivalent?") but that's Phase 2.

---

## 10. F4 — Form Helper

F4 splits across two capabilities with different owners:

### 10.1 F4a — Form Explainer (RAG, saathi's own build)

```
┌────────────────────────────────────────────────────────────────┐
│                   F4a — FORM EXPLAINER                          │
│                                                                 │
│  ┌─────────────────────────┐   ┌───────────────────────────┐   │
│  │  Form Selection          │   │  Field-by-Field View     │   │
│  │                         │   │                           │   │
│  │  ┌───────────────────┐  │   │  Form 80 — Question 1    │   │
│  │  │ Form 80           │  │   │                           │   │
│  │  │ Personal Details  │  │   │  🇬🇧 Family Name          │   │
│  │  └───────────────────┘  │   │  🇳🇵 थर                   │   │
│  │  ┌───────────────────┐  │   │                           │   │
│  │  │ Form 1221         │  │   │  📖 What this means:     │   │
│  │  │ Addl. Particulars │  │   │  Enter your family name  │   │
│  │  └───────────────────┘  │   │  exactly as shown on     │   │
│  │  ┌───────────────────┐  │   │  your passport. This     │   │
│  │  │ 485 Online Form   │  │   │  must match DHA records.│   │
│  │  └───────────────────┘  │   │                           │   │
│  │                         │   │  ⚠️ Common mistakes:     │   │
│  └─────────────────────────┘   │  • Using married name    │   │
│                                 │    when passport shows   │   │
│  ┌─────────────────────────┐   │    maiden name           │   │
│  │  RAG Pipeline            │   │  • Spelling errors      │   │
│  │                         │   │                           │   │
│  │  1. User selects form   │   │  📎 Source:              │   │
│  │     + field             │   │  immi.gov.au/form-80     │   │
│  │                         │   │  Verified: June 2026     │   │
│  │  2. Query → embedding   │   │                           │   │
│  │     → pgvector search   │   │  ┌───────────────────┐   │   │
│  │                         │   │  │ Was this helpful? │   │   │
│  │  3. Retrieve top-3      │   │  │ [👍] [👎]         │   │   │
│  │     relevant chunks     │   │  └───────────────────┘   │   │
│  │                         │   │                           │   │
│  │  4. Claude generates    │   │  ⚠️ This explanation is  │   │
│  │     bilingual           │   │  for information only.   │   │
│  │     explanation with    │   │  Not migration advice.   │   │
│  │     citations           │   └───────────────────────────┘   │
│  └─────────────────────────┘                                   │
└────────────────────────────────────────────────────────────────┘
```

Field label + explanation content can be pre-generated once per field (they don't vary per user) and served from `knowledge_base`, turning most requests into a DB read rather than a live Claude call — same cost lever manaslu uses for its own bilingual manifests.

### 10.2 F4b — Scan & Form-Fill (manaslu, consumed via API)

Saathi does **not** implement classify/extract/validate/transliterate/fill. It calls manaslu:

```
┌─────────────────────────────────────────────────────────────────────┐
│         F4b — WHAT SAATHI DOES (the rest is manaslu's, see its       │
│                docs/architecture/ — not duplicated here)             │
│                                                                      │
│  1. User opens Form Helper → Scan flow                               │
│     saathi BFF: POST manaslu /v1/sessions (forwards end-user JWT)    │
│                                                                      │
│  2. User uploads document(s)                                        │
│     saathi BFF: signed-URL handshake → GCS (manaslu's bucket)        │
│     or proxies multipart to manaslu /v1/sessions/{id}/documents      │
│                                                                      │
│  3. saathi opens SSE stream: /v1/sessions/{id}/messages              │
│     Renders events as they arrive:                                   │
│     ┌────────────────────┬──────────────────────────────────────┐   │
│     │ tool.started/finished│ progress indicator (classifying...) │   │
│     │ extraction.ready     │ review table: field, value, confidence,│  │
│     │                      │ source-doc crop, bilingual label     │   │
│     │                      │ ("pre-filled from your saved profile" │  │
│     │                      │  badge if sourced from the vault,     │  │
│     │                      │  not a fresh scan)                   │   │
│     │ review.required       │ pauses; if transliteration choice —  │   │
│     │                      │ show source Devanagari snippet +      │   │
│     │                      │ ranked candidates + free-text box     │   │
│     │ fill.completed        │ download link (filled PDF + annex)    │   │
│     └────────────────────┴──────────────────────────────────────┘   │
│                                                                      │
│  4. User confirms/corrects → saathi BFF: POST .../confirmations      │
│     (resumes manaslu's paused gap-resolution session)                            │
│                                                                      │
│  5. GET .../artifacts/{id} → signed URL to filled PDF + audit annex  │
└─────────────────────────────────────────────────────────────────────┘
```

Full event/endpoint contract, confidence-tier definitions, and the transliteration priority order: `manaslu/docs/architecture/06-service-api.md` and `04-transliteration.md`. Saathi-side detail (BFF routes, JWT forwarding, review-UI component spec): `docs/architecture/f4-manaslu-integration.md`.

**Confidence tiers (defined by manaslu, rendered by saathi):** HIGH ≥85% (pre-filled, green) · MED 50-84% (pre-filled amber, confirm required) · LOW <50% (blank, manual entry) · FAIL (blank, "could not read"). Never generated by saathi — these arrive in `extraction.ready` payloads.

---

## 10.5 Knowledge Pipeline — AU Visa Source Registry

Saathi's knowledge base is powered by the **AU Visa Source Registry**, an autonomous crawler that discovers, catalogues, and monitors 19 Australian government domains for visa-related content changes.

### How it works

```
┌──────────────────────────────────────────────────────────────┐
│            AU VISA SOURCE REGISTRY (cron: daily 6am)         │
│                                                               │
│  19 government domains crawled daily:                         │
│  ┌─────────────────────────────────────────────────────┐     │
│  │ immi.homeaffairs.gov.au — visa listings, fees, forms│     │
│  │ homeaffairs.gov.au — policy, media releases         │     │
│  │ legislation.gov.au — Migration Act, Regulations     │     │
│  │ mara.gov.au / portal.mara.gov.au — agent register   │     │
│  │ jobsandskills.gov.au — occupation lists              │     │
│  │ nsw.gov.au / vic.gov.au / qld.gov.au / wa.gov.au    │     │
│  │ migration.sa.gov.au — state nomination              │     │
│  │ abs.gov.au — population/migration statistics        │     │
│  │ aat.gov.au / art.gov.au — tribunal decisions        │     │
│  │ treasury.gov.au / budget.gov.au — budget measures   │     │
│  │ studyaustralia.gov.au — education pathways          │     │
│  └─────────────────────────────────────────────────────┘     │
│                          │                                    │
│                          ▼                                    │
│  ┌─────────────────────────────────────────────────────┐     │
│  │  Crawl (300 pages/run, polite 1s delay)              │     │
│  │  → Classify (17 categories)                          │     │
│  │  → Extract (visa subclasses, dates, summaries)       │     │
│  │  → Hash (change detection)                           │     │
│  │  → Upsert to Notion: "AU Visa Source Registry" DB    │     │
│  └─────────────────────────────────────────────────────┘     │
│                          │                                    │
│                          ▼                                    │
│  ┌─────────────────────────────────────────────────────┐     │
│  │  SkillSelect Monitor (cron: every 6h)                │     │
│  │  → Check invitation rounds page for new data         │     │
│  │  → Alert Discord #job_market on new round            │     │
│  └─────────────────────────────────────────────────────┘     │
└──────────────────────────────────────────────────────────────┘
                          │
                          ▼ feeds into
┌──────────────────────────────────────────────────────────────┐
│                    SAATHI KNOWLEDGE BASE                      │
│                                                               │
│  F1 Tracker     ← visa conditions, expiry rules              │
│  F2 Calculator  ← invitation round data, points criteria     │
│  F3 Checklist   ← document requirements per visa type        │
│  F4a Explainer  ← form field explanations, legislation refs  │
│  F5 News        ← announcements from all 19 domains          │
│  F6 Agents      ← MARA agent register verification            │
└──────────────────────────────────────────────────────────────┘
```

### Content Freshness Guarantee

Every piece of content in Saathi carries:
- **Source URL** — exact page the information came from
- **Last verified** — date the crawler last confirmed the page unchanged
- **Freshness tier** — green (<7 days), amber (7-30 days), red (>30 days), stale (>90 days — flagged for manual review)

The crawler detects changes via content hashing. When a page changes:
1. Notion record is updated with the new content + change log
2. Saathi's next RAG ingestion picks up the change
3. Old embeddings are replaced with new ones
4. Users see "Updated: today" instead of stale data

### Reliability Tiers

Every source page is tiered:
- **Tier 1** — legislation, official policy (legislation.gov.au, Migration Act)
- **Tier 2** — official guidance (immi.homeaffairs.gov.au, MARA)
- **Tier 3** — supporting material (state pages, ABS, JSA)

This feeds into Saathi's confidence weighting — Tier 1 sources are authoritative, Tier 3 sources are contextual.

### Crawler Details

| Attribute | Value |
|-----------|-------|
| Location | `~/Workspace/research/au-visa-sources/` |
| Repo | `judasprabin/research` (main branch) |
| Domains | 19 Australian government domains |
| Capacity | 300 pages/run, 15 per domain |
| Frequency | Daily at 6:00 AEST |
| CDN handling | Chrome/Safari UA rotation for blocked domains |
| Storage | Notion: "AU Visa Source Registry" database |
| Classifier | 17 categories with regex path matching |
| Deliver | `discord:#job_market` |

---

## 11. RAG & Knowledge Architecture

**Scope:** this is saathi's own knowledge service, backing F4a (field explanations) and F3 (checklist citations). It is unrelated to manaslu's internal data (documents/extractions/vault) — no overlap, no shared database.

### Knowledge Base Design

```
┌──────────────────────────────────────────────────────────────┐
│                  KNOWLEDGE PIPELINE                           │
│                                                               │
│  ┌─────────────────────┐     ┌──────────────────────────┐    │
│  │  Content Sources    │     │  Ingestion Pipeline      │    │
│  │                     │     │                          │    │
│  │  immi.gov.au        │     │  1. Scheduled scraper   │    │
│  │  • Form 80 page     │────▶│     (Cloud Run job,      │    │
│  │  • Form 1221 page   │     │      weekly)             │    │
│  │  • Visa conditions  │     │                          │    │
│  │  • Step-by-step     │     │  2. Change detection:    │    │
│  │    guides           │     │     hash comparison vs   │    │
│  │                     │     │     last ingested version│    │
│  │  SkillSelect        │     │                          │    │
│  │  • Invitation rounds│     │  3a. If CHANGED:         │    │
│  │  • Points table     │     │      → Split into chunks │    │
│  │                     │     │      → Generate embedding │    │
│  │  MARA website       │     │      → Upsert pgvector    │    │
│  │  • Agent register   │     │      → Update last_verified│    │
│  │                     │     │  3b. If UNCHANGED:       │    │
│  │                     │     │      → Update last_checked│    │
│  └─────────────────────┘     └──────────────────────────┘    │
│                                                               │
│  ┌──────────────────────────────────────────────────────┐    │
│  │  Vector Store (pgvector, Cloud SQL)                   │    │
│  │                                                       │    │
│  │  Chunking strategy:                                   │    │
│  │  • Form fields: 1 chunk per field (EN + NP)          │    │
│  │  • Visa conditions: 1 chunk per condition            │    │
│  │  • Guides: ~500-token chunks with 50-token overlap   │    │
│  │  • Checklist items: 1 chunk per item                 │    │
│  │                                                       │    │
│  │  Embedding model: Voyage AI voyage-multilingual-2    │    │
│  │  (1024-dim) — NOT OpenAI text-embedding-3, which     │    │
│  │  measurably degrades on Nepali/Devanagari — see       │    │
│  │  ADR-007                                              │    │
│  └──────────────────────────────────────────────────────┘    │
│                                                               │
│  ┌──────────────────────────────────────────────────────┐    │
│  │  Retrieval → Generation Flow                          │    │
│  │                                                       │    │
│  │  1. User selects form + field                        │    │
│  │  2. Query → Voyage embedding                          │    │
│  │  3. pgvector similarity search (cosine, top-3)        │    │
│  │  4. Claude Sonnet prompt:                              │    │
│  │     "Using these excerpts from immi.gov.au:          │    │
│  │      [chunk 1] [chunk 2] [chunk 3]                   │    │
│  │      Explain what 'Family Name' means in Form 80.    │    │
│  │      Respond in Nepali with English field name.      │    │
│  │      Cite the source. If uncertain, say so."         │    │
│  │  5. Return: { explanation_np, explanation_en,        │    │
│  │               source_urls, confidence }              │    │
│  │  6. Cache the result in knowledge_base — most fields  │    │
│  │     are answered once and served from DB thereafter   │    │
│  └──────────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────────┘
```

### Content Freshness Strategy
- **Scheduled scraper** (Cloud Scheduler → Cloud Run job) runs weekly
- **Change detection**: compare content hash to last ingested version
- **Staleness alert**: if any chunk has `last_verified > 90 days`, flag in admin dashboard
- **Manual override**: curated JSON files in `knowledge/` take precedence over auto-ingested content

### F5 News & Events ingestion (Phase 2 — same worker, two new tables)

F5 (PRD §4) extends this same ingestion worker rather than adding a new service:

- **`news_items`** — allowlisted-source feed items: headline, source URL + name, published date, subclass/category tags, `summary_np`/`summary_en` (Claude Haiku via **Batch API** — non-interactive, 50% off), `summarised_at`. Aggregation contract: headline + ≤2-sentence summary + attribution + link out; full text is never stored or shown.
- **`events`** — human-curated listings (lightweight admin sheet at MVP): title, date/time, city/online, free/paid, audience tags, organiser, registration URL, `marn_number` + `marn_verified_at` (**required, non-null, for any migration-topic seminar**), `listing_verified_at`, expiry.
- Personalisation is a subclass-tag match against the user's F1 visa record — no ML, no profiles.
- A rule-change news item can reference affected `knowledge_base` chunks, driving the "we're re-checking our checklist against this" state until re-verification.

### F6 Agent Connect data (Phase 2 — three tables, one hard boundary)

- **`agents`** — directory entries: name, `marn` + `marn_verified_at` (quarterly re-check job against the MARA register; lapsed → `delisted`), specialisation tags, languages, location/online, consult fee, response-time stats.
- **`enquiries`** — user → agent introductions: topic, message, contact preference/call windows, status (`sent → viewed → responded → closed | withdrawn`), timestamps per transition.
- **`enquiry_consents`** — the itemised share record: per-enquiry list of exactly which items were shared (contact / enquiry text / visa summary / points summary / checklist progress), consent version, granted/revoked timestamps. Revocation notifies the agent (deletion obligation) and closes the enquiry. Append-only history in `audit_log`.
- **Hard boundary:** no API path exists from manaslu (documents, extractions, vault, filled forms) into F6 — document sharing with agents happens outside Saathi, by the user, deliberately. This is structural, not policy.
- Agent-side surface at MVP: transactional email + a minimal token-authenticated response page (view enquiry, mark responded, propose call time) — a full agent portal only if volume justifies it.

---

## 12. Infrastructure & Deployment

### Technology Stack

| Layer | Technology | Why |
|-------|-----------|-----|
| **Frontend** | Next.js 14 (App Router) + PWA | Mobile-first; works offline; no App Store needed |
| **UI Components** | shadcn/ui + Tailwind CSS | Bilingual-ready; accessible; customizable |
| **i18n** | next-intl | Server + client translations; Devanagari font support |
| **Frontend hosting** | Cloud Run (Next.js standalone build) | Same deploy pattern as the API and as manaslu — one platform |
| **API** | FastAPI (Python) on Cloud Run | F1/F2/F3 CRUD + F4a RAG; same pattern manaslu already uses |
| **Primary AI (F4a)** | Claude Sonnet 5 (Anthropic) | Best Nepali support; structured JSON; consistent with manaslu's model choice |
| **Fallback AI** | Claude Haiku 4.5 | Cheaper; good for simple explanations |
| **Embeddings** | Voyage AI `voyage-multilingual-2` | Meaningfully better Nepali/Devanagari retrieval than OpenAI's embeddings — see ADR-007 |
| **Database** | Cloud SQL for PostgreSQL + pgvector | Same engine/pattern as manaslu (`08-data-layer.md`); AU region |
| **Auth** | GCP Identity Platform | The same token manaslu verifies as a resource server — no cross-cloud auth bridge |
| **Storage** | GCS (user-scoped, AU region) | F1-F3 assets; manaslu owns its own buckets for documents/PDFs |
| **Scheduling** | Cloud Scheduler + Cloud Run jobs | Visa reminders; knowledge refresh |
| **Push Notifications** | Firebase Cloud Messaging (FCM) | One SDK across web + any future native wrapper — see ADR-005 |
| **CI/CD** | GitHub Actions + Workload Identity Federation | Identical pattern to manaslu's (`09-infra-cicd.md`) and `karki-labs-infra` |
| **IaC** | Terraform in `karki-labs-infra` | Same repo already provisioning manaslu's infra |
| **Monitoring** | Cloud Logging + Cloud Monitoring + Error Reporting | Matches manaslu; one observability stack, not two |

### POC Cost Estimate

Cloud Run scales to zero at low volume, similar to the earlier Vercel/Railway free-tier assumption — flagging as **estimate, not verified GCP pricing**, re-run once Phase 0 is live:

| Service | Plan | Monthly Cost (est.) |
|---------|------|-------------|
| Cloud Run (web + API, min instances 0) | Pay-per-use, dev traffic | ~$0-5 |
| Cloud SQL (smallest always-on instance, AU region) | See BUILD-SCHEDULE.md cost section | ~$25-70 |
| Claude API | ~5K calls/month | ~$5 |
| Voyage AI embeddings | ~10K tokens/month | <$1 |
| GCS | Minimal at POC scale | <$1 |
| **TOTAL POC** | | **~$30-80/month** |

(Higher than the earlier Supabase/Vercel-based ~$5/mo estimate — Cloud SQL has no free tier and its always-on cost dominates. Manaslu's infra already accepts this tradeoff for one coherent stack. Cost lever: **share one Cloud SQL instance with manaslu** (separate databases, one instance) at POC scale, halving the fixed cost — decide in Phase 0. Full breakdown + verification flag: `docs/BUILD-SCHEDULE.md` cost section.)

### Environments
```
local      → docker-compose (Cloud SQL Auth Proxy) + `npm run dev` + `uvicorn api.app:app`
             manaslu run separately per its own local setup
dev        → GCP dev project (Terraform in karki-labs-infra) + Cloud Run dev revisions
production → GCP prod project + Cloud Run prod revisions
```

---

## 13. Security Architecture

```
Layer              Control
─────────────────────────────────────────────────────────
Transport          HTTPS everywhere (Cloud Run, GCS, Cloud SQL private IP)
Authentication     GCP Identity Platform JWT — same token manaslu verifies (no bridging)
Authorization      Row-owned Postgres tables (uid scoping) on ALL Cloud SQL tables
API Keys           Secret Manager only — never in client bundle or env-baked images
Document Storage   Not stored by saathi for F4b — that's manaslu's GCS buckets (AU region)
Token Storage      httpOnly cookies (PWA)
AI Requests        No PII in F4a logs (F4a never touches documents — that's F4b/manaslu)
Data Deletion      CASCADE on user delete; user-requested deletion < 30 days
Logging            No PII in Cloud Logging; uid hashed for traceability (matches manaslu's pattern)
PII Handling       Saathi's own DB has no passport/financial data — F1-F3 don't collect it;
                   all sensitive-document handling is manaslu's, governed by its own
                   `10-security-privacy.md` (AU residency, consent gate, audit trail)
```

---

## 14. Architecture Decision Records

### ADR-001: F1-F3 as a lightweight Cloud Run API; F4b delegated to manaslu (supersedes the Aug 8 "Edge Functions + FastAPI hybrid" draft)
**Status:** Accepted
**Decision:** One FastAPI service on Cloud Run handles F1/F2/F3 CRUD + F4a RAG. F4b (scan/extract/fill) is not built in this repo at all — saathi consumes manaslu's API.
**Rationale:** The original hybrid-backend split (Supabase Edge Functions for CRUD, separate FastAPI for AI) solved a problem that goes away once F4b moves to manaslu — there's no longer a "heavy AI pipeline" inside this repo big enough to justify a second runtime. One Cloud Run FastAPI service is simpler and matches manaslu's own deploy pattern.

### ADR-002: Rules Engine for Points Calculator (Not LLM)
**Status:** Accepted — unchanged
**Decision:** F2 Points Calculator is a deterministic JSON rules engine, not an LLM.
**Rationale:** Points calculation is high-stakes, must be correct every time. A JSON config + test suite guarantees correctness; an LLM could hallucinate.

### ADR-003: Decision Tree for Checklist (Not LLM)
**Status:** Accepted — unchanged
**Decision:** F3 Document Checklist uses branching decision trees stored as JSON, not an LLM.
**Rationale:** Checklist generation is deterministic — same visa type + answers always produces the same checklist.

### ADR-004: pgvector for RAG (Not Pinecone), scoped to saathi's own knowledge base
**Status:** Accepted
**Decision:** Use pgvector on saathi's own Cloud SQL instance for F4a/F3 knowledge retrieval.
**Rationale:** Same reasoning as manaslu's own ADR (`06-service-api.md` doesn't need vectors; this is saathi-only): free with the DB already running, sufficient at <10K chunks. Not shared with manaslu's database — separate concern, separate schema.

### ADR-005: Firebase Cloud Messaging (Not Web Push API alone, not OneSignal)
**Status:** Accepted — resolves a prior undocumented contradiction (the Aug 8 draft picked Web Push API in one place and a tech-research doc recommended OneSignal elsewhere, with no ADR reconciling them)
**Decision:** Use FCM for visa expiry reminders.
**Rationale:** Native to the GCP/Identity Platform stack already in use; works for web push today and any native wrapper later without a second SDK; no third-party vendor (OneSignal) needed, unlike raw Web Push API alone FCM still handles the iOS Safari web-push limitations more gracefully via its abstraction, though the underlying platform constraint (§16) doesn't disappear.

### ADR-006: Claude Sonnet as Primary AI for F4a
**Status:** Accepted
**Decision:** Claude Sonnet 5 for F4a field explanations; Haiku 4.5 fallback for simple cases.
**Rationale:** Best Nepali (Devanagari) support of any commercial model; consistent with manaslu's own model choice for its Sonnet-tier extraction work (`05-llm-strategy.md`) — one vendor across both repos.

### ADR-007: Voyage AI embeddings, not OpenAI (fixes a prior silent contradiction)
**Status:** Accepted
**Decision:** `voyage-multilingual-2` for the F4a/F3 knowledge base, not OpenAI's `text-embedding-3`.
**Rationale:** The Aug 8 draft's cost table listed OpenAI embeddings despite an earlier tech-research pass explicitly recommending Voyage AI *because* OpenAI's embeddings measurably degrade on Nepali — the exact quality problem this RAG system exists to avoid. No ADR previously documented why the switch happened; there's no good reason for it, so this reverts to Voyage.

### ADR-008: F4b is manaslu's responsibility, not rebuilt here
**Status:** Accepted
**Decision:** Saathi does not implement document classification, extraction, validation, transliteration, or AcroForm filling. It integrates with manaslu's REST+SSE API.
**Rationale:** manaslu already exists, is GCP-native, and was purpose-built around the vault/bilingual-explain moat identified in the July 13 competitive re-assessment (`research/dispora-nepal/competitive-analysis.md`). Rebuilding this from scratch (the Aug 8 draft's original Phase 4 plan, ~4 weeks of work) would duplicate solved, harder engineering for no benefit and risk diverging from the vault-first design that's the actual differentiator. See §1 and `docs/architecture/f4-manaslu-integration.md`.

---

## 15. Build Schedule

Full detail: [`docs/BUILD-SCHEDULE.md`](docs/BUILD-SCHEDULE.md). Summary:

| Phase | Weeks | What | Notes |
|-------|-------|------|-------|
| 0 | 1-2 | GCP foundation: Cloud Run, Cloud SQL, Identity Platform, CI/CD (Terraform in karki-labs-infra) | Matches manaslu's own infra pattern |
| 1 | 2-3 | F3 Document Checklist | No LLM; content + branching logic |
| 2 | 3-4 | F2 Points Calculator | Deterministic; test suite |
| 3 | 4-6 | F1 Visa Tracker | CRUD + FCM reminders + offline cache |
| 4 | 6-9 | F4 Form Helper | F4a: own RAG build. F4b: manaslu API integration (materially smaller than a from-scratch build — see ADR-008) |
| 5 | 9-11 | Polish & launch | Accessibility, legal sign-off (see §16), beta testing |

**Dependency note:** manaslu ships on its own M0-M4 timeline in its own repo (`manaslu/docs/architecture/11-mvp-build-plan.md`). Phase 4's F4b integration work can only complete once manaslu's API is live (its M3) — but Phases 0-3 have zero dependency on manaslu and can proceed in parallel.

---

## 16. Open Questions & Risks

| # | Question / Risk | Impact | Status | Resolution By |
|---|----------------|--------|--------|---------------|
| 1 | Form 80/1221 AcroForm field names (D3) | High — blocks F4b fill | 🔴 Open — **this is manaslu's blocker, not saathi's**; tracked there | manaslu's M2 |
| 2 | Claude data retention for migration documents (D5) | Medium — privacy compliance | 🔴 Open — manaslu's concern (F4b handles the documents); saathi's F4a sends no PII to Claude | Before manaslu's public launch |
| 3 | SkillSelect invitation round data freshness | Low — affects F2 comparison accuracy | 🟡 Monitor | Monthly review |
| 4 | Home Affairs website structure changes — may break knowledge scraper | Medium — affects F4a accuracy | 🟡 Monitor | Weekly change detection |
| 5 | FCM/Web Push support on iOS Safari — historically limited even via FCM's abstraction | Low — affects F1 reminders | 🟡 Monitor | Test during Phase 3 |
| 6 | Cost scaling: Claude + Cloud SQL always-on cost if traction hits | Medium — cost spike | 🟡 Monitor | Set cost alerts at $50/mo |
| 7 | **Legal: MARN-registered migration agent's written opinion on F4b autofill — not yet obtained** (legal-memo.md Condition 4) | **High — regulatory risk, gates public launch** | 🔴 **Open, not mitigated** — the fallback-answer-sheet is a contingency plan, not a substitute for the opinion itself | Before any public launch of F4b |
| 8 | manaslu integration readiness — saathi's F4b UI can't be finished until manaslu's API contract is stable (currently at its doc-only stage, no code yet either) | Medium — schedule risk for Phase 4 | 🟡 Monitor | Track against manaslu's own M-milestones |
| 9 | GCP POC cost estimate (§12) is not yet verified against real GCP pricing | Low | 🟡 Open | Re-run after Phase 0 |
| 10 | F5 curation ops: news summaries + event vetting (MARN checks) are recurring human work with no owner yet; copyright posture (aggregation-only) needs a once-over in the legal review | Medium — gates F5 launch, not core product | 🟡 Open | Before Phase 6 (F5 build) |
| 11 | F6 privacy: sharing user PII with third-party agents is an APP 6 disclosure — needs the itemised-consent framework, agent-side data-handling terms (delete-on-revoke), and privacy-policy coverage reviewed by the P5-7 lawyer; referral-fee arrangement needs its own disclosure check | Medium — gates F6 launch, not core product | 🟡 Open | Before Phase 7 (F6 build) |

---

*Architecture compiled: August 8, 2026 · Unified with manaslu: August 8, 2026*
