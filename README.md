# Saathi — AI Settlement Companion for Nepalese Diaspora in Australia

Bilingual (English/Nepali) PWA that helps Nepalese migrants in Australia navigate the immigration system.

## Four Core Features

| Feature | What It Does |
|---------|-------------|
| ⏱️ **Visa Tracker** | Track visa expiry, conditions, and deadlines with push reminders |
| 🧮 **Points Calculator** | Calculate skilled migration points score with SkillSelect comparison |
| 📋 **Document Checklist** | Generate personalised visa document checklists (6 visa types) |
| 📝 **Form Helper** | Bilingual form field explanations + document scan → auto-fill pipeline |

## Quick Start

```bash
# Clone
git clone https://github.com/judasprabin/saathi.git
cd saathi

# Frontend
cd web && npm install && npm run dev

# API
cd api && pip install -r requirements.txt && uvicorn app.main:app --reload

# Supabase (requires Supabase CLI)
supabase start
```

## Documentation

| Doc | Description |
|-----|-------------|
| [ARCHITECTURE.md](./ARCHITECTURE.md) | Master architecture (69KB) — system design, data model, all features |
| [docs/PRD.md](./docs/PRD.md) | Product requirements — 4 features, target user, success metrics |
| [docs/BUILD-SCHEDULE.md](./docs/BUILD-SCHEDULE.md) | Phase-by-phase build plan with tasks, estimates, dependencies |
| [docs/architecture/ui-ux-flows.md](./docs/architecture/ui-ux-flows.md) | Screen-by-screen UX design for all features |
| [docs/architecture/scan-pipeline.md](./docs/architecture/scan-pipeline.md) | Document scan → classification → extraction → fill pipeline |
| [docs/research/market-research.md](./docs/research/market-research.md) | Market analysis: 20+ products, Nepal-AU corridor data |
| [docs/legal/legal-memo.md](./docs/legal/legal-memo.md) | Legal analysis of visa form auto-fill under Australian law |
| [diagrams/system-architecture.html](./diagrams/system-architecture.html) | Interactive SVG architecture diagram |

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Next.js 14 (App Router) + PWA + shadcn/ui |
| i18n | next-intl (English + Nepali) |
| Light API | Supabase Edge Functions (Deno/TypeScript) |
| AI API | FastAPI (Python) + Railway |
| Primary AI | Claude Sonnet 4.5 (Anthropic) |
| Database | Supabase PostgreSQL + pgvector |
| Auth | Supabase Auth (email + Google OAuth) |
| Storage | Supabase Storage (user-scoped) |
| Monitoring | Sentry |

## Architecture Decisions

Six key decisions that shaped Saathi:
1. **Hybrid backend** — Edge Functions for CRUD (F1-F3), FastAPI for AI pipelines (F4)
2. **Rules engine** — Points calculator is deterministic JSON, not LLM (correctness > flexibility)
3. **Decision trees** — Checklist generation is deterministic logic, not LLM
4. **pgvector** — RAG runs in Supabase, no external vector DB needed
5. **Web Push API** — Native browser notifications, no third-party service
6. **Claude Sonnet** — Best Nepali (Devanagari) support of any commercial model

## Regulatory

Saathi provides INFORMATION only. It does NOT:
- Assess eligibility for any visa
- Recommend visa pathways
- Lodge or submit applications
- Prepare forms on a person's behalf

⚠️ All features carry a disclaimer: "Consult a registered migration agent (verify at mara.gov.au using their MARN)."

## Status

**Pre-development** — Architecture approved. See [BUILD-SCHEDULE.md](./docs/BUILD-SCHEDULE.md) for implementation timeline (12 weeks to beta).

---

*Built for the Nepalese community in Australia. 🇳🇵🇦🇺*