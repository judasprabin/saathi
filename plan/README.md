# Saathi — Executive Summary

**AI Settlement & Immigration Companion for the Nepalese Diaspora in Australia**

## What It Is

Saathi is a focused, evidence-backed utility platform that helps Nepalese migrants in Australia navigate the immigration system. It does NOT give migration advice or lodge applications — it provides information, tools, and connections sourced from 19 verified Australian government domains.

## Six Features

| # | Feature | What It Does | Status |
|---|---------|-------------|--------|
| F1 | **Visa Tracker** | Track expiry dates, visa conditions, and deadline reminders (180/90/30/7 days) | MVP |
| F2 | **Points Calculator** | Deterministic skilled migration points engine with live SkillSelect comparison | MVP |
| F3 | **Document Checklist** | Personalised checklist generator for 6 visa types with per-item guidance | MVP |
| F4a | **Form Explainer** | RAG-powered field-by-field bilingual explanations of immigration forms | MVP |
| F4b | **Auto-Fill (manaslu)** | Document scan → profile vault → AcroForm PDF fill for Form 80/1221 | MVP |
| F5 | **News & Events** | Curated immigration news from 19 domains + MARN-verified seminars | Phase 2 |
| F6 | **Agent Connect** | MARA-registered agent directory with enquiry and consent-based share | Phase 2 |

## Knowledge Pipeline

All content is powered by the **AU Visa Source Registry** — an autonomous crawler that monitors 19 Australian government domains daily (300 pages/run, 17 categories, change detection). Every explanation in Saathi carries a source URL and "last verified" timestamp.

## Architecture (3 repos + 1 infra repo)

```
┌───────────────────────────────────────────────────────────┐
│                     GCP Project: saathi-prod               │
│                                                            │
│  GKE Cluster (asia-southeast1 / australia-southeast1)      │
│  ┌──────────────────────────────────────────────────┐     │
│  │  saathi-web (Next.js PWA)    │  saathi-api (FastAPI) │  │
│  │  F1 Tracker, F2 Calculator,  │  F1-F3 CRUD + F4a RAG│  │
│  │  F3 Checklist, F4a UI,       │  + BFF for manaslu   │  │
│  │  F4b Review UI               │                       │  │
│  ├──────────────────────────────┼───────────────────────┤  │
│  │  manaslu (FastAPI agent)     │  Cloud SQL (PostgreSQL│  │
│  │  classify/extract/fill/vault │  + pgvector for RAG)  │  │
│  └──────────────────────────────────────────────────┘     │
│                                                            │
│  External tools: GCS (storage), Identity Platform (auth),  │
│  Cloud Monitoring, Cloud Logging, Secret Manager, IAM      │
└───────────────────────────────────────────────────────────┘

Crawler: judasprabin/research → Notion → feeds Saathi knowledge base
IaC:      karki-labs-infra → Terraform for GCP resources
```

## Target Metrics

| Metric | Target |
|--------|--------|
| Beta users | 500 (TestFlight + Play Store) |
| Week-4 retention | ≥ 25% |
| Receipt scans/week | ≥ 2 per user |
| Form completions | ≥ 1 per user/month |
| Points calculations | ≥ 1 per user |
| Checklist completions | ≥ 1 per user |
| Content freshness | < 24 hours (19 domains verified daily) |

## Timeline

| Phase | Weeks | Deliverable |
|-------|-------|-------------|
| Phase 0 | 1-2 | GKE cluster, Cloud SQL, CI/CD, auth |
| Phase 1 | 2-4 | F3 Checklist (first shippable feature) |
| Phase 2 | 4-6 | F2 Calculator + F1 Tracker |
| Phase 3 | 6-10 | F4a Explainer (RAG) + F4b integration (manaslu) |
| Phase 4 | 10-12 | Polish, legal review, beta launch |

**Total: 12 weeks to beta | P0: 1 developer + part-time content curator**

## Regulatory

Saathi provides INFORMATION only. Does NOT: assess eligibility, recommend pathways, lodge applications, or prepare forms on anyone's behalf. All features carry a MARA disclaimer. Legal opinion (Migration Act 1958 s.276) pending — gates F4b public launch, not development.
