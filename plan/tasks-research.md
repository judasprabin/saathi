# Research Repo (Knowledge Pipeline) — Prioritized Task List

**Repo:** `judasprabin/research` | **Stack:** Python + Notion API + cron  
**Scope:** AU Visa Source Registry crawler + SkillSelect monitor + Notion knowledge base

---

## Phase 0 — Crawler Stabilization (Current) | Priority: P0

| # | Task | Est. | Status |
|---|------|------|--------|
| R0.1 | CDN unblock: browser UA rotation, Akamai detection, retry logic | Done | ✅ Complete |
| R0.2 | URL path corrections: MARA, SkillSelect, Jobs & Skills, forms verified | Done | ✅ Complete |
| R0.3 | Classifier overhaul: 17 categories, 35+ URL patterns, reduced "other" from 64% to 40% | Done | ✅ Complete |
| R0.4 | Config expansion: 19 domains, 300 pages/run, 15 per domain | Done | ✅ Complete |
| R0.5 | Cron setup: daily 6am AEST + SkillSelect every 6h | Done | ✅ Complete |

---

## Phase 1 — Classifier Improvement (Week 1) | Priority: P1

| # | Task | Est. | Depends |
|---|------|------|---------|
| R1.1 | Analyze current 34 "other" records → identify missing patterns | 1d | R0.4 |
| R1.2 | Add patterns for common Home Affairs sub-pages | 0.5d | R1.1 |
| R1.3 | Fix "state nomination" and "invitation rounds" categories → pages currently routing to wrong bucket | 0.5d | R1.1 |
| R1.4 | Run classifier on existing 86 records → verify "other" drops below 15% | 0.5d | R1.2 |
| R1.5 | Add visa subclass detection to "settlement info" pages → tag which visa they relate to | 0.5d | R1.3 |

**Exit:** "other" category < 15% of total records. State nomination and invitation rounds categories populated.

---

## Phase 2 — Content Quality (Week 1-2) | Priority: P1

| # | Task | Est. | Depends |
|---|------|------|---------|
| R2.1 | Summary quality check: review auto-generated summaries for 20 random records | 0.5d | — |
| R2.2 | Improve summary extraction: use og:description, meta description, or first meaningful paragraph | 1d | R2.1 |
| R2.3 | Date extraction improvement: parse more date formats from meta tags and visible text | 0.5d | R2.2 |
| R2.4 | Dead link detection: periodic re-check of all existing URLs (--check-only mode) | 0.5d | — |
| R2.5 | Add redirect chain tracking: if URL redirects, record final URL + intermediate hops | 0.5d | R2.4 |

**Exit:** 90% of records have useful summaries (not just page title). Dead links detected within 24h.

---

## Phase 3 — Saathi Knowledge Feed (Week 2-3) | Priority: P0

| # | Task | Est. | Depends |
|---|------|------|---------|
| R3.1 | Knowledge export API: endpoint to fetch records by category, freshness, or visa subclass | 1d | — |
| R3.2 | JSON dump: daily export of all active records → GCS bucket for saathi-knowledge CronJob to consume | 1d | R3.1 |
| R3.3 | Change feed: only export records that changed since last run (reduce saathi-knowledge workload) | 0.5d | R3.2 |
| R3.4 | Saathi knowledge CronJob integration test: crawler writes → CronJob reads → pgvector updated | 1d | R3.3 |

**Exit:** Saathi's knowledge base automatically updates within 24h of a government page changing.

---

## Phase 4 — Monitoring & Alerts (Week 3) | Priority: P2

| # | Task | Est. | Depends |
|---|------|------|---------|
| R4.1 | Crawler health dashboard: success rate, new/updated/skipped/dead/failed per run | 1d | — |
| R4.2 | Staleness alert: any record not checked in > 7 days → Discord notification | 0.5d | R4.1 |
| R4.3 | CDN block alert: if > 3 domains blocked in one run → Discord notification | 0.5d | R4.1 |
| R4.4 | Content change alert: if key pages (visa listing, processing times, legislation) change → immediate Discord | 0.5d | R3.3 |
| R4.5 | Run log persistence: store run summaries in GCS for long-term trend analysis | 0.5d | R4.1 |

---

## Always-On Tasks

| Task | Cadence | Owner |
|------|---------|-------|
| Review "other" category records | Weekly | Engineering |
| Add new government domains as they launch | As needed | Engineering |
| Update classifier patterns for site restructures | When sites change | Engineering |
| Verify Notion API rate limits not breached | Monthly | Engineering |
