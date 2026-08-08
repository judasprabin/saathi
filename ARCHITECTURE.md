# Saathi — Master Architecture Document

**Project:** AI Settlement & Immigration Companion for Nepalese Diaspora in Australia
**Version:** 1.0
**Date:** August 8, 2026
**Author:** Prabin Karki
**Status:** Architecture approved — ready for implementation

> This is the single source of truth for Saathi's architecture. All 4 features, the scan/form-fill pipeline, and infrastructure decisions are documented here. Component-specific detail files live in `docs/architecture/`.

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
10. [Feature Architecture — F4 Form Helper + Scan Pipeline](#10-f4-form-helper--scan-pipeline)
11. [RAG & Knowledge Architecture](#11-rag--knowledge-architecture)
12. [Infrastructure & Deployment](#12-infrastructure--deployment)
13. [Security Architecture](#13-security-architecture)
14. [Architecture Decision Records](#14-architecture-decision-records)
15. [Build Schedule](#15-build-schedule)
16. [Open Questions & Risks](#16-open-questions--risks)

---

## 1. System Overview

### What Saathi Is

A focused, bilingual (English/Nepali) PWA that helps Nepalese migrants in Australia navigate the immigration system with four utility tools:

| # | Feature | What It Does | Complexity |
|---|---------|-------------|------------|
| F1 | **Visa Tracker** | Track visa expiry, conditions, and deadlines | Low |
| F2 | **Points Calculator** | Calculate skilled migration points score | Medium |
| F3 | **Document Checklist** | Generate personalised visa document checklists | Medium |
| F4 | **Form Helper** | Explain immigration form fields in Nepali; scan documents → auto-fill forms | High |

### What Saathi Is NOT

- A migration agent — does not lodge applications or give advice
- An ImmiAccount integration — does not connect to Home Affairs systems
- A marketplace (initially) — no agent matching until traction is proven
- A community platform — focused purely on utility tools

### Regulatory Boundary (Non-Negotiable)

```
✅ ALLOWED: explaining form fields, showing required documents, calculating points,
            tracking user-entered dates, translating official content into Nepali

❌ PROHIBITED: assessing eligibility, recommending visa pathways, lodging applications,
              preparing forms on a person's behalf, giving migration advice

🔄 HANDOFF: whenever a question crosses into advice territory → "Consult a registered
            migration agent (verify at mara.gov.au using their MARN)"
```

---

## 2. Architecture Principles

1. **Bilingual by default** — Every UI label, explanation, and error message exists in English AND Nepali. No English-only dead ends.

2. **Cite everything** — Every checklist item, calculator rule, and form explanation carries a source URL and "last verified" date. Stale information is more dangerous than no information.

3. **Fail safe, not silent** — Every AI call has a confidence threshold. Low-confidence outputs are flagged, not auto-accepted. The user always has a manual fallback path.

4. **Cheap until proven** — Use free tiers wherever possible (Supabase, Claude, Vercel). Only pay when traction justifies it. The entire POC should cost < $50/month.

5. **Progressive disclosure** — Don't overwhelm new users with all 4 features. Onboard one feature at a time with clear value demonstration.

6. **Offline-capable where possible** — F1 (Tracker) and F2 (Calculator) work without internet. F3 (Checklist) and F4 (Form Helper) require connectivity for AI calls.

7. **RLS everywhere** — Row Level Security on all Supabase tables from day one. Users can only access their own data.

---

## 3. Repository Structure

```
saathi/                              # Monorepo root
├── web/                             # Next.js PWA (frontend)
│   ├── app/                         # App Router pages
│   │   ├── (auth)/                  # Login/register
│   │   ├── (dashboard)/             # Main app shell
│   │   │   ├── tracker/             # F1 — Visa Tracker
│   │   │   ├── calculator/          # F2 — Points Calculator
│   │   │   ├── checklist/           # F3 — Document Checklist
│   │   │   ├── form-helper/         # F4 — Form Helper
│   │   │   └── settings/            # Profile, language toggle
│   │   └── api/                     # Next.js API routes (light proxy)
│   ├── components/
│   │   ├── ui/                      # Design system (shadcn/ui based)
│   │   ├── features/                # Feature-specific components
│   │   │   ├── tracker/
│   │   │   ├── calculator/
│   │   │   ├── checklist/
│   │   │   └── form-helper/
│   │   └── shared/                  # Shared: LanguageToggle, Disclaimer, etc.
│   ├── lib/
│   │   ├── supabase.ts              # Supabase client (browser)
│   │   ├── i18n.ts                  # Internationalization (EN/NP)
│   │   └── constants.ts             # Visa types, form lists, etc.
│   ├── public/
│   │   └── locales/                 # EN/NP translation files
│   └── next.config.js
│
├── api/                             # FastAPI (heavy AI pipelines)
│   ├── app/
│   │   ├── routers/
│   │   │   ├── scan.py              # F4 — document upload + classify
│   │   │   ├── extract.py           # F4 — schema + open extraction
│   │   │   ├── fill.py              # F4 — PDF fill (AcroForm)
│   │   │   ├── explain.py           # F4 — RAG field explanation
│   │   │   └── health.py            # Health check
│   │   ├── services/
│   │   │   ├── claude_client.py     # Claude Vision + text API wrapper
│   │   │   ├── classifier.py        # Document type classification
│   │   │   ├── extractor.py         # Schema-driven extraction
│   │   │   ├── validator.py         # MRZ checksum, date plausibility, etc.
│   │   │   ├── confidence.py        # Per-field confidence scoring
│   │   │   ├── pdf_filler.py        # pdf-lib AcroForm population
│   │   │   ├── rag_service.py       # Vector search + LLM grounding
│   │   │   └── knowledge_refresh.py # Scheduled ingestion of Home Affairs pages
│   │   ├── schemas/
│   │   │   ├── documents.py         # Per-document-type extraction schemas
│   │   │   ├── forms.py             # Form-field mapping manifests
│   │   │   └── points.py            # Points calculator rules schema
│   │   └── core/
│   │       ├── config.py            # Settings from env vars
│   │       ├── supabase.py          # Supabase admin client
│   │       └── errors.py            # Typed error responses
│   ├── tests/
│   └── requirements.txt
│
├── supabase/
│   ├── functions/                   # Edge Functions (lightweight APIs)
│   │   ├── tracker/
│   │   │   └── index.ts             # Visa CRUD + reminder scheduling
│   │   ├── calculator/
│   │   │   └── index.ts             # Points calculation endpoint
│   │   ├── checklist/
│   │   │   └── index.ts             # Checklist generation endpoint
│   │   └── _shared/
│   │       ├── auth.ts              # JWT validation middleware
│   │       ├── db.ts                # Supabase client wrapper
│   │       └── errors.ts            # Error response helpers
│   └── migrations/
│       ├── 001_users.sql            # User profiles
│       ├── 002_visas.sql            # Visa tracker tables
│       ├── 003_checklists.sql       # Checklist + document tables
│       ├── 004_form_extractions.sql # Form extraction audit tables
│       └── 005_knowledge_base.sql   # RAG knowledge base tables
│
├── knowledge/                       # Curated knowledge base
│   ├── forms/                       # Form field manifests
│   │   ├── form-80.json             # AcroForm field names + labels
│   │   └── form-1221.json
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
│   ├── PRD.md                       # Product requirements (copied reference)
│   ├── ARCHITECTURE.md              # This file
│   ├── architecture/
│   │   ├── f1-visa-tracker.md       # F1 detailed architecture
│   │   ├── f2-points-calculator.md  # F2 detailed architecture
│   │   ├── f3-document-checklist.md # F3 detailed architecture
│   │   ├── f4-form-helper.md        # F4 detailed architecture
│   │   ├── scan-pipeline.md         # Document scan + form-fill pipeline
│   │   ├── rag-architecture.md      # RAG & knowledge base design
│   │   └── ui-ux-flows.md           # Detailed UI/UX flows per feature
│   ├── research/
│   │   ├── market-research.md       # Comprehensive market analysis
│   │   ├── competitor-analysis.md   # Competitive landscape
│   │   └── user-research.md         # Target user research
│   └── legal/
│       ├── legal-memo.md            # Visa form-fill legality analysis
│       └── fallback-answer-sheet.md # Legal fallback architecture
│
├── diagrams/
│   └── system-architecture.html     # Interactive SVG architecture diagram
│
├── .env.example                     # Environment variables template
├── docker-compose.yml               # Local development (FastAPI + Supabase local)
└── README.md
```

---

## 4. System Context Diagram

```
                         ┌─────────────────────────────────┐
                         │     SAATHI PLATFORM              │
                         │                                  │
  ┌──────────┐           │  ┌────────────────────────────┐ │
  │  Nepalese │           │  │   Next.js PWA              │ │
  │  Migrant  │──HTTPS──▶│  │   (mobile-first, bilingual) │ │
  │  in AU    │           │  │                             │ │
  └──────────┘           │  │  F1 Tracker   F2 Calculator │ │
                         │  │  F3 Checklist F4 Form Helper│ │
                         │  └──────────┬─────────────────┘ │
                         │             │                     │
                         │    ┌────────▼─────────────────┐  │
                         │    │  Supabase Edge Functions  │  │
                         │    │  (lightweight API layer)  │  │
                         │    │  /tracker  /calculator    │  │
                         │    │  /checklist               │  │
                         │    └──────────┬─────────────────┘  │
                         │               │                    │
                         │    ┌──────────▼─────────────────┐  │
                         │    │  FastAPI (AI Pipeline)      │  │
                         │    │  /scan  /extract  /fill     │  │
                         │    │  /explain  (RAG)            │  │
                         │    └──────────┬─────────────────┘  │
                         │               │                    │
                         │    ┌──────────▼─────────────────┐  │
                         │    │  Supabase                   │  │
                         │    │  PostgreSQL · Auth · Storage│  │
                         │    └────────────────────────────┘  │
                         └─────────────────────────────────────┘
                                          │
                    ┌─────────────────────┼──────────────────────┐
                    ▼                     ▼                      ▼
           ┌──────────────┐    ┌──────────────────┐    ┌──────────────┐
           │ Claude Sonnet │    │ Home Affairs     │    │ SkillSelect   │
           │ (Anthropic)   │    │ (immi.gov.au)    │    │ (live data)   │
           │ Vision + Text │    │ Source of truth  │    │ Invitation    │
           │ Nepali capable│    │ for all content  │    │ rounds data   │
           └──────────────┘    └──────────────────┘    └──────────────┘
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
│  │ capable  │ │ capable  │ │ only     │ │ │(RAG)   │ │+ Fill │ │   │
│  └────┬─────┘ └────┬─────┘ └────┬─────┘ │ └────┬───┘ └───┬───┘ │   │
│       │            │            │        └──────┼───────┼──────┘   │
│       │            │            │               │       │           │
│  ┌────▼────────────▼────────────▼───────────────▼───────▼──────┐   │
│  │              Shared: Auth · i18n · Disclaimer · Design Sys  │   │
│  └─────────────────────────────────────────────────────────────┘   │
└──────────────────────────────┬──────────────────────────────────────┘
                               │ HTTPS + JWT
                    ┌──────────▼──────────┐
                    │  SUPABASE EDGE      │
                    │  FUNCTIONS (Deno)   │
                    │                     │
                    │  /tracker    F1     │──▶ PostgreSQL (visas table)
                    │  /calculator F2     │──▶ PostgreSQL (saved results)
                    │  /checklist  F3     │──▶ PostgreSQL (checklists)
                    │                     │
                    │  Fast, near-DB,     │
                    │  sub-50ms responses │
                    └──────────┬──────────┘
                               │
                    ┌──────────▼──────────┐
                    │  FASTAPI (Python)    │
                    │  AI Pipeline Server  │
                    │                     │
                    │  /scan/upload        │──▶ Supabase Storage
                    │  /scan/classify      │──▶ Claude Vision
                    │  /scan/extract       │──▶ Claude Vision
                    │  /scan/fill          │──▶ pdf-lib
                    │  /explain/field      │──▶ Claude + pgvector
                    │                     │
                    │  Heavy AI work,      │
                    │  2-5s responses      │
                    └──────────┬──────────┘
                               │
                    ┌──────────▼──────────┐
                    │  SUPABASE            │
                    │                     │
                    │  PostgreSQL          │
                    │  ├── users           │
                    │  ├── visas           │
                    │  ├── checklists      │
                    │  ├── form_extractions│
                    │  ├── knowledge_base  │
                    │  └── audit_log       │
                    │                     │
                    │  Auth (JWT)          │
                    │  Storage (user docs) │
                    │  pgvector (RAG)      │
                    └─────────────────────┘
```

---

## 6. Data Architecture

### Entity Relationship Diagram

```
users ─────────┬─────────────────────────────────────────────┐
│ id (PK)      │                                             │
│ email        │                                             │
│ name_np      │  visas ───────────────────┐                 │
│ name_en      │  │ id (PK)               │                 │
│ language_pref│  │ user_id (FK → users)   │                 │
│ created_at   │  │ visa_subclass          │                 │
│              │  │ visa_type              │                 │
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
              │  │ user_id (FK → users)   │                  │
              │  │ visa_type              │                  │
              │  │ answers (JSONB)         │                  │
              ├──┤ generated_checklist     │                  │
              │  │   (JSONB)              │                  │
              │  │ status                  │                  │
              │  │ created_at              │                  │
              │  └─────────────────────────┘                  │
              │                                             │
              │  form_extractions ─────────┐                 │
              │  │ id (PK)                │                 │
              │  │ user_id (FK → users)    │                 │
              │  │ form_type               │                 │
              │  │ source_doc_urls (JSONB) │                 │
              └──┤ extracted_fields (JSONB)│                 │
                 │ confidence_scores (JSONB)                  │
                 │ filled_pdf_url          │                 │
                 │ model_used              │                 │
                 │ audit_log (JSONB)       │                 │
                 │ created_at              │                 │
                 └─────────────────────────┘                 │
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
                 │ embedding (vector(1536))│                 │
                 └─────────────────────────┘                 │
```

### Key Indexes
```sql
CREATE INDEX idx_visas_user_expiry ON visas(user_id, expiry_date);
CREATE INDEX idx_checklist_user ON checklist_sessions(user_id, created_at DESC);
CREATE INDEX idx_form_extractions_user ON form_extractions(user_id, created_at DESC);
CREATE INDEX idx_knowledge_embedding ON knowledge_base 
  USING ivfflat (embedding vector_cosine_ops);
```

### RLS Policies (Every Table)
```sql
-- Pattern applied to all user-scoped tables
ALTER TABLE visas ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Users can only access own visas"
  ON visas FOR ALL
  USING (auth.uid() = user_id);
```

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
│  │  pg_cron job ────┼──▶  │  │ Next Steps           │    │   │
│  │  runs daily      │     │  │ → Apply for 485      │    │   │
│  │  checks expiry   │     │  │   before 2027-03-15  │    │   │
│  │  fires at:       │     │  └─────────────────────┘    │   │
│  │  180d, 90d,      │     └─────────────────────────────┘   │
│  │  30d, 7d before  │                                       │
│  └──────────────────┘                                       │
│                                                              │
│  Data: visas table (user-scoped, RLS)                        │
│  API: Edge Function /tracker (CRUD)                          │
│  Notifications: Web Push API + optional email                │
│  Offline: Service Worker caches visa data locally            │
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
- **pg_cron** runs daily at 2am AEST
- Query: `SELECT * FROM visas WHERE expiry_date - CURRENT_DATE IN (180, 90, 30, 7) AND reminder_Nd_sent = false`
- Sends push notification via Web Push API
- Sets `reminder_Nd_sent = true` flag
- No external scheduler needed at POC scale

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
│  API: Edge Function /calculator (validate + log anonymously)       │
│  SkillSelect data: Fetched from knowledge/points-criteria/        │
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

### Decision Tree Storage

Each visa type has a JSON decision tree file:
```json
{
  "visa_subclass": "485",
  "streams": ["graduate_work", "post_study_work"],
  "questions": [
    {
      "id": "stream",
      "label_en": "Which stream are you applying for?",
      "label_np": "तपाईं कुन stream बाट apply गर्दै हुनुहुन्छ?",
      "options": [
        {"value": "graduate_work", "label_en": "Graduate Work", "label_np": "ग्रेजुएट वर्क"},
        {"value": "post_study_work", "label_en": "Post-Study Work", "label_np": "पोस्ट-स्टडी वर्क"}
      ]
    },
    {
      "id": "onshore",
      "label_en": "Are you applying from inside Australia?",
      "condition": "always",
      "options": [
        {"value": true, "label_en": "Yes", "label_np": "हो"},
        {"value": false, "label_en": "No", "label_np": "होइन"}
      ]
    }
  ],
  "items": [
    {
      "id": "cert-passport",
      "category": "identity",
      "condition": "always",
      "title_en": "Certified copy of passport bio page",
      "title_np": "पासपोर्टको प्रमाणित प्रतिलिपि",
      "description_en": "A certified copy of the bio-data page of your passport...",
      "description_np": "तपाईंको पासपोर्टको जैविक डाटा पृष्ठको प्रमाणित प्रतिलिपि...",
      "how_to_get_en": "Take your passport to a JP (Justice of the Peace)...",
      "how_to_get_np": "आफ्नो पासपोर्ट लिएर JP कहाँ जानुहोस्...",
      "common_mistakes_en": ["Not certified by authorised person", "Expired certification (>6 months)"],
      "common_mistakes_np": ["अधिकृत व्यक्तिबाट प्रमाणित नगरिएको", "प्रमाणीकरणको म्याद सकिएको (>६ महिना)"],
      "source_url": "https://immi.homeaffairs.gov.au/visas/getting-a-visa/visa-listing/temporary-graduate-485",
      "last_verified": "2026-06-28"
    }
  ]
}
```

### Design Decision: Decision Tree, NOT LLM

Same reasoning as F2 — checklist generation is deterministic. No LLM required for the POC version of F3. An LLM could enhance it later (e.g., "I have an unusual document — is it equivalent?") but that's Phase 2.

---

## 10. F4 — Form Helper + Scan Pipeline

### 10.1 Form Explainer (RAG-Based)

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

### 10.2 Document Scan + Auto-Fill Pipeline

```
┌─────────────────────────────────────────────────────────────────────┐
│                    F4b — SCAN & FORM-FILL PIPELINE                   │
│                                                                      │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────────────┐  │
│  │ 1.UPLOAD │──▶│2.CLASSIFY│──▶│3.EXTRACT │──▶│ 4.VALIDATE       │  │
│  │          │   │          │   │          │   │                  │  │
│  │ User     │   │ Claude   │   │ Claude   │   │ MRZ checksum    │  │
│  │ uploads  │   │ Vision:  │   │ Vision:  │   │ Date plausibility│  │
│  │ passport │   │ "This is │   │ "Extract │   │ ABN/BSB format  │  │
│  │ payslip  │   │  a ___"  │   │  fields" │   │ Cross-doc check │  │
│  │ bank stmt│   │          │   │          │   │                  │  │
│  └──────────┘   └──────────┘   └──────────┘   └────────┬─────────┘  │
│                                                         │            │
│                      ┌──────────────────────────────────┘            │
│                      ▼                                               │
│  ┌──────────────────────────────────────────────────────┐           │
│  │                5. CONFIDENCE TIERING                  │           │
│  │                                                       │           │
│  │  ≥ 85%  → 🟢 HIGH    Auto-fill, green highlight      │           │
│  │  50-84% → 🟡 MEDIUM  Pre-fill, amber, confirm required│           │
│  │  < 50%  → 🔴 LOW     Leave blank, user enters manually│           │
│  │  FAIL   → ⚪ BLANK   "Could not read — enter manually"│           │
│  └───────────────────────┬──────────────────────────────┘           │
│                          │                                           │
│                          ▼                                           │
│  ┌──────────────────────────────────────────────────────┐           │
│  │                6. USER REVIEW                         │           │
│  │                                                       │           │
│  │  ┌─────────────┬──────────────┬──────────────────┐   │           │
│  │  │ Field Name  │ Extracted    │ Source Document  │   │           │
│  │  ├─────────────┼──────────────┼──────────────────┤   │           │
│  │  │ Family Name │ KARKI     🟢 │ [passport img]   │   │           │
│  │  │ Given Names │ PRABIN    🟢 │ [passport img]   │   │           │
│  │  │ DOB         │ 01/01/1990🟡 │ [passport img]   │   │           │
│  │  │ Employer    │ _________🔴 │ [payslip img]    │   │           │
│  │  └─────────────┴──────────────┴──────────────────┘   │           │
│  │                                                       │           │
│  │  User confirms, corrects, or enters each field        │           │
│  └───────────────────────┬──────────────────────────────┘           │
│                          │                                           │
│                          ▼                                           │
│  ┌──────────────────────────────────────────────────────┐           │
│  │                7. PDF FILL (AcroForm)                  │           │
│  │                                                       │           │
│  │  pdf-lib: write confirmed values → Form 80 PDF       │           │
│  │  Output: filled + annotated PDF with audit trail      │           │
│  └───────────────────────┬──────────────────────────────┘           │
│                          │                                           │
│                          ▼                                           │
│  ┌──────────────────────────────────────────────────────┐           │
│  │                8. EXPORT & AUDIT                       │           │
│  │                                                       │           │
│  │  Download filled PDF | Store in Supabase Storage      │           │
│  │  Audit log: user_id, doc_type, fields, confidence     │           │
│  │  Retention: user-controlled (GDPR/Privacy Act)        │           │
│  └──────────────────────────────────────────────────────┘           │
└─────────────────────────────────────────────────────────────────────┘
```

### F4 API Contracts
```
POST   /scan/upload          → Validate + store in Supabase Storage
POST   /scan/classify        → Claude Vision: classify document type
POST   /scan/extract         → Schema-driven + open extraction
POST   /scan/fill            → Map extracted fields → AcroForm PDF fill
GET    /scan/:id             → Retrieve extraction result + audit log

POST   /explain/field        → RAG: explain a form field in Nepali
GET    /forms                → List supported forms
GET    /forms/:form_id/fields → List all fields for a form
```

### Confidence Scoring Strategy
Since Claude doesn't expose per-token confidence natively:
1. **Dual extraction** — Run two independent Claude calls with different prompts (schema-driven + open); if both agree on value → high confidence
2. **MRZ checksum** — Passport numbers validated against ISO 7064 MOD 7/11
3. **Date plausibility** — Check against document type norms
4. **Cross-document consistency** — Name on passport must match name on payslip

---

## 11. RAG & Knowledge Architecture

### Knowledge Base Design

```
┌──────────────────────────────────────────────────────────────┐
│                  KNOWLEDGE PIPELINE                           │
│                                                               │
│  ┌─────────────────────┐     ┌──────────────────────────┐    │
│  │  Content Sources    │     │  Ingestion Pipeline      │    │
│  │                     │     │                          │    │
│  │  immi.gov.au        │     │  1. Scheduled scraper   │    │
│  │  • Form 80 page     │────▶│     (weekly)             │    │
│  │  • Form 1221 page   │     │                          │    │
│  │  • Visa conditions  │     │  2. Change detection:    │    │
│  │  • Step-by-step     │     │     hash comparison vs   │    │
│  │    guides           │     │     last ingested version│    │
│  │                     │     │                          │    │
│  │  SkillSelect        │     │  3a. If CHANGED:         │    │
│  │  • Invitation rounds│     │      → Split into chunks │    │
│  │  • Points table     │     │      → Generate embedding │    │
│  │                     │     │      → Upsert pgvector    │    │
│  │                     │     │      → Update last_verified│    │
│  │  MARA website       │     │                          │    │
│  │  • Agent register   │     │  3b. If UNCHANGED:       │    │
│  │                     │     │      → Update last_checked│    │
│  └─────────────────────┘     │      → Skip re-ingestion │    │
│                              └──────────────────────────┘    │
│                                                               │
│  ┌──────────────────────────────────────────────────────┐    │
│  │  Vector Store (pgvector 1536-dim)                    │    │
│  │                                                       │    │
│  │  Chunking strategy:                                   │    │
│  │  • Form fields: 1 chunk per field (EN + NP)          │    │
│  │  • Visa conditions: 1 chunk per condition            │    │
│  │  • Guides: ~500-token chunks with 50-token overlap   │    │
│  │  • Checklist items: 1 chunk per item                 │    │
│  │                                                       │    │
│  │  Embedding model: text-embedding-3-small (OpenAI)     │    │
│  │  → 1536 dimensions, $0.02/1M tokens                  │    │
│  │  → Free tier covers entire knowledge base            │    │
│  └──────────────────────────────────────────────────────┘    │
│                                                               │
│  ┌──────────────────────────────────────────────────────┐    │
│  │  Retrieval → Generation Flow                          │    │
│  │                                                       │    │
│  │  1. User selects form + field                        │    │
│  │  2. Query → embedding                                │    │
│  │  3. pgvector similarity search (cosine, top-3)        │    │
│  │  4. Claude prompt:                                    │    │
│  │     "Using these excerpts from immi.gov.au:          │    │
│  │      [chunk 1] [chunk 2] [chunk 3]                   │    │
│  │      Explain what 'Family Name' means in Form 80.    │    │
│  │      Respond in Nepali with English field name.      │    │
│  │      Cite the source. If uncertain, say so."         │    │
│  │  5. Return: { explanation_np, explanation_en,        │    │
│  │               source_urls, confidence }              │    │
│  └──────────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────────┘
```

### Content Freshness Strategy
- **Scheduled scraper** (GitHub Action or pg_cron) runs weekly
- **Change detection**: compare content hash to last ingested version
- **Staleness alert**: if any chunk has `last_verified > 90 days`, flag in admin dashboard
- **Manual override**: curated JSON files in `knowledge/` take precedence over auto-ingested content

### Bilingual Chunking
Each knowledge chunk is stored with both English and Nepali text:
```sql
INSERT INTO knowledge_base (content_type, title_en, title_np, content_en, content_np, source_url, embedding)
VALUES (
  'form_field',
  'Form 80 — Question 1: Family Name',
  'फारम ८० — प्रश्न १: थर',
  'Enter your family name exactly as it appears on your passport...',
  'तपाईंको राहदानीमा जस्तो छ त्यस्तै थर लेख्नुहोस्...',
  'https://immi.homeaffairs.gov.au/form-80',
  '[1536-dim vector]'
);
```

---

## 12. Infrastructure & Deployment

### Technology Stack

| Layer | Technology | Why |
|-------|-----------|-----|
| **Frontend** | Next.js 14 (App Router) + PWA | Mobile-first; works offline; no App Store needed |
| **UI Components** | shadcn/ui + Tailwind CSS | Bilingual-ready; accessible; customizable |
| **i18n** | next-intl | Server + client translations; Devanagari font support |
| **Light API** | Supabase Edge Functions (Deno/TS) | Collocated with DB; fast; same as Macrofi |
| **AI API** | FastAPI (Python) + Railway | Heavy pipelines; streaming; RAG |
| **Primary AI** | Claude Sonnet 4.5 (Anthropic) | Best Nepali support; vision + text; structured JSON |
| **Fallback AI** | Claude Haiku 4.5 | Cheaper; good for simple explanations |
| **Database** | Supabase PostgreSQL + pgvector | Auth + data + vector search in one |
| **Auth** | Supabase Auth (email + Google OAuth) | Free; JWT; RLS integration |
| **Storage** | Supabase Storage (user-scoped buckets) | Document uploads; RLS policies |
| **Scheduling** | pg_cron (PostgreSQL) | Visa reminders; knowledge refresh |
| **Push Notifications** | Web Push API | Free; no third-party service needed |
| **Hosting** | Vercel (frontend) + Railway (FastAPI) | Free tiers cover POC |
| **Monitoring** | Sentry (free tier) | Error tracking for both frontend + API |
| **CI/CD** | GitHub Actions | Deploy on push to main |

### POC Cost Estimate

| Service | Plan | Monthly Cost |
|---------|------|-------------|
| Supabase | Free (2 projects, 500MB DB) | $0 |
| Vercel | Hobby (100GB bandwidth) | $0 |
| Railway | Starter ($5 credit) | $0 |
| Claude API | ~5K calls/month × $0.003/1K tokens | ~$5 |
| OpenAI Embeddings | ~10K tokens/month | ~$0.01 |
| Sentry | Free (5K events) | $0 |
| **TOTAL POC** | | **~$5/month** |

### Environments
```
local    → supabase start + `npm run dev` + `uvicorn api.app:app`
staging  → Supabase staging project + Vercel preview deploys
production → Supabase Pro + Vercel production + Railway
```

---

## 13. Security Architecture

```
Layer              Control
─────────────────────────────────────────────────────────
Transport          HTTPS everywhere (Vercel + Railway + Supabase)
Authentication     Supabase Auth JWT (RS256), 1hr access + refresh token
Authorization      Row Level Security on ALL PostgreSQL tables
API Keys           ONLY in server-side env vars (Edge Function secrets + FastAPI .env)
                   NEVER in client bundle
Document Storage   Supabase Storage with user-scoped signed URLs
Token Storage      httpOnly cookies (PWA) or expo-secure-store (if native wrapper)
AI Requests        No PII in logs; Claude API data retention reviewed
Data Deletion      CASCADE on user delete; user-requested deletion < 30 days
Logging            No PII in Sentry; user_id only for debugging
PII Handling       Passport numbers, DOBs encrypted at rest in Supabase
```

---

## 14. Architecture Decision Records

### ADR-001: Hybrid Backend (Edge Functions + FastAPI)
**Status:** Accepted
**Decision:** Use Supabase Edge Functions for simple CRUD (F1, F2, F3) and FastAPI for AI-heavy pipelines (F4).
**Rationale:** Edge Functions are fast, collocated with DB, and sufficient for CRUD. FastAPI handles streaming, complex Python ML/AI libraries, and longer execution times needed for RAG + document extraction pipelines.

### ADR-002: Rules Engine for Points Calculator (Not LLM)
**Status:** Accepted
**Decision:** F2 Points Calculator is a deterministic JSON rules engine, not an LLM.
**Rationale:** Points calculation is high-stakes, must be correct every time, and the rules don't change frequently. An LLM could hallucinate. A JSON config file + test suite guarantees correctness.

### ADR-003: Decision Tree for Checklist (Not LLM)
**Status:** Accepted
**Decision:** F3 Document Checklist uses branching decision trees stored as JSON, not an LLM.
**Rationale:** Checklist generation is deterministic — the same visa type + answers always produces the same checklist. LLM adds cost, latency, and hallucination risk with no benefit for POC. LLM enhancement (e.g., unusual document handling) can be added in Phase 2.

### ADR-004: pgvector for RAG (Not Pinecone)
**Status:** Accepted
**Decision:** Use pgvector extension in Supabase for vector storage and similarity search.
**Rationale:** Already using Supabase — adding pgvector is free with zero additional infrastructure. Pinecone would add cost ($70+/mo) and complexity. The knowledge base is small enough (< 10K chunks) that pgvector's IVFFlat index is more than sufficient.

### ADR-005: Web Push API (Not OneSignal/Firebase)
**Status:** Accepted
**Decision:** Use native Web Push API for visa expiry reminders.
**Rationale:** PWA supports Web Push natively. No third-party service, no cost, no vendor lock-in. Sufficient for POC's simple reminder needs (1-2 notifications per user per month).

### ADR-006: Claude Sonnet as Primary AI
**Status:** Accepted
**Decision:** Claude Sonnet 4.5 as the primary AI model for F4 (Form Helper + Scan Pipeline).
**Rationale:** Best Nepali (Devanagari) support of any commercial model. Native vision capabilities for document scanning. Structured JSON output mode. Claude Haiku 4.5 as cost-saving fallback for simpler explanations.

---

## 15. Build Schedule

### Phase 0 — Foundation (Weeks 1–2)
| Week | Tasks | Deliverable |
|------|-------|-------------|
| W1 | Repo setup (monorepo), Supabase project, Next.js + shadcn/ui skeleton, i18n setup with EN/NP, design system tokens | Blank app with bilingual shell, auth working |
| W2 | Supabase schema (users, visas, knowledge_base), Edge Function template, FastAPI skeleton, CI/CD pipeline | Database + APIs running locally |

**Exit criteria:** Full dev environment boots with one command. Auth working. Schema deployed.

### Phase 1 — F3 Document Checklist (Weeks 2–3)
| Week | Tasks | Deliverable |
|------|-------|-------------|
| W2-3 | Build visa-type selection UI, branching questionnaire component, decision tree engine, checklist renderer with collapsible items, print/export | Working F3 for all 6 visa types |
| W3 | Populate knowledge base with all checklist items (EN+NP), test with all branching paths | Complete, verified checklist content |

**Exit criteria:** User can select any supported visa type, answer questions, and generate + print a correct checklist with Nepali explanations.

### Phase 2 — F2 Points Calculator (Weeks 3–4)
| Week | Tasks | Deliverable |
|------|-------|-------------|
| W3 | Build step-by-step wizard component, JSON rules engine, points calculation logic | Calculator engine working |
| W4 | Results screen with breakdown, SkillSelect comparison, save/share, disclaimer integration | Complete F2 with test suite (10 profiles verified) |

**Exit criteria:** Calculator produces correct score for all 10 test profiles. Passes CI with 100% accuracy.

### Phase 3 — F1 Visa Tracker (Weeks 4–6)
| Week | Tasks | Deliverable |
|------|-------|-------------|
| W4-5 | Visa CRUD UI (add/edit/delete), dashboard with countdown timer, condition cards, next-steps display | Working F1 UI |
| W5 | pg_cron reminder job, Web Push API integration, offline caching via Service Worker | Reminders firing, offline support |
| W6 | Multi-visa support, notification preferences | Complete F1 |

**Exit criteria:** User can track multiple visas, receive reminders at 180/90/30/7 days, view tracker offline.

### Phase 4 — F4 Form Helper + Scan Pipeline (Weeks 6–10)
| Week | Tasks | Deliverable |
|------|-------|-------------|
| W6-7 | RAG pipeline: knowledge ingestion scraper, pgvector setup, embedding generation, retrieval endpoint | RAG answering form field questions |
| W7-8 | Form Helper UI: form selection, field list, bilingual explanation display with citations | Working F4a (Form Explainer) |
| W8-9 | Scan pipeline: upload API, classification, schema extraction, open extraction, confidence tiering | Document extraction working |
| W9-10 | Review UI (side-by-side extraction + source), PDF fill (AcroForm via pdf-lib), audit log, export | Complete F4 with end-to-end scan → fill |

**Exit criteria:** User can get a correct Nepali explanation for any supported form field. User can upload passport + payslips → review extractions → download filled Form 80.

### Phase 5 — Polish (Weeks 10–12)
| Week | Tasks | Deliverable |
|------|-------|-------------|
| W10 | Accessibility audit (WCAG 2.1 AA), performance optimization, PWA installability | Lighthouse score ≥ 90 |
| W11 | Legal review (confirm Form Helper doesn't constitute immigration assistance), privacy policy, T&Cs | Legal sign-off |
| W12 | Beta testing with 10-20 Nepalese migrants, feedback collection, bug fixes | Beta-ready product |

---

## 16. Open Questions & Risks

| # | Question / Risk | Impact | Status | Resolution By |
|---|----------------|--------|--------|---------------|
| 1 | AcroForm field names for Form 80 — need to verify actual field names by inspecting PDF | High — blocks F4 PDF fill | 🔴 Open | Before W8 |
| 2 | Claude data retention for migration documents — confirm with Anthropic | Medium — privacy compliance | 🔴 Open | Before launch |
| 3 | SkillSelect invitation round data freshness — manual update process | Low — affects F2 comparison accuracy | 🟡 Monitor | Monthly review |
| 4 | Home Affairs website structure changes — may break knowledge scraper | Medium — affects F4 accuracy | 🟡 Monitor | Weekly change detection |
| 5 | Nepali-script field values in PDFs — AcroForm may not support Unicode | Medium — affects F4 fill | 🔴 Open | Test during W9 |
| 6 | Web Push API support on iOS — historically limited | Low — affects F1 reminders | 🟡 Monitor | Test during W5 |
| 7 | Cost scaling: Claude API if product goes viral in Nepalese Facebook groups | Medium — cost spike | 🟡 Monitor | Set cost alerts at $50/mo |
| 8 | Legal: does auto-fill constitute "preparing a form"? (see legal-memo.md) | High — regulatory risk | 🟡 Mitigated | Answer sheet fallback available |

---

*Architecture compiled: August 8, 2026*
*Next: See docs/architecture/ for per-feature detailed specs*