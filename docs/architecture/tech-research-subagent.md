# Saathi — Technical Architecture Research

**Project:** Saathi — AI Settlement & Immigration Companion for Nepalese Diaspora in Australia  
**Document:** Architecture Research & Technology Recommendations  
**Date:** August 8, 2026  
**Author:** Architecture Research (based on PRD v2.0, F4 Scan Architecture, Macrofi patterns)  
**Status:** Research complete — ready for implementation decisions

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [F1 — Visa Tracker Architecture](#2-f1--visa-tracker)
3. [F2 — Points Calculator Architecture](#3-f2--points-calculator)
4. [F3 — Document Checklist Architecture](#4-f3--document-checklist)
5. [F4 — Form Helper Architecture](#5-f4--form-helper--scanfill)
6. [Cross-Cutting Architecture](#6-cross-cutting-architecture)
7. [Macrofi Pattern Comparison](#7-macrofi-pattern-comparison)
8. [Build-vs-Buy Decisions](#8-build-vs-buy-decisions)
9. [Implementation Sequence](#9-implementation-sequence)

---

## 1. Executive Summary

### Recommended Tech Stack

| Layer | Choice | Rationale |
|-------|--------|-----------|
| **Frontend** | Next.js 14+ PWA (App Router) | Mobile-first PWA, service workers for offline, SSR where needed |
| **Main Backend** | **FastAPI (Python)** → Railway | Best for: complex rules engine (F2), RAG pipeline (F4), PDF manipulation, bilingual processing. Macrofi used pure Edge Functions — but Saathi's compute profile is different. |
| **Lightweight Backend** | Supabase Edge Functions (Deno/TS) | For: auth hooks, push notification dispatch, pg_notify listeners, lightweight API endpoints (GET visa, GET checklist). Complement FastAPI, don't replace it. |
| **LLM** | Claude Sonnet 4 (Anthropic) | Best Nepali generation; Vision for document scanning (F4); prompt caching cuts costs 90% on repeated system prompts |
| **Database** | Supabase PostgreSQL | RLS, pgvector, pg_cron, pg_notify — all in one; proven pattern from Macrofi |
| **Vector DB** | **pgvector** (in Supabase) | Zero additional infrastructure; collocated with app data; sufficient for Saathi's scale (thousands of document chunks, not millions) |
| **Auth** | **Supabase Auth** | Built-in RLS integration; social login; session management; Macrofi validation; simpler than NextAuth for this stack |
| **Push Notifications** | **OneSignal** → then Supabase Push | OneSignal has the best PWA support + Nepali-language segmentation. Supabase Push maturing but not PWA-optimal yet. |
| **Background Jobs** | pg_cron + FastAPI background tasks | pg_cron for DB-level scheduling (reminder checks, data freshness); FastAPI BackgroundTasks for async processing |
| **Cache** | Supabase query cache + Claude prompt cache | Two-layer: DB query caching (built-in) + Claude prompt caching (cached system prompts save 90% on repeated calls) |
| **Hosting** | Vercel (frontend) + Railway (FastAPI) + Supabase (DB/Edge) | Proven, cheap at MVP scale, scales vertically before needing k8s |
| **CI/CD** | GitHub Actions | Deploy FastAPI to Railway, Edge Functions via Supabase CLI, Frontend to Vercel |
| **Monitoring** | Sentry + Supabase Logs | Same as Macrofi pattern (Sentry for errors, Supabase dashboard for DB perf) |

### Architecture Decision: FastAPI + Edge Functions Hybrid

**Why not pure Edge Functions like Macrofi?**

Macrofi's scan-service is a **single-purpose service** — receive image, call AI, store result. Edge Functions work beautifully for this: 150ms cold start, Deno runtime, collocated with DB. But Saathi has four distinct features with different compute profiles:

| Feature | Compute Profile | Edge Functions Fit? |
|---------|----------------|---------------------|
| F1 Visa Tracker | Lightweight CRUD + scheduled jobs | ✅ Yes — good fit |
| F2 Points Calculator | Complex deterministic rules engine with 12+ scoring criteria | ❌ Poor — needs Python's rules clarity; testing infrastructure |
| F3 Document Checklist | Branching logic + content management | ✅ Partially — simple queries OK; content mgmt needs file system access |
| F4 Form Helper | RAG pipeline, PDF manipulation, bilingual processing, Vision API calls | ❌ Poor — needs Python ecosystem (pdfrw, pymupdf, langchain/llamaindex); Vision API calls may exceed 10s Edge Function timeout |

**Hybrid approach:**
- **FastAPI** handles: F2 (calculator), F4 (RAG + extraction + PDF fill), admin/content management endpoints, bilingual processing pipeline
- **Edge Functions** handle: auth webhooks, F1 (lightweight CRUD endpoints), F3 (checklist queries), push notification dispatch, pg_notify listeners
- Both share: Supabase DB, Supabase Auth, Supabase Storage

This gives us the best of both worlds: FastAPI's rich ecosystem for the hard problems, Edge Functions for cheap, fast, collocated endpoints.

---

## 2. F1 — Visa Tracker

### 2.1 Feature Summary
User enters visa subclass + grant date. System shows: expiry, days remaining, key conditions, "next step." Push reminders at 180, 90, 30, 7 days before expiry.

### 2.2 Push Notification Architecture

**Recommendation: OneSignal (MVP) → Supabase Push (post-MVP)**

| Option | PWA Support | Nepali Segmentation | Free Tier | Complexity | Verdict |
|--------|------------|---------------------|-----------|------------|--------|
| **OneSignal** | ✅ Excellent — built for PWA | ✅ Language-based segments, timezone-aware delivery | ✅ 10K subscribers free | Low — 1 SDK, 1 API key | **MVP choice** |
| Firebase Cloud Messaging | ❌ Android/iOS native only; no PWA | ❌ No language segmentation | ✅ Free | Medium — requires Firebase project | ❌ No PWA support without native wrapper |
| Supabase Push | ⚠️ Experimental; requires FCM/APNs bridge | ❌ No built-in segmentation | ✅ Free (on Supabase) | High — DIY bridge to FCM/APNs | **Future choice** when stable |
| Web Push API (native) | ✅ Native browser API | ❌ Manual segmentation; no dashboard | ✅ Free | High — build everything from scratch | ❌ Too much infra for MVP |

**OneSignal specifics:**
- PWA SDK: `@onesignal/onesignal-pwa-sdk` — registers service worker, handles permission prompt
- Segmentation: tag users by `visa_type`, `language_preference`, `expiry_window`
- Templates: pre-define Nepali reminder templates at each interval (180, 90, 30, 7 days)
- API: trigger from FastAPI or Edge Function via REST API
- Delivery: timezone-aware (Australia/Sydney, Australia/Melbourne)

**Migration path to Supabase Push:**
When Supabase Push matures (expected late 2026/2027), migrate by:
1. Add Supabase Push as secondary channel (dual-send during transition)
2. A/B test delivery rates for 2 weeks
3. Flip primary to Supabase Push, deprecate OneSignal

### 2.3 Background Jobs for Reminder Scheduling

**Recommendation: pg_cron (scheduling) + FastAPI (execution)**

Architecture:
```
pg_cron runs every hour:
  SELECT users WHERE visa_expiry - NOW() IN (180, 90, 30, 7 days)
  → INSERT INTO notification_queue (user_id, visa_id, reminder_type, scheduled_for)
  
FastAPI background worker (every 5 min via APScheduler):
  SELECT FROM notification_queue WHERE status='pending' AND scheduled_for <= NOW()
  → Call OneSignal API
  → UPDATE status='sent'
```

**Why pg_cron over external schedulers:**

| Option | Pros | Cons | Verdict |
|--------|------|------|---------|
| **pg_cron** | Native Postgres; no extra infra; transactional (cron job runs inside DB); zero cost | Limited to DB operations; need external service to actually send | **Best for scheduling** |
| APScheduler (Python) | Full control; can call any API; easy to test | Separate process; needs monitoring; state management | **Best for execution** |
| Celery + Redis | Mature; battle-tested | Overkill for hourly cron; adds Redis infra cost ($7-15/mo) | Not worth it at MVP scale |
| Supabase Edge Scheduled Functions | New feature; Deno-based cron | Still beta; cold start on schedule trigger | Evaluate when stable |

**Why the split:**
- pg_cron queries the DB and populates a queue — it stays within Postgres where it's fastest
- FastAPI worker picks up queue items and calls external APIs (OneSignal) — where Python's HTTP client + error handling shines
- Decoupling = pg_cron never blocks on external API calls; the worker can retry failed deliveries

### 2.4 Database Schema

```sql
-- Visa types reference table (cached from Home Affairs)
CREATE TABLE visa_types (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    subclass VARCHAR(10) NOT NULL UNIQUE,        -- '500', '485', '189'
    name_en VARCHAR(200) NOT NULL,               -- 'Student Visa'
    name_np VARCHAR(200) NOT NULL,               -- 'विद्यार्थी भिसा'
    category VARCHAR(50) NOT NULL,               -- 'student', 'graduate', 'skilled', 'employer'
    key_conditions JSONB NOT NULL DEFAULT '[]',  -- [{condition: "8105", description_en: "...", description_np: "..."}]
    typical_next_step_en TEXT,                   -- 'Apply for 485 Temporary Graduate visa'
    typical_next_step_np TEXT,
    source_url TEXT,                             -- Home Affairs page URL
    last_verified_at TIMESTAMPTZ,
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- User's tracked visas
CREATE TABLE user_visas (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
    visa_type_id UUID NOT NULL REFERENCES visa_types(id),
    grant_date DATE NOT NULL,
    expiry_date DATE NOT NULL,
    visa_grant_number VARCHAR(50),               -- Optional: for reference
    status VARCHAR(20) DEFAULT 'active',         -- active, expired, cancelled, replaced
    notes TEXT,                                   -- User's personal notes
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Notification queue for reminders
CREATE TABLE notification_queue (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
    user_visa_id UUID NOT NULL REFERENCES user_visas(id) ON DELETE CASCADE,
    reminder_type VARCHAR(20) NOT NULL,          -- 'expiry_180d', 'expiry_90d', 'expiry_30d', 'expiry_7d'
    status VARCHAR(20) DEFAULT 'pending',        -- pending, sent, failed, skipped
    scheduled_for TIMESTAMPTZ NOT NULL,
    sent_at TIMESTAMPTZ,
    error_message TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_user_visas_user ON user_visas(user_id);
CREATE INDEX idx_user_visas_expiry ON user_visas(expiry_date) WHERE status = 'active';
CREATE INDEX idx_notification_queue_status ON notification_queue(status, scheduled_for);

-- RLS policies
ALTER TABLE user_visas ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Users can only see their own visas" ON user_visas
    FOR ALL USING (auth.uid() = user_id);

ALTER TABLE notification_queue ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Users can only see their own notifications" ON notification_queue
    FOR SELECT USING (auth.uid() = user_id);
```

### 2.5 pg_cron Job

```sql
-- Run every hour: enqueue upcoming reminders
SELECT cron.schedule(
    'visa-reminder-check',
    '0 * * * *',  -- Every hour
    $$
    INSERT INTO notification_queue (user_id, user_visa_id, reminder_type, scheduled_for)
    SELECT 
        uv.user_id,
        uv.id,
        CASE 
            WHEN uv.expiry_date - CURRENT_DATE = 180 THEN 'expiry_180d'
            WHEN uv.expiry_date - CURRENT_DATE = 90  THEN 'expiry_90d'
            WHEN uv.expiry_date - CURRENT_DATE = 30  THEN 'expiry_30d'
            WHEN uv.expiry_date - CURRENT_DATE = 7   THEN 'expiry_7d'
        END,
        NOW()
    FROM user_visas uv
    WHERE uv.status = 'active'
      AND uv.expiry_date - CURRENT_DATE IN (180, 90, 30, 7)
      AND NOT EXISTS (
          SELECT 1 FROM notification_queue nq 
          WHERE nq.user_visa_id = uv.id 
            AND nq.reminder_type = CASE 
                WHEN uv.expiry_date - CURRENT_DATE = 180 THEN 'expiry_180d'
                WHEN uv.expiry_date - CURRENT_DATE = 90  THEN 'expiry_90d'
                WHEN uv.expiry_date - CURRENT_DATE = 30  THEN 'expiry_30d'
                WHEN uv.expiry_date - CURRENT_DATE = 7   THEN 'expiry_7d'
            END
      );
    $$
);
```

### 2.6 Visa Condition Data Freshness

**Recommendation: Cached from Home Affairs, refreshed weekly via scraper**

Strategy:
1. **Weekly scraper job** (FastAPI + APScheduler) hits Home Affairs visa pages
2. Extracts: visa conditions, eligibility, "next steps"
3. Diffs against current `visa_types` table
4. If changed → update record, bump `last_verified_at`, log in `content_changelog`
5. If unchanged → bump `last_verified_at` only (shows freshness without noise)

**Why not live scraping per request:**
- Home Affairs pages change rarely (months between updates)
- Live scraping adds 2-5s latency to every visa tracker load
- Rate limiting risk from Home Affairs servers

**Fallback if scraper fails:**
- Serve stale data with a warning: "Last updated [date]. Check [Home Affairs link] for latest."
- Never show no data — stale with a warning is better than a blank page

---

## 3. F2 — Points Calculator

### 3.1 Scoring Engine Architecture

**Recommendation: Static JSON config + Python rules engine (deterministic, auditable)**

**Why NOT LLM-driven:**
- Points calculation is **deterministic** — given the same inputs, the same score MUST result
- LLMs can hallucinate point values; a single point error can mislead a user about their eligibility
- Testing an LLM calculator requires statistical sampling (was it right 95% of the time?)
- Testing a rules engine is exact (did it produce _exactly_ 85 points for this profile?)

**Why NOT database-driven:**
- Points criteria change rarely (1-2x per year when Home Affairs updates the points table)
- Database-driven means migrations for every criteria change → slower iteration
- JSON config is version-controlled, reviewable in PRs, deployable instantly

**Recommended approach:**

```python
# points_config.yaml — version controlled, reviewed, auditable
version: "2026-07-01"
last_updated: "2026-07-01"
source: "https://immi.homeaffairs.gov.au/visas/getting-a-visa/visa-listing/skilled-independent-189/points-table"
scoring_criteria:
  age:
    category: "age"
    label_en: "Age at time of invitation"
    label_np: "निमन्त्रणाको समयमा उमेर"
    brackets:
      - range: [18, 24]
        points: 25
      - range: [25, 32]
        points: 30
      - range: [33, 39]
        points: 25
      - range: [40, 44]
        points: 15
      - range: [45, 100]
        points: 0
    validation:
      min_age: 18
      max_age: 44
      disqualifying: "Applicants aged 45+ are not eligible for GSM points test"

  english_language:
    category: "english"
    label_en: "English Language Ability"
    label_np: "अंग्रेजी भाषा क्षमता"
    test_types:
      - name: "IELTS"
        scoring:
          - band: "≥ 8.0 in each component"
            points: 20
            label: "Superior"
          - band: "≥ 7.0 in each component"
            points: 10
            label: "Proficient"
          - band: "≥ 6.0 in each component"
            points: 0
            label: "Competent"
      - name: "PTE Academic"
        scoring:
          - band: "≥ 79 in each component"
            points: 20
            label: "Superior"
          - band: "≥ 65 in each component"
            points: 10
            label: "Proficient"
          - band: "≥ 50 in each component"
            points: 0
            label: "Competent"
      # ... TOEFL, OET
  # ... 10 more criteria
```

```python
# points_engine.py — deterministic calculator
from dataclasses import dataclass
from typing import List
import yaml

@dataclass
class PointsResult:
    total_points: int
    breakdown: List[dict]  # [{category, points, label_en, label_np, explanation}]
    improvement_areas: List[dict]  # [{category, current_points, potential_points, action}]
    minimum_eligible: bool  # Must be ≥ 65 for 189/190/491 EOI
    disclaimer: str

class PointsCalculator:
    def __init__(self, config_path: str = "points_config.yaml"):
        with open(config_path) as f:
            self.config = yaml.safe_load(f)
    
    def calculate(self, profile: dict) -> PointsResult:
        breakdown = []
        total = 0
        
        for criterion_key, criterion in self.config['scoring_criteria'].items():
            points = self._score_criterion(criterion, profile)
            breakdown.append({
                'category': criterion['category'],
                'points': points,
                'label_en': criterion['label_en'],
                'label_np': criterion['label_np'],
            })
            total += points
        
        return PointsResult(
            total_points=total,
            breakdown=breakdown,
            improvement_areas=self._find_improvements(profile),
            minimum_eligible=total >= 65,
            disclaimer="This is an estimate for your information only. For a formal assessment, consult a registered migration agent (MARN)."
        )
    
    def _score_criterion(self, criterion: dict, profile: dict) -> int:
        # Bracket matching, validation, edge cases
        ...
```

### 3.2 SkillSelect Invitation Round Data

**Recommendation: Manual update + admin UI**

- SkillSelect publishes invitation round data at irregular intervals (3-6 months)
- The data is a simple table: minimum points per occupation, invitations issued, date
- Web scraping is fragile — the page format changes; at MVP scale, manual update is faster and more reliable
- Build a simple admin UI: paste the latest round data → system parses → updates `invitation_rounds` table
- Automate scraping only when the admin UI becomes a bottleneck (100+ updates/year — won't happen)

```sql
CREATE TABLE invitation_rounds (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    round_date DATE NOT NULL,
    visa_subclass VARCHAR(10) NOT NULL,         -- '189', '190', '491'
    occupation_code VARCHAR(10),                 -- ANZSCO code
    occupation_name VARCHAR(200),
    minimum_points INTEGER,
    invitations_issued INTEGER,
    source_url TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 3.3 Testing Strategy

**Recommendation: Golden profile set + property-based testing**

```
tests/
├── fixtures/
│   ├── profiles/                # 20+ hand-crafted profiles
│   │   ├── max_points.json      # 45-point profile (everything maxed)
│   │   ├── min_eligible.json    # Exactly 65 points
│   │   ├── student_grad.json    # Typical 485 → 189 pathway
│   │   ├── regional.json        # 491 regional pathway
│   │   └── ...
│   └── expected_scores.json     # Expected scores for each profile
├── test_points_engine.py
└── conftest.py
```

**Test layers:**
1. **Unit tests:** Each scoring criterion tested in isolation — age bracket boundaries (17, 18, 24, 25, 32, 33, 44, 45, 46), English test score mappings, partner points combinations
2. **Integration tests:** Full profiles → expected scores. Regressions caught on PR.
3. **Property tests:** `hypothesis` library — generate random valid profiles, verify: 0 ≤ total ≤ 130 (max possible), each category within its min/max range, disqualification conditions correctly zero out score
4. **Snapshot tests:** Each time the config changes, `expected_scores.json` must be updated and reviewed as part of the PR

**Config review process:**
When Home Affairs updates the points table:
1. Update `points_config.yaml`
2. Run `pytest tests/test_points_engine.py` — existing profiles may break (expected!)
3. Update `expected_scores.json` for any legitimate score changes
4. PR description MUST include: link to official Home Affairs update; summary of what changed; impact on typical profiles

---

## 4. F3 — Document Checklist

### 4.1 Decision Engine Architecture

**Recommendation: Hybrid — Rule-based decision tree + LLM explanation generation**

The branching logic (which questions to ask, which documents to include) is **deterministic** — it should be a decision tree. The explanations (what each document is, how to get it, common mistakes) are **generative** — LLM excels here.

```
User selects visa type (e.g., 485 Graduate Work)
  ↓
Decision Tree Engine (rule-based)
  ├── Question 1: "Did you study in Australia?"
  │   ├── YES → include: completion letter, academic transcript
  │   └── NO  → skip
  ├── Question 2: "Do you have a partner to include?"
  │   ├── YES → include: partner passport, relationship evidence, English test (for points)
  │   └── NO  → skip
  ├── Question 3: "Are you applying onshore or offshore?"
  │   ├── ONSHORE → include: current visa grant notice, Australian address
  │   └── OFFSHORE → include: offshore police clearance
  ├── Question 4: "Have you completed an Australian police check in the last 12 months?"
  │   └── ...
  └── → Generates checklist JSON
  ↓
LLM (Claude Sonnet) — explanation generation
  ├── For each document in checklist:
  │   ├── What it is (in Nepali)
  │   ├── Where to get it (specific agency, URL, cost, processing time)
  │   ├── Common mistakes (in Nepali)
  │   └── Format requirements
  └── → Cached explanations (LLM called once per document type, not per user)
  ↓
Rendered checklist (printable, sharable)
```

### 4.2 Content Management Strategy

**Recommendation: Markdown files + Claude pre-generation + Database cache**

```
content/
├── checklists/
│   ├── 485-graduate-work/
│   │   ├── tree.yaml              # Decision tree (which questions, which docs)
│   │   ├── documents/             # Per-document explanations
│   │   │   ├── completion-letter.md
│   │   │   ├── academic-transcript.md
│   │   │   ├── english-test.md
│   │   │   └── ...
│   │   └── metadata.yaml          # Source URLs, last verified dates
│   ├── 485-post-study-work/
│   ├── 189/
│   ├── 190/
│   └── ...
```

**Why Markdown, not a CMS:**
- Version-controlled in git — PR reviewable, diff-able
- No CMS overhead (no Strapi/Contentful subscription at MVP)
- Markdown → easy to translate to Nepali (LLM can do initial translation, human reviews)
- Can be served directly or pre-rendered to HTML

**Freshness strategy:**
- Each document's metadata has a `last_verified` date and `source_url`
- Weekly pg_cron job checks: documents where `last_verified < 30 days ago` → flag for review
- Admin dashboard shows "stale documents" with links to Home Affairs source pages
- When Home Affairs changes a requirement → update the markdown, bump `last_verified`

```sql
CREATE TABLE document_templates (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    visa_subclass VARCHAR(10) NOT NULL,
    stream VARCHAR(50),                          -- 'graduate-work', 'post-study-work', NULL
    document_key VARCHAR(100) NOT NULL,           -- 'completion-letter', 'english-test'
    title_en VARCHAR(200) NOT NULL,
    title_np VARCHAR(200) NOT NULL,
    description_np TEXT,                          -- LLM-generated explanation
    how_to_obtain_np TEXT,
    common_mistakes_np TEXT[],
    format_requirements TEXT,
    source_url TEXT,
    last_verified_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE(visa_subclass, stream, document_key)
);

CREATE TABLE checklist_trees (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    visa_subclass VARCHAR(10) NOT NULL UNIQUE,
    tree_config JSONB NOT NULL,                  -- Serialized decision tree
    last_verified_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE user_checklists (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
    visa_subclass VARCHAR(10) NOT NULL,
    answers JSONB NOT NULL,                      -- User's answers to branching questions
    documents JSONB NOT NULL,                    -- Generated checklist items
    created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 4.3 Decision Tree Implementation

```yaml
# tree.yaml — declarative decision tree
visa_subclass: "485"
streams:
  graduate-work:
    questions:
      - id: "australian_study"
        label_en: "Did you complete your study in Australia?"
        label_np: "के तपाईंले अष्ट्रेलियामा अध्ययन पूरा गर्नुभयो?"
        type: boolean
        affects: ["completion_letter", "academic_transcript", "australian_study_evidence"]
        always_required: false
        
      - id: "partner_included"
        label_en: "Are you including a partner or dependents in your application?"
        label_np: "के तपाईं आफ्नो आवेदनमा पार्टनर वा आश्रित समावेश गर्दै हुनुहुन्छ?"
        type: boolean
        affects: ["partner_passport", "relationship_evidence", "partner_english"]
        always_required: false

      - id: "skill_assessment"
        label_en: "Have you completed a skills assessment?"
        label_np: "के तपाईंले सीप मूल्यांकन पूरा गर्नुभयो?"
        type: boolean
        affects: []
        always_required: true  # Always required for 485 Graduate Work
        document: "skills_assessment"

    always_required_docs:
      - "passport"
      - "birth_certificate"
      - "visa_grant_notice"
      - "australian_police_check"
      - "overseas_police_check"  # if lived overseas 12+ months in last 10 years
      - "passport_photo"
      - "health_examination"
      - "form_80"
```

---

## 5. F4 — Form Helper & Scan/Fill

> **Note:** The detailed scan/form-fill pipeline is covered in `architecture-doc-scan-form-fill.md`. This section covers the NEW research: RAG architecture, vector DB, chunking, citations.

### 5.1 RAG Architecture for Field Explanations

The Form Helper has two modes:
1. **Chat mode:** User asks "What does 'usual country of residence' mean on Form 80?" → RAG retrieves relevant Home Affairs documentation → Claude generates Nepali explanation with citation
2. **Scan mode:** User uploads form page → Claude Vision classifies + extracts → maps to form fields → provides field-by-field explanations (covered in existing doc)

**RAG Pipeline:**

```
                ┌────────────────────────────────────────┐
                │         Ingestion Pipeline (weekly)      │
                │                                          │
  Home Affairs  │  scrape → clean → chunk → embed → store │
  Pages ───────▶│  (httpx) (BeautifulSoup)  (Voyage AI)  │
                │                    ↓                     │
                │          pgvector (Supabase)             │
                └────────────────────────────────────────┘

                ┌────────────────────────────────────────┐
                │         Retrieval Pipeline (per query)   │
                │                                          │
  User asks ───▶│  embed query → vector search (top 5)    │
  "What is      │       ↓                                  │
  Form 80       │  + keyword search (top 3) → dedup       │
  Part C?"      │       ↓                                  │
                │  rerank by relevance → top 3 chunks     │
                │       ↓                                  │
                │  Claude Sonnet: generate Nepali answer   │
                │  + citations to source URLs              │
                └────────────────────────────────────────┘
```

### 5.2 Vector Database: pgvector over Pinecone

**Decision: pgvector (in Supabase)**

| Criterion | pgvector (Supabase) | Pinecone | Verdict |
|-----------|---------------------|----------|---------|
| **Cost** | Included in Supabase Pro ($25/mo) | $70/mo (starter) | pgvector wins |
| **Infrastructure** | Zero — same DB | Separate service to manage | pgvector wins |
| **Data colocation** | Vectors + metadata + user data in one DB | Metadata lives elsewhere | pgvector wins for RLS |
| **Performance** | IVFFlat/HNSW indexes; <10ms for <100K vectors | Optimized; sub-5ms at scale | Pinecone wins at 1M+ vectors |
| **RLS integration** | Native — `WHERE auth.uid() = user_id` | Must implement at app layer | pgvector wins |
| **Saathi scale** | Will have ~1-2K document chunks (enough for years) | Built for millions | pgvector is sufficient |

**When to switch to Pinecone:** Only when Saathi reaches 100K+ document chunks (years of immigration content, forum posts, etc.) AND query latency exceeds 50ms AND you have revenue to justify the cost. Not before.

### 5.3 Chunking Strategy for Bilingual Content

**Recommendation: Semantic chunking with bilingual overlap**

Home Affairs documents are dense, structured, and contain both English (legal text) and context that needs Nepali explanation. Chunking strategy:

```
Source: "Form 80 — Personal Particulars for Character Assessment"
         ┌──────────────────────────────────────┐
         │ English original (scraped from Home   │
         │ Affairs + form fill instructions)      │
         │ ─────────────────────────────────────  │
         │ Nepali translation/explanation         │
         │ (Claude-generated, human-reviewed)     │
         └──────────────────────────────────────┘

Chunking approach:
1. Split by section headings (e.g., "Part C — Citizenship", "Part D — Address History")
2. Each chunk = one logical concept (a form section, a field group, an instruction block)
3. Chunk metadata: { section_id, form_name, part_label, is_nepali, source_url, last_updated }
4. Store BOTH English and Nepali text as separate chunks with mutual references
5. At query time: if user asks in Nepali, prefer Nepali chunks; if English, English chunks; always include both in context for Claude to cross-reference
```

```
chunk_size: ~500 tokens (large enough for a form section, small enough for precise retrieval)
chunk_overlap: 50 tokens (catch boundary-crossing concepts)
embedding_model: voyage-multilingual-2 (Voyage AI) — best for bilingual Nepali+English
embedding_dim: 1024
```

**Why Voyage AI over OpenAI/Cohere embeddings:**
- Voyage's multilingual models are specifically trained on low-resource languages including Nepali
- OpenAI's `text-embedding-3-large` performs well on English but degrades on Nepali
- Voyage `multilingual-2` costs $0.06/1M tokens — competitive

### 5.4 Grounding LLM Responses with Citations

**Recommendation: Claude's native Citations feature + chunk metadata**

Claude supports a Citations API that returns source references alongside generated text. Combined with our chunk metadata:

```python
# RAG service — FastAPI endpoint
@router.post("/form-helper/ask")
async def ask_form_question(
    query: FormQueryRequest,
    user_id: UUID = Depends(get_current_user)
):
    # 1. Embed user query
    query_embedding = voyage_client.embed(query.text, model="voyage-multilingual-2")
    
    # 2. Hybrid search: vector + keyword
    vector_results = await pgvector_search(query_embedding, top_k=5)
    keyword_results = await fulltext_search(query.text, top_k=3)
    combined = deduplicate_and_rerank(vector_results, keyword_results)
    
    # 3. Build context with citations
    context_chunks = []
    for chunk in combined:
        context_chunks.append({
            "content": chunk.text,
            "source": chunk.source_url,
            "section": chunk.section_label,
            "last_updated": chunk.last_updated,
            "language": chunk.language  # 'en' or 'np'
        })
    
    # 4. Claude with citations
    response = claude_client.messages.create(
        model="claude-sonnet-4-20250514",
        max_tokens=1024,
        system="""You are Saathi, an AI assistant for Nepalese migrants in Australia.
        Answer questions about Australian immigration forms in Nepali.
        Always cite the source of your information.
        If you don't know, say so. Never make up form field meanings.
        For legal questions, direct users to a registered migration agent (mara.gov.au).
        Always include this disclaimer: "यो जानकारी मात्र हो। कानूनी सल्लाहको लागि दर्ता भएको माइग्रेसन एजेन्टसँग परामर्श गर्नुहोस्।"
        """,
        messages=[{
            "role": "user",
            "content": [
                {"type": "text", "text": f"Context from official Australian immigration pages:\n\n{format_context(context_chunks)}\n\nUser question: {query.text}\n\nPlease answer in Nepali with citations to the source."}
            ]
        }]
    )
    
    # 5. Extract citations from response
    return {
        "answer_np": response.content[0].text,
        "citations": [
            {
                "source_url": chunk["source"],
                "section": chunk["section"],
                "last_updated": chunk["last_updated"]
            }
            for chunk in context_chunks
        ],
        "disclaimer": "यो जानकारी मात्र हो। कानूनी सल्लाहको लागि दर्ता भएको माइग्रेसन एजेन्टसँग परामर्श गर्नुहोस्।",
        "fallback_handoff": None  # Set to "mara.gov.au" if triggered
    }
```

### 5.5 Form Field Manifest (from existing F4 doc — key decisions)

The existing `architecture-doc-scan-form-fill.md` is thorough on the scan pipeline. Key architecture decisions to reinforce:

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Classification | Claude Vision | Handles ambiguous documents; per-call cost negligible at MVP |
| OCR/Extraction | Claude Vision (primary) | Best Nepali/Devanagari support; bounding box coordinates |
| Key-Value Extraction | Schema-driven + Open fallback | Structured output for known fields; open extraction catches unexpected data |
| PDF Fill | pdf-lib (AcroForm) | Most AU immigration forms use AcroForm fields; lossless, reversible |
| Confidence Tiers | ≥85% auto-fill; 50-84% amber confirm; <50% manual | Same pattern as Macrofi's confidence thresholds |

**Reinforced decision on Claude Vision over AWS Textract/GCP DocAI:**
- The existing doc correctly identifies Claude Vision as superior for Nepali documents
- Multi-modal models (Gemini, Claude) are improving rapidly; dedicated OCR services are losing their edge
- At MVP scale, the cost difference is negligible (~$0.01/document vs vendor lock-in with AWS/GCP)
- Re-evaluate at 10K+ documents/month; if Claude costs exceed $50/month, then evaluate Azure DI

---

## 6. Cross-Cutting Architecture

### 6.1 Authentication: Supabase Auth

**Decision: Supabase Auth (confirmed from PRD)**

**Why Supabase Auth over NextAuth.js:**

| Criterion | Supabase Auth | NextAuth.js (Auth.js v5) |
|-----------|--------------|--------------------------|
| RLS integration | Native — `auth.uid()` works automatically | Manual — must pass user_id to every DB query |
| Social login | Built-in (Google, Apple, Facebook, GitHub) | Built-in via providers |
| Session management | JWT + refresh tokens; auto-refresh | JWT or Database sessions |
| Row Level Security | Deep integration with PostgreSQL RLS | No built-in RLS knowledge |
| Pricing | Free for 50K MAU | Free (self-hosted) |
| Macrofi validation | ✅ Already proven in Macrofi scan-service | Not tested |
| Nepali phone auth | Not natively supported | Can add custom provider |

**RLS enforcement pattern (same as Macrofi):**
```sql
-- Every table gets this pattern
ALTER TABLE user_visas ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Users own their data" ON user_visas
    FOR ALL USING (auth.uid() = user_id);
```

**Edge Functions use ANON_KEY + JWT for user-data reads; SERVICE_ROLE_KEY only for background jobs (notification dispatch, cron).**

### 6.2 API Design Patterns

**Recommendation: REST (primary) + Supabase client SDK (direct reads for simple queries)**

```
API Surface:

FastAPI (main backend — Railway):
├── POST   /api/v1/points/calculate          # F2: Calculate points score
├── GET    /api/v1/points/history             # F2: User's calculation history
├── GET    /api/v1/invitation-rounds          # F2: Latest SkillSelect data
├── POST   /api/v1/checklists/generate        # F3: Generate checklist from answers
├── GET    /api/v1/checklists/:id             # F3: Get saved checklist
├── POST   /api/v1/form-helper/ask            # F4: RAG chat query
├── POST   /api/v1/form-helper/scan-document  # F4: Upload + classify + extract
├── POST   /api/v1/form-helper/fill-form      # F4: Fill target PDF with extracted values
├── GET    /api/v1/admin/verify-content       # Admin: View stale content alerts
├── POST   /api/v1/admin/update-rounds        # Admin: Update SkillSelect round data

Supabase Edge Functions (lightweight):
├── POST   /functions/v1/send-notification    # F1: Trigger push notification
├── POST   /functions/v1/sync-visa-data       # F1: Cron webhook for visa data refresh
├── GET    /functions/v1/visa-tracker          # F1: Get user's tracked visas
├── POST   /functions/v1/visa-tracker          # F1: Add/track a visa

Supabase Client SDK (direct from frontend):
├── auth.users                                # Auth operations (login, register, reset)
├── user_visas (read)                         # F1: User reads their own visas (RLS)
├── user_checklists (read)                    # F3: User reads saved checklists (RLS)
├── invitation_rounds (read)                  # F2: Public, anonymous read allowed
```

**API Design Rules:**
1. FastAPI endpoints use JWT validation (same as Macrofi auth middleware pattern)
2. Edge Functions use the same JWT pattern: validate Bearer token, extract `auth.uid()`
3. Direct Supabase SDK reads only for simple CRUD where RLS handles auth — never for business logic
4. All responses follow the same error shape as Macrofi: `{ "error": "CODE", "message": "...", "retryable": bool }`
5. Idempotency keys on mutation endpoints (POST /scan-document, POST /calculate)

### 6.3 Database Schema Design (Full)

```sql
-- === SHARED REFERENCE DATA ===

-- Visa types (F1, F2, F3, F4 all reference this)
CREATE TABLE visa_types (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    subclass VARCHAR(10) NOT NULL UNIQUE,
    name_en VARCHAR(200) NOT NULL,
    name_np VARCHAR(200) NOT NULL,
    category VARCHAR(50) NOT NULL,
    key_conditions JSONB DEFAULT '[]',
    typical_next_step_en TEXT,
    typical_next_step_np TEXT,
    source_url TEXT,
    last_verified_at TIMESTAMPTZ,
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- === F1: VISA TRACKER ===
CREATE TABLE user_visas ( ... );        -- (Section 2.4)
CREATE TABLE notification_queue ( ... ); -- (Section 2.4)

-- === F2: POINTS CALCULATOR ===
CREATE TABLE points_calculations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES auth.users(id) ON DELETE SET NULL,  -- NULL if anonymous
    profile_data JSONB NOT NULL,           -- User's inputs
    result_data JSONB NOT NULL,            -- Calculated scores + breakdown
    config_version VARCHAR(20) NOT NULL,   -- Which points_config.yaml version
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE invitation_rounds ( ... );    -- (Section 3.2)

-- === F3: DOCUMENT CHECKLIST ===
CREATE TABLE document_templates ( ... );   -- (Section 4.2)
CREATE TABLE checklist_trees ( ... );      -- (Section 4.2)
CREATE TABLE user_checklists ( ... );      -- (Section 4.2)

-- === F4: FORM HELPER ===
CREATE TABLE form_definitions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    form_code VARCHAR(20) NOT NULL UNIQUE,      -- '80', '1221', '485-online'
    title_en VARCHAR(200) NOT NULL,
    title_np VARCHAR(200) NOT NULL,
    fields JSONB NOT NULL,                      -- [{field_name, label_en, label_np, type, required, acroform_name}]
    source_url TEXT,
    last_verified_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE document_extractions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
    document_type VARCHAR(50) NOT NULL,         -- 'passport', 'payslip', 'visa_grant', etc.
    source_file_url TEXT NOT NULL,              -- Supabase Storage URL
    extracted_data JSONB NOT NULL,              -- All extracted key-value pairs
    confidence_tiers JSONB NOT NULL,            -- {field: 'HIGH'|'MEDIUM'|'LOW'|'FAIL'}
    model_used VARCHAR(50) NOT NULL,            -- 'claude-sonnet-4'
    classification_confidence DECIMAL(3,2),
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE form_fills (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
    form_code VARCHAR(20) NOT NULL REFERENCES form_definitions(form_code),
    extraction_ids UUID[] NOT NULL,             -- Which document_extractions were used
    filled_fields JSONB NOT NULL,               -- {field_name: value, source_document, confidence}
    output_pdf_url TEXT,                        -- Filled PDF stored in Supabase Storage
    status VARCHAR(20) DEFAULT 'draft',         -- draft, review, complete
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- === RAG / VECTOR STORE ===
CREATE EXTENSION IF NOT EXISTS vector;

CREATE TABLE content_chunks (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    form_code VARCHAR(20),                     -- NULL for general immigration content
    section_label VARCHAR(200),
    content_text TEXT NOT NULL,
    content_text_np TEXT,                      -- Nepali translation if available
    language VARCHAR(2) NOT NULL DEFAULT 'en', -- 'en' or 'np'
    embedding VECTOR(1024),                    -- voyage-multilingual-2 = 1024 dims
    source_url TEXT NOT NULL,
    chunk_index INTEGER,
    last_verified_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX ON content_chunks USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);

-- === ADMIN / CONTENT MANAGEMENT ===
CREATE TABLE content_changelog (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    table_name VARCHAR(100) NOT NULL,
    record_id UUID NOT NULL,
    change_type VARCHAR(20) NOT NULL,           -- 'update', 'verify', 'deprecate'
    previous_hash VARCHAR(64),
    new_hash VARCHAR(64),
    changed_by UUID REFERENCES auth.users(id),
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- === INDEXES ===
-- F1
CREATE INDEX idx_notification_queue_pending ON notification_queue(status, scheduled_for) WHERE status = 'pending';
-- F2
CREATE INDEX idx_points_calculations_user ON points_calculations(user_id, created_at DESC);
-- F3
CREATE INDEX idx_user_checklists_user ON user_checklists(user_id, created_at DESC);
-- F4
CREATE INDEX idx_document_extractions_user ON document_extractions(user_id, created_at DESC);
CREATE INDEX idx_content_chunks_form ON content_chunks(form_code) WHERE form_code IS NOT NULL;
```

### 6.4 Caching Strategy

**Three-layer cache:**

| Layer | What | How | Invalidation |
|-------|------|-----|--------------|
| **Claude Prompt Cache** | System prompts for RAG, extraction, classification | Anthropic's `cache_control` (ephemeral, 5-min TTL) | Automatic — Anthropic handles |
| **Supabase Query Cache** | Frequent reads: visa_types, document_templates, invitation_rounds | Supabase built-in (PostgreSQL shared buffers) | Automatic via TTL or on update |
| **FastAPI Response Cache** | Points calculation results (same profile = same result), checklist templates | `@lru_cache` on pure functions; Redis if needed at scale | Clear on config update |

**Claude Prompt Caching savings:**
- System prompt (RAG instructions): ~500 tokens → cached, 90% cost reduction on repeated calls
- Form definition schemas (extraction prompts): ~200 tokens each → cached
- At 1,000 queries/day: $0.15/day without caching → $0.015/day with caching (10x savings)

### 6.5 Cost Optimization for Claude API Calls

**Monthly cost model at MVP scale (~500 MAU):**

| Feature | Calls/user/month | Tokens/call | Cost/call | Users | Monthly Cost |
|---------|-----------------|-------------|-----------|-------|-------------|
| F2: Points calc explanation | 2 | ~1,000 out | $0.003 | 500 | $3.00 |
| F3: Checklist explanation gen | 1 | ~2,000 out | $0.006 | 500 | $3.00 |
| F4: Form Helper chat | 10 | ~500 in + 800 out | $0.005 | 500 | $25.00 |
| F4: Document scan/extract | 3 | ~500 in + 700 out (vision) | $0.02 | 500 | $30.00 |
| **Total (no caching)** | | | | | **~$61/mo** |
| **Total (with caching)** | | | | | **~$25/mo** |

**Cost optimization techniques:**
1. **Prompt caching** (60% savings on repeated system prompts)
2. **Pre-generated explanations** — common checklist items and form field explanations generated once, cached in DB, served instantly. LLM only called for novel questions or edge cases.
3. **Tiered model routing** — use Claude Haiku for classification (cheap, fast), Claude Sonnet only for generation (quality-dependent)
4. **Rate limiting per user** — 20 free form helper queries/day; premium tier for unlimited
5. **Response caching** — if two users ask the same question within 24 hours, serve cached response (semantic dedup via embedding similarity)

### 6.6 Monitoring & Observability

**Stack (same as Macrofi):**
- **Sentry** — application errors, performance traces (FastAPI + Edge Functions)
- **Supabase Dashboard** — DB performance, query metrics, RLS violations
- **Anthropic Console** — Claude API usage, token counts, latency
- **Custom health dashboard** (optional, post-MVP) — content freshness, failed extractions, notification delivery rate

**Key alerts:**
1. Content freshness: any `visa_types.last_verified_at > 30 days` → alert admin
2. Extraction failure rate: `EXTRACTION_FAILED > 15%` of scans → investigate prompts
3. Claude API spend: daily cost exceeds $5 → review
4. Notification failures: > 5% of push notifications fail → check OneSignal

### 6.7 CI/CD Pipeline

```yaml
# .github/workflows/deploy.yml — simplified
name: Deploy Saathi

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Python tests (FastAPI)
        run: |
          pip install -r requirements.txt
          pytest tests/ --cov
      - name: Run points calculator golden tests
        run: pytest tests/test_points_engine.py --golden

  deploy-fastapi:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Deploy to Railway
        run: railway up --service saathi-api

  deploy-edge-functions:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Deploy Supabase Edge Functions
        run: |
          npx supabase functions deploy send-notification
          npx supabase functions deploy sync-visa-data

  deploy-frontend:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Deploy to Vercel
        run: vercel deploy --prod --token=${{ secrets.VERCEL_TOKEN }}
```

---

## 7. Macrofi Pattern Comparison

### 7.1 What Saathi Can Reuse from Macrofi

| Macrofi Pattern | Applicable to Saathi? | Adaptation Notes |
|----------------|----------------------|-----------------|
| **RLS on every table** | ✅ Directly reusable | Same pattern — every user-data table gets RLS |
| **JWT auth middleware** | ✅ Directly reusable | Same pattern for both FastAPI and Edge Functions |
| **Confidence tiering (0.80/0.50)** | ✅ Directly reusable | F4 scan pipeline uses identical confidence model |
| **Dual-model AI (primary + fallback from different provider)** | ✅ Pattern is reusable | Saathi can use Claude Sonnet (primary) + Gemini Flash (fallback for cost) |
| **pg_notify event bus** | ⚠️ Partially useful | Saathi's features are less event-driven; use for notification dispatch + async processing |
| **Sentry error tracking** | ✅ Directly reusable | Same code pattern, just different endpoints |
| **Idempotency keys** | ✅ Directly reusable | Use on POST /scan-document, POST /calculate |
| **Typed error responses** | ✅ Directly reusable | Same `{error, message, retryable}` shape |
| **Golden test sets** | ✅ Pattern is reusable | Golden profiles for F2; golden documents for F4 extraction |
| **Edge Functions as primary compute** | ❌ Not for Saathi | Saathi needs FastAPI for the heavy features (explained in §1) |

### 7.2 What Saathi Should Do Differently

| Macrofi Decision | Saathi Decision | Why Different |
|-----------------|----------------|---------------|
| Pure Edge Functions | FastAPI + Edge Functions hybrid | Saathi has 4 distinct features with heterogeneous compute needs |
| Single-purpose scan service | Multi-feature platform | Saathi is a product, not a single microservice |
| Mobile-native (React Native) | PWA (Next.js) | Different distribution strategy; Nepalese community finds apps through social/WhatsApp links, not app stores |
| No admin UI needed | Admin UI for content management | Saathi has content that needs regular human verification |
| Free tier AI (Gemini) | Paid Claude (with caching) | Claude is better at Nepali generation; cost is manageable with caching |

### 7.3 Macrofi Patterns to AVOID for Saathi

1. **Don't couple all features in one Edge Function** — Macrofi's single-service model works because it does one thing. Saathi would become unmanageable as a single function.
2. **Don't skip content freshness monitoring** — Macrofi's scan service doesn't need stale content detection. Saathi's visa data goes stale and needs active monitoring.
3. **Don't use free-tier AI for quality-critical output** — Macrofi can tolerate an extraction error (user corrects it). Saathi's form explanations and points calculations must be accurate — Claude's quality matters more than Gemini's free tier.

---

## 8. Build-vs-Buy Decisions

| Component | Build | Buy/Use | Rationale |
|-----------|-------|---------|-----------|
| **Push Notifications** | | OneSignal (free tier) | Building push infra for PWA is weeks of work; OneSignal works in hours |
| **Auth** | | Supabase Auth | Battle-tested, RLS integrated, free for MVP scale |
| **Database** | | Supabase PostgreSQL | RLS + pgvector + pg_cron in one; $25/mo Pro is cheap |
| **LLM** | | Anthropic Claude | Best Nepali; prompt caching; citations API |
| **Embeddings** | | Voyage AI (multilingual-2) | Best low-resource language support including Nepali |
| **Vector DB** | | pgvector (in Supabase) | Zero infra; collocated; sufficient scale |
| **PDF Generation** | Build (pdf-lib) | | AcroForm manipulation is core IP; no good SaaS for AU immigration forms |
| **Frontend** | Build (Next.js PWA) | | Core product; no template gets this right |
| **FastAPI Backend** | Build | | Core business logic; not commoditizable |
| **Content Scraping** | Build (httpx + BS4) | | Home Affairs pages are simple; no need for scraping SaaS |
| **Monitoring** | | Sentry (free) | Same as Macrofi; proven |
| **CI/CD** | Build (GitHub Actions) | | Simple deployment pipeline |
| **RAG Pipeline** | Build (FastAPI + pgvector) | | Core IP; integrates tightly with bilingual content |
| **Points Calculator** | Build (Python engine) | | Deterministic rules; can't buy an AU immigration points calculator |
| **Document Checklist** | Build (decision tree) | | Branching logic is simple; don't over-engineer |
| **Admin UI** | Build (Next.js page) | | Simple CRUD; a single page in the main app |

**Total monthly infrastructure cost at MVP (500 MAU):**
- Supabase Pro: $25
- Railway (FastAPI): $5 (hobby)
- Vercel (Frontend): $0 (hobby)
- Claude API: ~$25 (with caching)
- Voyage AI: ~$2 (embeddings)
- OneSignal: $0 (free tier)
- Sentry: $0 (free tier)
- **Total: ~$57/month**

---

## 9. Implementation Sequence

### Phase 1 — Foundation (Weeks 1-3)
Build the simplest version of each feature — validate user behavior, not perfection.
1. **F2 Points Calculator** — Static HTML prototype → FastAPI endpoint → Next.js form UI. Golden test set. No LLM needed; pure rules engine.
2. **Supabase setup** — project, migrations, RLS, auth flow
3. **CI/CD pipeline** — GitHub Actions deploy to Railway + Vercel

### Phase 2 — Content Features (Weeks 3-6)
4. **F3 Document Checklist** — Decision tree YAML → FastAPI endpoint → Checklist UI. Pre-generate Nepali explanations with Claude (one-time cost).
5. **F1 Visa Tracker** — Data model → pg_cron scheduling → OneSignal integration → Tracker UI
6. **Content ingestion pipeline** — Scrape Home Affairs pages → populate visa_types, document_templates

### Phase 3 — RAG & Scan (Weeks 6-10)
7. **F4 Form Helper (Chat mode)** — Ingestion pipeline → pgvector embeddings → RAG query endpoint → Chat UI
8. **F4 Form Helper (Scan mode)** — Document upload → Claude Vision classification → Schema extraction → Review UI
9. **PDF fill** — Field manifest → pdf-lib AcroForm fill → Export

### Phase 4 — Polish (Weeks 10-12)
10. **Admin UI** — Content freshness dashboard, SkillSelect round data entry, document template editor
11. **Cost optimization** — Implement prompt caching, semantic response dedup, tiered model routing
12. **Performance optimization** — CDN for static content, query optimization, lazy loading

---

## Appendix: Key Technology References

| Technology | URL | Cost |
|-----------|-----|------|
| Claude API (Anthropic) | https://docs.anthropic.com | Pay-per-token |
| Claude Prompt Caching | https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching | 90% discount on cached tokens |
| Claude Citations | https://docs.anthropic.com/en/docs/build-with-claude/citations | Built-in |
| Voyage AI Embeddings | https://docs.voyageai.com | $0.06/1M tokens |
| pgvector | https://github.com/pgvector/pgvector | Free (Postgres extension) |
| pg_cron | https://github.com/citusdata/pg_cron | Free (Postgres extension) |
| pdf-lib | https://pdf-lib.org | Free (open source) |
| OneSignal | https://onesignal.com | Free up to 10K subscribers |
| Supabase | https://supabase.com | $25/mo (Pro) |
| FastAPI | https://fastapi.tiangolo.com | Free |
| Next.js PWA | https://nextjs.org | Free |
| Vercel | https://vercel.com | Free (hobby) |
| Railway | https://railway.app | $5/mo (hobby) |
| Sentry | https://sentry.io | Free (developer) |
| GitHub Actions | https://github.com/features/actions | Free |

---

*Architecture research completed: August 8, 2026*
*Based on: Saathi PRD v2.0, F4 Scan Architecture, Macrofi ARCHITECTURE.md + PHASE-ARCHITECTURE.md, product market research*