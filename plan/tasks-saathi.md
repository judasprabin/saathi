# Saathi Repo — Prioritized Task List

**Repo:** `judasprabin/saathi` | **Stack:** Next.js 14 + FastAPI + GKE  
**Scope:** Frontend (F1-F6 UI) + API (F1-F3 CRUD + F4a RAG + manaslu BFF)

---

## Phase 0 — Foundation (Week 1-2) | Priority: P0

| # | Task | Est. | Depends |
|---|------|------|---------|
| 0.1 | Monorepo scaffold: `/web` (Next.js), `/api` (FastAPI), `/k8s` (manifests), `/db` (migrations) | 1d | — |
| 0.2 | Dockerfiles: web (multi-stage Next.js), api (Python slim), knowledge (batch job) | 0.5d | 0.1 |
| 0.3 | Cloud SQL schema: users, visas, checklist_sessions, points_results, knowledge_base (pgvector), audit_log | 1d | 0.1 |
| 0.4 | GCP Identity Platform setup: email/password + Google OAuth, Firebase Admin SDK for FastAPI | 1d | 0.3 |
| 0.5 | K8s manifests: Deployments (web, api, knowledge), Services, ConfigMap, Gateway HTTPRoute | 1d | 0.2 |
| 0.6 | Cloud Build pipeline: `cloudbuild.yaml` — build images → push Artifact Registry → deploy to GKE | 1d | 0.5 |
| 0.7 | DB migrations tooling: Alembic + Cloud SQL Proxy sidecar in Cloud Build | 0.5d | 0.3 |
| 0.8 | Secret Manager integration: External Secrets Operator → mount Anthropic, Voyage, DB keys | 0.5d | 0.5 |
| 0.9 | Shared UI: shadcn/ui setup, design tokens, Disclaimer component, SourceCitation, ConfidenceBadge | 1d | 0.1 |
| 0.10 | Routing + navigation shell: sidebar, tab bar, page layout, auth guard | 1d | 0.9 |
| 0.11 | FastAPI skeleton: health check, CORS, error middleware, Supabase-to-GCP migration layer | 1d | 0.1 |
| 0.12 | Observability: OpenTelemetry SDK for traces, structured JSON logging, Cloud Monitoring metrics | 0.5d | 0.11 |

**Phase 0 exit:** `kubectl apply -f k8s/` boots everything. Auth working. `api.saathi.app/health` returns 200.

---

## Phase 1 — F3 Document Checklist (Week 2-3) | Priority: P0

| # | Task | Est. | Depends |
|---|------|------|---------|
| 1.1 | DB: checklist_sessions table with RLS (owner_uid) | 0.5d | 0.3 |
| 1.2 | API: `/checklist` CRUD endpoints (FastAPI router) | 1d | 1.1 |
| 1.3 | API: decision tree engine — load JSON rules per visa type | 1d | 1.2 |
| 1.4 | UI: visa type selector (card grid, 6 types) | 1d | 0.10 |
| 1.5 | UI: branching questionnaire (step-by-step, progress bar) | 1d | 1.4 |
| 1.6 | UI: checklist renderer (categorized accordion, status badges, progress) | 1d | 1.5 |
| 1.7 | UI: checklist item detail (expand/collapse: what, how to get, common mistakes) | 0.5d | 1.6 |
| 1.8 | UI: print/export PDF (print CSS, basic PDF generation) | 0.5d | 1.7 |
| 1.9 | Content: populate checklist rules JSON for all 6 visa types (curated from crawler data) | 2d | 1.3 |
| 1.10 | Test: verify all branching paths for 6 visa types | 1d | 1.9 |

**Phase 1 exit:** User selects visa → answers questions → gets correct checklist → prints it. All 6 visa types verified.

---

## Phase 2 — F2 Points Calculator + F1 Visa Tracker (Week 4-6) | Priority: P0

