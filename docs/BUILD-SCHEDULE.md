# Saathi — Build Schedule & Development Plan

**Version:** 1.0 | **Date:** August 8, 2026

Complete implementation plan with phases, task breakdown, estimates, and dependencies.

---

## Timeline Overview

```
Phase 0: Foundation        Weeks 1-2    → Repo, schema, auth, CI/CD
Phase 1: F3 Checklist      Weeks 2-3    → First shippable feature
Phase 2: F2 Calculator     Weeks 3-4    → Second feature
Phase 3: F1 Visa Tracker   Weeks 4-6    → Third feature + reminders
Phase 4: F4 Form Helper    Weeks 6-10   → RAG + scan pipeline
Phase 5: Polish            Weeks 10-12  → Testing, legal, beta launch

Total: 12 weeks to beta-ready product
```

## Resource Assumptions
- **1 full-time developer** (solo/small team)
- **Part-time content curator** for Nepali translations + knowledge base maintenance
- **Legal review** (migration lawyer opinion) — outsourced, budget AU$500-1500
- Estimates assume focused engineering; multiply by 1.5-2x if also doing PM/design/QA

---

## Phase 0 — Foundation (Weeks 1-2)

**Goal:** Bootable dev environment, database schema, auth working

| ID | Task | Est. | Depends On | Deliverable |
|----|------|------|------------|-------------|
| P0-1 | Monorepo init: Next.js + FastAPI + Supabase skeleton | 1 day | — | `npm run dev` boots all services |
| P0-2 | Supabase project setup (local + staging), schema migration for users + visas tables | 1 day | P0-1 | DB running locally with RLS |
| P0-3 | shadcn/ui + Tailwind + i18n setup (next-intl, EN/NP) | 1 day | P0-1 | Bilingual shell rendering |
| P0-4 | Supabase Auth (email + Google OAuth), JWT middleware for Edge Functions | 1 day | P0-2 | Login/register working |
| P0-5 | FastAPI skeleton with health endpoint, Supabase admin client | 0.5 day | P0-2 | `/api/health` returns 200 |
| P0-6 | CI/CD: GitHub Actions (lint, typecheck, deploy to Vercel + Railway) | 1 day | P0-1 | Push to main → auto-deploy |
| P0-7 | Design system tokens, bilingual components (Disclaimer, LanguageToggle, ConfidenceBadge) | 1 day | P0-3 | Shared component library |
| P0-8 | Sentry setup (frontend + FastAPI), error tracking working | 0.5 day | P0-1 | Errors visible in Sentry dashboard |

**Phase 0 exit criteria:**
- Fresh clone + `npm install` + `supabase start` boots everything locally
- Auth working with JWT + RLS
- CI/CD deploys on push to main

---

## Phase 1 — F3 Document Checklist (Weeks 2-3)

| ID | Task | Est. | Depends On |
|----|------|------|------------|
| P1-1 | Checklist DB schema: `checklist_sessions` table + RLS | 1 day | P0-2 |
| P1-2 | Edge Function `/checklist`: session CRUD endpoints | 1 day | P1-1 |
| P1-3 | Visa type selection UI (card grid, 6 types with EN/NP labels) | 1 day | P0-7 |
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
| P2-3 | Edge Function `/calculator`: validate + compute endpoint | 0.5 day | P2-2 |
| P2-4 | Step-by-step wizard UI (progress bar, back/next, 12 questions) | 2 days | P0-7 |
| P2-5 | Question components: radio groups, dropdowns with points shown inline | 1 day | P2-4 |
| P2-6 | English test score mapper (IELTS/PTE/TOEFL → points auto-lookup) | 0.5 day | P2-5 |
| P2-7 | Results screen: score visualization, breakdown, SkillSelect comparison | 1 day | P2-4 |
| P2-8 | Save/load calculations (Supabase, user-scoped) | 0.5 day | P2-7 |
| P2-9 | "How to improve" suggestions (logic: identify max gainable points) | 0.5 day | P2-2 |
| P2-10 | Test suite: 10 known profiles verified for 100% correctness | 1 day | P2-2 |

**Exit:** Calculator produces correct score for all 10 test profiles. Passes CI on every push.

---

## Phase 3 — F1 Visa Tracker (Weeks 4-6)

| ID | Task | Est. | Depends On |
|----|------|------|------------|
| P3-1 | Visas DB schema: `visas` table (already in P0-2, verify RLS) | 0.5 day | P0-2 |
| P3-2 | Edge Function `/tracker`: visa CRUD endpoints | 1 day | P3-1 |
| P3-3 | Add visa wizard (3 steps: subclass → dates → conditions) | 1.5 days | P0-7 |
| P3-4 | Visa subclass UI (card grid with icons, auto-populated conditions) | 1 day | P3-3 |
| P3-5 | Dashboard: countdown timer component, condition cards, next-steps | 2 days | P3-3 |
| P3-6 | Expiry warning state (< 30 days): red-amber theme, actionable steps | 0.5 day | P3-5 |
| P3-7 | Multi-visa support: list view + individual cards | 0.5 day | P3-5 |
| P3-8 | pg_cron reminder job: daily check for 180/90/30/7 day thresholds | 1 day | P3-2 |
| P3-9 | Web Push API integration: register, send, handle click | 1.5 days | P3-8 |
| P3-10 | Service Worker: cache visa data for offline viewing | 1 day | P3-5 |
| P3-11 | Notification preferences UI (enable/disable per threshold) | 0.5 day | P3-9 |

