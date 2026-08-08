# Saathi — AI Settlement Companion for Nepalese Diaspora in Australia

Bilingual (English/Nepali) PWA that helps Nepalese migrants in Australia navigate the immigration system. Built across **two repos**: this one (frontend + F1/F2/F3 + F4 integration), and [`manaslu`](../manaslu) (headless scan/extract/form-fill agent that powers F4b). See [ARCHITECTURE.md §1](./ARCHITECTURE.md#1-system-overview) for the boundary.

**Positioning:** mechanical form-filling is already commoditized by competitors (Instafill, FormMate80). Saathi's moat is the persistent profile vault (fill once, later forms arrive pre-filled), bilingual field-by-field explanation, and a fill-only trust posture — not raw fill speed. See [docs/research/market-and-competitive-analysis.md](./docs/research/market-and-competitive-analysis.md).

## Four Core Features

| Feature | What It Does | Built in |
|---------|-------------|----------|
| ⏱️ **Visa Tracker** | Track visa expiry, conditions, and deadlines with push reminders | saathi |
| 🧮 **Points Calculator** | Calculate skilled migration points score with SkillSelect comparison | saathi |
| 📋 **Document Checklist** | Generate personalised visa document checklists (6 visa types) | saathi |
| 📝 **Form Helper** | Bilingual field explanations (saathi) + document scan → vault → auto-fill (manaslu) | saathi + manaslu |

## Quick Start

```bash
# Clone both repos — saathi needs manaslu running for F4b
git clone https://github.com/judasprabin/saathi.git
git clone https://github.com/judasprabin/manaslu.git

cd saathi

# Frontend
cd web && npm install && npm run dev

# API
cd api && pip install -r requirements.txt && uvicorn app.main:app --reload

# Cloud SQL (local dev via Cloud SQL Auth Proxy — see docs/architecture/f4-manaslu-integration.md)
```

## Documentation

| Doc | Description |
|-----|-------------|
| [ARCHITECTURE.md](./ARCHITECTURE.md) | Master architecture — system design, data model, all features, saathi/manaslu boundary |
| [docs/PRD.md](./docs/PRD.md) | Product requirements — 4 features, target user, success metrics |
| [docs/BUILD-SCHEDULE.md](./docs/BUILD-SCHEDULE.md) | Phase-by-phase build plan with tasks, estimates, dependencies |
| [docs/architecture/ui-ux-flows.md](./docs/architecture/ui-ux-flows.md) | Screen-by-screen UX design for all features |
| [docs/architecture/f4-manaslu-integration.md](./docs/architecture/f4-manaslu-integration.md) | Saathi's side of the F4b contract (the pipeline itself is documented in `manaslu/docs/architecture/`) |
| [docs/research/market-and-competitive-analysis.md](./docs/research/market-and-competitive-analysis.md) | Consolidated market + competitive analysis (TAM/SOM + the July 2026 competitive re-assessment) |
| [docs/legal/legal-memo.md](./docs/legal/legal-memo.md) | Legal analysis of visa form auto-fill under Australian law |
| [diagrams/saathi-ui-mockup.html](./diagrams/saathi-ui-mockup.html) | Interactive HTML mockup, all 4 feature screens |
| [diagrams/saathi-screen-designs.html](./diagrams/saathi-screen-designs.html) | Screen-by-screen design board — all 27 screens with options, states, and worked examples per page |

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Next.js 14 (App Router) + PWA + shadcn/ui, on Cloud Run |
| i18n | next-intl (English + Nepali) |
| API | FastAPI (Python) on Cloud Run — F1/F2/F3 CRUD + F4a RAG |
| Primary AI | Claude Sonnet 5 (Anthropic) |
| Embeddings | Voyage AI `voyage-multilingual-2` (Nepali-quality retrieval) |
| Database | Cloud SQL for PostgreSQL + pgvector |
| Auth | GCP Identity Platform — same token manaslu verifies as a resource server |
| Storage | GCS (user-scoped, AU region) |
| Notifications | Firebase Cloud Messaging |
| Monitoring | Cloud Logging + Cloud Monitoring + Error Reporting |
| IaC | Terraform in `karki-labs-infra` |

manaslu's stack (separate repo, same GCP project family): Cloud Run agent loop (Claude Opus/Sonnet/Haiku tiered), Cloud SQL, GCS, Identity Platform — see `manaslu/README.md`.

## Architecture Decisions

Eight key decisions that shaped Saathi — full rationale in [ARCHITECTURE.md §14](./ARCHITECTURE.md#14-architecture-decision-records):
1. **F1-F3 as one lightweight Cloud Run API; F4b delegated to manaslu** — not rebuilt here
2. **Rules engine** — Points calculator is deterministic JSON, not LLM
3. **Decision trees** — Checklist generation is deterministic logic, not LLM
4. **pgvector** — RAG runs on saathi's own Cloud SQL, scoped to F4a/F3 only
5. **Firebase Cloud Messaging** — one push SDK, no third-party vendor
6. **Claude Sonnet** — Best Nepali (Devanagari) support of any commercial model
7. **Voyage AI embeddings, not OpenAI** — OpenAI's embeddings measurably degrade on Nepali
8. **F4b is manaslu's responsibility** — already built around the vault/bilingual-explain moat; rebuilding it here would duplicate solved, harder engineering

## Regulatory

Saathi provides INFORMATION only. It does NOT:
- Assess eligibility for any visa
- Recommend visa pathways
- Lodge or submit applications
- Prepare forms on a person's behalf

⚠️ All features carry a disclaimer: "Consult a registered migration agent (verify at mara.gov.au using their MARN)."

## Status

**Pre-development** — Architecture approved. See [BUILD-SCHEDULE.md](./docs/BUILD-SCHEDULE.md) for implementation timeline (~11 weeks to beta, assuming manaslu's M3 lands within that window).

---

*Built for the Nepalese community in Australia. 🇳🇵🇦🇺*