| # | Task | Est. | Depends |
|---|------|------|---------|
| 2.1 | DB: points_results table + visas table (verify existing from Phase 0) | 0.5d | 0.3 |
| 2.2 | API: `/calculator` — deterministic points engine (Python rules class) | 1d | 2.1 |
| 2.3 | API: `/tracker` — visa CRUD endpoints | 1d | 2.1 |
| 2.4 | API: `/tracker/reminders` — pg_cron schedule + FCM push dispatch | 1d | 2.3 |
| 2.5 | UI: Points calculator wizard (12 steps, progress, running total) | 2d | 0.10 |
| 2.6 | UI: Results screen (score visualization, breakdown, SkillSelect comparison) | 1d | 2.5 |
| 2.7 | UI: "How to improve" suggestions (identify max gainable points) | 0.5d | 2.6 |
| 2.8 | UI: Save/share results | 0.5d | 2.6 |
| 2.9 | UI: Visa tracker — add visa wizard (subclass → dates → conditions) | 1.5d | 0.10 |
| 2.10 | UI: Visa dashboard (countdown ring, conditions, next steps) | 1.5d | 2.9 |
| 2.11 | UI: Expiry warning state (< 30 days) + multi-visa support | 1d | 2.10 |
| 2.12 | UI: Push notification preferences + FCM web integration | 1d | 2.4 |
| 2.13 | Test: Points calculator — 10 known profiles verified correct | 1d | 2.2 |

**Phase 2 exit:** Calculator correct on all 10 profiles. Tracker shows reminders. Push notifications fire.

---

## Phase 3 — F4a Form Explainer + F4b manaslu Integration (Week 6-10) | Priority: P0

| # | Task | Est. | Depends |
|---|------|------|---------|
| 3.1 | CronJob: `saathi-knowledge` — pull from Notion registry → chunk → embed (Voyage AI) → upsert pgvector | 2d | 0.11 |
| 3.2 | API: `/explain/field` — query pgvector → retrieve top-3 chunks → Claude generates explanation | 1.5d | 3.1 |
| 3.3 | API: manaslu BFF — create session, forward events (SSE proxy), handle confirmations | 2d | 0.11 |
| 3.4 | UI: Form selector (Form 80, 1221, 485 sections, EOI) | 0.5d | 0.10 |
| 3.5 | UI: Field explainer (field-by-field view, source citation, feedback buttons) | 2d | 3.2 |
| 3.6 | UI: manaslu review screen — side-by-side extraction + source crop (renders manaslu SSE events) | 2d | 3.3 |
| 3.7 | UI: Confidence tier display (HIGH/MED/LOW/FAIL badges, edit/confirm/skip actions) | 1d | 3.6 |
| 3.8 | UI: PDF download + audit log view | 1d | 3.6 |
| 3.9 | Integration test: upload documents → classify → extract → review → download filled PDF | 2d | 3.8 |

**Phase 3 exit:** User gets field explanation with citation. User uploads docs → reviews → downloads filled Form 80.

---

## Phase 4 — Polish & Beta Launch (Week 10-12) | Priority: P0/P1

| # | Task | Est. | Depends |
|---|------|------|---------|
| 4.1 | Accessibility: WCAG 2.1 AA audit (keyboard, screen reader, contrast) | 1d | All |
| 4.2 | Performance: Lighthouse ≥ 90, bundle optimization, image lazy loading | 1d | All |
| 4.3 | PWA: manifest.json, service worker, offline fallback for F1/F2 | 1d | All |
| 4.4 | Error hardening: every screen tested with network offline, API down, AI timeout | 1d | All |
| 4.5 | Legal review: privacy policy, T&Cs, MARA disclaimer audit | 1d | All |
| 4.6 | Landing page: `saathi.app` — value prop, feature preview, waitlist | 1d | — |
| 4.7 | Beta test: 20 users, feedback collection, bug fixes | 1w elapsed | 4.1-4.5 |
| 4.8 | Launch: TestFlight + Play Store Beta + Product Hunt | 1d | 4.7 |

## Phase 5 — F5 News + F6 Agent Connect (Post-Beta) | Priority: P2

| # | Task | Est. | Depends |
|---|------|------|---------|
| 5.1 | F5: News feed UI — articles from crawler data, categorized, freshness badges | 2d | 3.1 |
| 5.2 | F5: Events calendar — MARN-verified seminars, state info sessions | 1.5d | 5.1 |
| 5.3 | F6: Agent directory UI — search, filter by location/visa type, MARA verification badge | 2d | — |
| 5.4 | F6: Enquiry flow — itemised share-consent, revocation, rate limiting | 2d | 5.3 |
| 5.5 | F5/F6: Integration tests + beta feedback | 2d | 5.1-5.4 |

## Always-On / Cross-Cutting

| Task | Cadence | Owner |
|------|---------|-------|
| Dependency updates (npm, pip) | Weekly | Engineering |
| Security scan (container images, deps) | Weekly (CI) | Engineering |
| Cost review (GCP billing dashboard) | Monthly | Engineering |
| Content freshness alert (>90d stale) | Daily (automated) | Knowledge CronJob |
| Golden test re-run | Every PR touching prompts/pipeline | CI |