**Exit:** User can track multiple visas, receive push reminders, view tracker offline.

---

## Phase 4 — F4 Form Helper + Scan Pipeline (Weeks 6-10)

| ID | Task | Est. | Depends On |
|----|------|------|------------|
| P4-1 | Knowledge ingestion scraper: fetch Home Affairs pages, change detection | 2 days | — |
| P4-2 | pgvector setup: embeddings generation (text-embedding-3-small), IVFFlat index | 1 day | P0-2 |
| P4-3 | Chunking pipeline: split content, generate EN+NP embeddings, store | 1.5 days | P4-1, P4-2 |
| P4-4 | RAG retrieval endpoint: query → embedding → pgvector search → top-3 chunks | 1 day | P4-3 |
| P4-5 | Claude explanation service: RAG context + form field → bilingual explanation | 1 day | P4-4 |
| P4-6 | Form selection UI (card grid with field counts, descriptions) | 1 day | P0-7 |
| P4-7 | Field explainer UI (chat-like, citation footer, feedback buttons) | 2 days | P4-5, P4-6 |
| P4-8 | Document upload API (FastAPI): validate, store in Supabase Storage | 1 day | P0-5 |
| P4-9 | Document classifier: Claude Vision → classify type, return confidence | 1 day | P4-8 |
| P4-10 | Schema extraction: per-document-type Claude prompts, structured JSON output | 2 days | P4-9 |
| P4-11 | Open extraction pass: second Claude call without schema, merge + dedup | 1 day | P4-10 |
| P4-12 | Validation layer: MRZ checksum, date plausibility, ABN/BSB format, cross-doc | 1.5 days | P4-11 |
| P4-13 | Confidence tiering: ≥85% green, 50-84% amber, <50% red, fail blank | 0.5 day | P4-12 |
| P4-14 | Extraction review UI (side-by-side: field + value + source snippet) | 2 days | P4-13 |
| P4-15 | PDF fill: pdf-lib AcroForm population from confirmed extractions | 1.5 days | P4-14 |
| P4-16 | Audit log: extraction events, user corrections, confidence scores | 1 day | P4-15 |
| P4-17 | Export: download filled PDF, share, store in Supabase | 0.5 day | P4-15 |
| P4-18 | Form field manifest: verify AcroForm names for Form 80 + Form 1221 | 1 day | — |
| P4-19 | End-to-end test: upload passport + payslips → review → download filled Form 80 | 1 day | P4-17 |

**Exit:** User can get bilingual explanation for any form field. User can upload documents → review extractions → download filled PDF.

---

## Phase 5 — Polish & Launch (Weeks 10-12)

| ID | Task | Est. | Depends On |
|----|------|------|------------|
| P5-1 | Accessibility audit: WCAG 2.1 AA, axe DevTools scan, keyboard navigation | 1 day | All features |
| P5-2 | Performance optimization: Lighthouse audit, bundle size, lazy loading | 1 day | All features |
| P5-3 | PWA installability: manifest.json, service worker, offline fallbacks | 0.5 day | P3-10 |
| P5-4 | Error state hardening: every screen tested with network offline, API down, AI timeout | 1 day | All features |
| P5-5 | Content freshness alert: flag knowledge base items > 90 days stale | 0.5 day | P4-3 |
| P5-6 | Privacy policy, T&Cs, age gate (16+), data deletion flow | 1 day | — |
| P5-7 | Legal review: migration lawyer confirms F4 does not constitute immigration assistance | External | P4-15 |
| P5-8 | Beta testing: 10-20 Nepalese migrants, feedback collection, bug fixes | 1 week elapsed | P5-1 to P5-6 |
| P5-9 | Landing page (saathi.app) with waitlist, Nepali + English | 1 day | — |
| P5-10 | Launch: Product Hunt, Nepalese Facebook groups, Reddit r/Nepal | 1 day | P5-8 |

---

## Critical Path

```
P0-1 → P0-2 → P0-4 (Auth)
              → P1-1 → P1-2 (F3 Checklist API)
              → P3-1 → P3-2 (F1 Tracker API)
              → P4-2 → P4-3 (RAG vector store)

The longest chain is P0-1 → P4-2 → P4-4 → P4-5 → P4-7 → P4-10 → P4-14 → P4-15 → P4-19 (F4 full pipeline)
F4 is the gating feature — begin Phase 4 infrastructure (P4-1 to P4-3) in parallel with Phase 1-3
```

## Parallelization Opportunities

| Can run in parallel | When |
|--------------------|------|
| F3 Checklist + F2 Calculator | Both are independent after Phase 0 |
| Knowledge base population (P1-9) | Start immediately, runs alongside coding |
| RAG infrastructure (P4-1 to P4-5) | Can start after Phase 0, no dependency on F1-F3 |
| Landing page (P5-9) | Anytime — no dependency on product features |
| Legal review (P5-7) | Start as early as possible — has external lead time |

## Cost Tracking

| Phase | External Cost (Monthly) |
|-------|------------------------|
| Phase 0-2 | $0 (Supabase free + Vercel hobby + Railway starter) |
| Phase 3-4 | ~$5/mo (Claude API usage for testing F4) |
| Phase 5 | ~$10/mo (beta users on Claude API) |
| Post-launch | ~$30-50/mo at 1K MAU, scale linearly with Claude API |

**Total POC cost: < $100 total over 12 weeks**

---

*Schedule compiled: August 8, 2026*