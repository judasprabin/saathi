# Saathi — AI Settlement Companion for Skilled Migrants to Australia

**This repo is documentation, specs, and research only — no application code.** Saathi's product is built across four other repos:

| Repo | What it is |
|---|---|
| [`lukla`](https://github.com/judasprabin/lukla) | The one frontend — every screen for every feature (F1–F6 + the Landscape Navigator) |
| [`thamel`](https://github.com/judasprabin/thamel) | Backend: F1 Tracker, F2 Calculator, F3 Checklist, F4a Explainer, plus the BFF to manaslu for F4b — personal data, resource-server auth |
| [`koshi`](https://github.com/judasprabin/koshi) | Backend: the Visa Landscape Navigator's data (occupation ceilings, EOI thresholds, state nomination status) — public data, no end-user identity |
| [`manaslu`](https://github.com/judasprabin/manaslu) | Headless document scan/extract/vault/fill agent for F4b — reached only via thamel's BFF |

Read this repo for **what to build and why**: the PRD, the architecture decisions, the screen-by-screen UX designs, the market and legal research. Read the four repos above for the actual, current implementation.

**Positioning:** mechanical form-filling is already commoditized by competitors (Instafill, FormMate80). Saathi's moat is the persistent profile vault (fill once, later forms arrive pre-filled), bilingual field-by-field explanation, and a fill-only trust posture — not raw fill speed. See [docs/research/market-and-competitive-analysis.md](./docs/research/market-and-competitive-analysis.md).

## Features

| Feature | What It Does | Built in |
|---------|-------------|----------|
| ⏱️ **Visa Tracker** (F1) | Track visa expiry, conditions, and deadlines with push reminders | thamel + lukla |
| 🧮 **Points Calculator** (F2) | Calculate skilled migration points score with SkillSelect comparison | thamel + lukla |
| 📋 **Document Checklist** (F3) | Generate personalised visa document checklists (6 visa types) | thamel + lukla |
| 📝 **Form Helper** (F4) | Bilingual field explanations (F4a) + document scan → vault → auto-fill (F4b) | thamel + manaslu + lukla |
| 🗺️ **Landscape Navigator** | Live occupation ceilings, EOI thresholds, state nomination status | koshi + lukla |
| 📰 **News & Agents** (F5/F6) | Phase 2 — not yet built | — |

## Documentation

| Doc | Description |
|-----|-------------|
| [ARCHITECTURE.md](./ARCHITECTURE.md) | Original master architecture — system design, data model, feature specs. Historical/content reference now that the code it describes lives in separate repos; still accurate for *what* each feature does. |
| [docs/PRD.md](./docs/PRD.md) | Product requirements — features, target user, success metrics |
| [docs/BUILD-SCHEDULE.md](./docs/BUILD-SCHEDULE.md) | Original phase-by-phase plan — superseded by each repo's own implementation plans, kept as the content/task reference they were built from |
| [docs/architecture/ui-ux-flows.md](./docs/architecture/ui-ux-flows.md) | Screen-by-screen UX design for all features — the design source `lukla` builds from |
| [docs/architecture/f4-manaslu-integration.md](./docs/architecture/f4-manaslu-integration.md) | The F4b contract between thamel and manaslu |
| [docs/superpowers/specs/2026-08-13-visa-landscape-navigator-design.md](./docs/superpowers/specs/2026-08-13-visa-landscape-navigator-design.md) | The Landscape Navigator's original design — see its header note for the koshi/lukla split |
| [docs/research/market-and-competitive-analysis.md](./docs/research/market-and-competitive-analysis.md) | Consolidated market + competitive analysis (TAM/SOM + the July 2026 competitive re-assessment) |
| [docs/legal/legal-memo.md](./docs/legal/legal-memo.md) | Legal analysis of visa form auto-fill under Australian law |
| [diagrams/saathi-screen-designs.html](./diagrams/saathi-screen-designs.html) | Screen-by-screen design board — 39 screens with options, states, worked examples |
| [diagrams/saathi-landscape-navigator-mockup.html](./diagrams/saathi-landscape-navigator-mockup.html) | Working interactive prototype `lukla` ports directly into React |

**Brand note:** the screen designs above use Nepal-flag red/blue, from when this product was Nepali-community-specific. Positioning has since moved to nationality-agnostic skilled migration; `lukla`'s spec retires those colors in favor of the validated neutral dataviz palette. Treat the designs' *content and layout* as current, their *color values* as superseded.

## Regulatory

Saathi provides INFORMATION only. It does NOT:
- Assess eligibility for any visa
- Recommend visa pathways
- Lodge or submit applications
- Prepare forms on a person's behalf

⚠️ All features carry a disclaimer: "Consult a registered migration agent (verify at mara.gov.au using their MARN)."

## Status

**Pre-development across all four implementation repos.** Each has its own design spec and (where written) implementation plan under `docs/superpowers/`.
