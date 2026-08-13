# Visa Landscape Navigator — Design Spec

**Status:** Superseded by a repo split — see note below. Design and mockup remain the source of truth for *what* this feature is; implementation now happens in two separate repos.
**Date:** 2026-08-13, technical resolution 2026-08-14, split 2026-08-14
**Author:** Prabin Karki, via brainstorming session with Claude
**Visual reference:** [`diagrams/saathi-landscape-navigator-mockup.html`](../../../diagrams/saathi-landscape-navigator-mockup.html) — the final approved interactive prototype. Open it in a browser; it's the primary spec artifact, this document explains the decisions behind it.

> **Split into two repos (2026-08-14):** this feature's backend and frontend are now built as separate microservices rather than inside saathi — **[`koshi`](https://github.com/judasprabin/koshi)** (backend: extraction, schema, API — no end-user identity, public data only) and **[`lukla`](https://github.com/judasprabin/lukla)** (frontend: the UI described in this spec, served at `saathi.app/landscape` via a shared load balancer). The `docs/superpowers/plans/2026-08-14-visa-landscape-navigator-occupation-slice.md` implementation plan in *this* repo is superseded — its backend tasks (1–5) became koshi's plan, its frontend tasks (6–8) became lukla's plan. This document and the mockup remain accurate for *what* the feature is and *why* each decision was made; koshi's and lukla's own specs cover what changed because of the split (auth model, data sourcing detail, routing).

---

## 1. Why this exists

Three independent AI research passes (`grok.md`, `openai.md`, `perplexity.md` — competitive research the user collected) plus three competitor links (Migration Manager, ImmiIQ, VisaDocket — all B2B migration-agent software, not consumer competitors) converged on one finding: **F1 (tracker), F2 (calculator), and F3 (checklist) are more commoditized than the existing plan accounted for.** `openai.md`'s competitor table shows Home Affairs' own free myVEVO app, Visa Tracker Pro, Hazuu's apps, VisaFlow AI, and RouteToVisa already covering tracker/calculator/checklist territory for free. This is separate from and additional to the July 2026 competitive analysis (`research/dispora-nepal/competitive-analysis.md`), which was specifically about form-filling commoditization (Instafill, FormMate80).

The question this spec answers: **what does Saathi build that isn't already free elsewhere, and that a solo builder can actually ship?**

## 2. Strategic decisions (made before any visual design)

| Decision | Choice | Reasoning |
|---|---|---|
| Audience | **Nationality-agnostic**, not Nepali-specific | User's explicit call, after weighing that this trades away the free community-distribution channel for a different kind of defensible narrowness: deep on one problem (evidence/pathway clarity for skilled migration) rather than deep on one community. Destination country is still Australia only — visa subclass numbers (189/190/491 etc.) and all legal guardrails (Migration Act 1958, MARA) are AU-specific throughout. |
| Buyer | **B2C first.** B2B2C (licensing to migration agents, per `perplexity.md`'s suggestion) explicitly deferred to a later phase, contingent on B2C traction | Matches the project's existing validate-before-expand discipline (same shape as the kill-criteria pattern in the July competitive analysis). |
| Scope relative to existing plan | **Augment, not replace.** F1/F2/F3 stay as lightweight, low-investment utility features. This feature becomes the new flagship/differentiator sitting alongside them | Avoids re-litigating already-built F1-F6 work; this is additive. |
| Visa scope | **Skilled, points-tested pathways first** — 189, 190, 491, with 482/186 shown for context/comparison | Matches `perplexity.md`'s specific MVP recommendation and the user's own phrasing ("skilled migration"). Partner/student/family visa data is out of scope for this feature. |

## 3. The core concept

**Not** a evidence-readiness/claim-tracking tool (the first direction explored — see §4). **Not** a fear-based "risk check" (the second direction explored — see §4). What shipped: a **live, data-driven landscape navigator** — real official numbers (occupation ceilings, EOI invitation thresholds, state nomination status, processing times) presented as an explorable, personalized dashboard, not a form to fill in or a prediction about any individual's chances.

**Why this is buildable, not just a good idea:** it sits on infrastructure that already exists.
- The **AU Visa Source Registry crawler** (`research/au-visa-sources/`) already scrapes SkillSelect, Home Affairs, and state nomination pages daily — this feature's backing data is largely what that crawler already collects, not a new pipeline.
- The **pathway-map mockup** built earlier in this same design conversation (journey-style visa pathway diagram) is directly reusable as this feature's underlying navigation structure.
- **F2's points-criteria engine** is reused directly for the points-breakdown and improvement-calculator content — not rebuilt.

## 4. Design directions considered and rejected (context so these aren't re-proposed)

1. **Claim-to-evidence matrix** (rows = visa claims, columns = supporting evidence/confidence/gaps, from `perplexity.md`/`openai.md`'s "evidence graph"/"consistency checker" ideas). Rejected: requires the user to proactively build and maintain a matrix — real homework with no immediate payoff. "I don't think general users will come and fill all the details to track this" (user).
2. **Passive/derived version of the same idea** (readiness computed automatically from documents scanned via manaslu + F2 answers, surfaced passively). Better, but user asked for something more immediate and visceral than an analytical readiness score.
3. **"60-Second Refusal Risk Check"** (quick questions → "N things likely to cause delay/refusal," personalized). **Rejected on legal grounds** — this is a personalized prediction about a specific case's outcome, which is squarely the "immigration assistance" territory s.276 of the Migration Act reserves for MARA-registered agents. Same line `AustraliaVisa.ai` already sits uncomfortably close to per the July competitive analysis. The safe reframe (fact-finding — "3 things found in your documents," e.g. name-spelling mismatches — not risk-prediction) was validated as legally sound but was set aside when a stronger direction (see below) emerged.
4. **"Would you have been invited?" comparative verdict.** Rejected as "not impressive" — assessed as just F2 (points calculator) reframed, not a genuinely new differentiator.
5. **Scam & Agent Check** (paste a MARN/message, get instant verified/red-flag verdict). Not rejected, just deferred — good as a separate, cheap, shareable utility, not strong enough alone to be the flagship.

The winning direction emerged when the user described their own original vision directly (not mediated through the AI research docs): *"this product should be able to provide any immigrants... navigate a prospective path easily, understanding the current state of invitation, criteria and seats available in depth."* That is what this spec describes.

## 5. Information architecture

### 5.1 Personalization spine, not five disconnected reports

Early iterations built five separate report "levels" (Occupation / State / Country / Visa Type / Trends), each showing a different, disconnected example (Software Engineer for Occupation, NSW for State, 189 for Visa Type — no relationship between them). User feedback: *"the way we present this... should be very clear by providing opportunity to see what's happening overall"* for **one person's specific situation**, not five unrelated demos.

**Resolved structure:**
1. **Hero** — national KPIs, zero input required, no account needed to browse.
2. **"Explore your own path"** — a lightweight selector (occupation, state, visa types under consideration). Not a form; pill-based, one tap per choice. Changing a selection updates everything tagged **"Your..."** below it.
3. **"Your..." sections** — Your Occupation, Your State, Your Options (visa comparison), Closing Your Gap (points levers) — all scoped to the current selection, with a persistent breadcrumb stating what's being shown.
4. **"All..." sections nested inside the relevant "Your" section** — All States (map), All Visa Types (full comparison table), All Occupations (national ranking) — so the aggregate/comparative view sits next to the personalized one rather than being a separate, disconnected page. This was a second round of feedback: the first personalization pass over-corrected and dropped the aggregate views entirely.
5. **"Zooming out: National"** — country-wide data (application funnel, 5-year trend, EOI pool growth) that isn't about the user's specific choices at all — explicitly tagged as such.
6. **"Reference"** — standing content (English test score bands, skills-assessment body turnaround) that's neither personal nor national-trend, just lookup material.

Every section carries a small tag (`Your...` / `All...` / `Zooming out` / `Reference`) so it's visually unambiguous which of the four categories a given panel belongs to.

### 5.2 Content inventory (what's in the approved mockup)

- **Your Occupation:** ceiling-usage gauge (issued/cap, pace vs. last year), points distribution among recent invitees with median marker, occupation-list-status badges (MLTSSL/STSOL/Regional)
- **Your State:** full nomination criteria (state's own point minimum, residency commitment, fee, decision timeline, documents, approval pattern), what specifically changed on the list recently (occupations added/removed by name), state list size trend
- **All States:** clickable/hoverable map, every state's status + fee + decision time for the current occupation
- **Your Options:** full 189-vs-491 (or whichever two are selected) comparison — points, time to grant, **time to permanent residency** (see §6.3), age limit, work rights, family inclusion, residence requirement, cost
- **All Visa Types:** 189/190/491/482/186 side by side — permanence, sponsor requirement, threshold, time, occupation-list requirement, pathway onward (491→191, 482→186), cost
- **Closing Your Gap:** full current points breakdown (all ~12 criteria), then ranked improvement levers (points gained → mechanical time cost, e.g. "+10, English Competent→Proficient, ~2-3 months")
- **National — All Occupations:** ceiling usage ranked across occupations, current selection highlighted against the rest (emphasis pattern), 3-round momentum indicator
- **National — funnel:** EOI submitted → invited → granted, broken down **by pathway** (491 shows a lower invited→granted rate — flagged as a real, sourced difference, not manufactured drama)
- **National — 5-year trend:** all three visa types' threshold history, with a policy-change annotation
- **Reference:** English test score bands (IELTS/PTE/TOEFL/OET) and skills-assessment bodies (ACS/Engineers Australia/VETASSESS/TRA), with cost and validity/turnaround

## 6. Cross-cutting design principles

### 6.1 "What this means" insight lines
Every major chart carries one short, stated takeaway rather than leaving the reader to infer it from raw numbers (e.g., "Median invitee had 73 points — beating that, not the 85 headline figure, is what actually matters most rounds"). These must be **computed from the underlying data, deterministically** — templated, not LLM free-text — consistent with the project's existing "deterministic before LLM" architectural principle (ADR-002/003).

### 6.2 Personalization via emphasis, not new chart types
Where the user's own situation needs to stand out against aggregate data (the occupation ranking, eventually the threshold trend), the fix is the "emphasis" chart pattern — one entity highlighted, the rest grayed — not a new visualization. Reuse over reinvention.

### 6.3 Honesty correction: time-to-permanence, not just time-to-grant
The dashboard initially showed only "time to grant" per visa (e.g., 491 at ~9 months, which reads as fast). This is **misleading** for provisional pathways — 491 requires 3 further years of regional residency before 191 (permanent) eligibility, making its true time-to-permanence ~4 years, slower than 189/190. This was treated as a **correction owed**, not a nice-to-have, given the whole product's positioning has been "we tell you what others gloss over" since the original competitive analysis. **Every pathway comparison in this feature must show both figures.**

### 6.4 Legal boundary — informational and comparative only, never predictive
Everything in this feature must stay on the "explain, don't recommend" side of Migration Act s.276. Concretely:
- ✅ Allowed: "recent 190 invitations needed 75+ points, you have 68" (comparative fact)
- ❌ Not allowed: "you are likely to be refused" / "you should apply for X" (personalized outcome prediction or pathway recommendation)
- The points-improvement levers are phrased as mechanical facts (action → points → typical time), never as advice ("you should do X").
- This is the same guardrail already governing F1-F6; this feature does not relax it anywhere, including in future alert/comparison features (§8).

### 6.5 Source and freshness, per panel
Not one generic "updated daily" claim in the header — each panel that states a fact carries its own source tag (e.g., "SkillSelect · verified today," "Home Affairs · verified today"), matching the citation discipline already used elsewhere in Saathi (checklist items, F4a explanations).

## 7. Interactivity requirements

The approved prototype is genuinely interactive, not a static image of interactivity — this is a requirement, not a nice-to-have, once built for real:
- **Working selector:** changing the occupation (at minimum) live-updates the ceiling gauge, badges, breadcrumb, and insight text. State/visa-type selectors follow the same pattern.
- **Hover tooltips with crosshair** on trend lines (nearest-point snapping, showing exact round + value) and per-bar tooltips on histograms/bar charts.
- **"View as table" toggle** on chart panels — the accessibility and trust fallback (nothing hidden behind a shape you have to interpret).
- **Sticky, scroll-aware jump navigation** — given the page's real length (8 sections), a persistent nav bar that highlights the current section and jumps on click.

## 8. Explicitly deferred (tracked, not dropped)

- Side-by-side occupation comparison tool (compare 2-3 candidates at once, not just browse one at a time)
- "What changed since you last looked" digest — cheap to build (diffing the crawler's own daily output), natural bridge into F5's news feed, high retention value
- Threshold-crossing / ceiling-crossing alerts (reuses F1's existing FCM notification infrastructure)
- EOI queue depth by points band (how many people are waiting at each level, not just the invited threshold) — deferred on data-availability uncertainty, not by choice
- Full dedicated State Report and Visa Type Report pages beyond the "Your.../All..." summaries shown here (e.g., a complete single-visa deep dive analogous to what `reports-complete.html` explored mid-session)
- Next-step CTAs bridging into F1 (track this pathway), F3 (start this checklist), F4 (form help), F6 (talk to a MARN agent) — identified as important (this feature is meant to be the app's spine tying those together) but not yet designed screen-by-screen

## 9. Technical questions — resolved 2026-08-14

### 9.1 The crawler gap (new finding, not anticipated when §9 was first written)

Checked the AU Visa Source Registry crawler's actual current schema (`research/au-visa-sources/notion_registry.py`) rather than assuming. It tracks **pages**, not **facts**: title, URL, category, related-subclass tags, a free-text 2-3 sentence summary, and change status (new/updated/unchanged/dead). It detects that a SkillSelect results page changed; it does not extract "85 points" or "2,624 of 3,200" as a queryable number anywhere in its schema. The original §9 line ("should follow the crawler's existing data shape") undersold this — there is a missing extraction layer between what the crawler produces and what this feature needs, not just a schema-design task.

**Resolution:** a new ingestion/extraction step, tiered the same way manaslu's document extraction already is (`manaslu/docs/architecture/02-scan-extraction.md`) — deterministic template/table parsers for the known, consistently-structured government page types (SkillSelect round tables, occupation ceiling pages, state nomination lists) as the primary tier; Claude-assisted extraction as fallback only for pages that don't fit a known template. This is the same "deterministic before LLM" tiering already validated twice elsewhere in this project family (manaslu's D-something hybrid extraction; the general ADR-002/003 principle) — not a new philosophy to invent.

### 9.2 Data model

Cloud SQL Postgres (Saathi's already-established DB), not a static committed build artifact like bato's `graph.json`. Deliberately not copying bato's pattern here: bato's graph is static published rules, appropriate for a build-time artifact; this feature's data is explicitly time-varying and updates daily, which calls for a live queryable database instead.

Tables: `occupations`, `visa_subclasses`, `ceiling_usage` (occupation × program-year × issued/cap, time-series), `eoi_rounds` (visa × round-date × threshold, optionally occupation-scoped), `state_nomination_status` (state × occupation × status/fee/decision-time), `processing_times` (visa × as-of-date × median-days). Every row carries `source_url` + `retrieved_at`, matching the provenance discipline already required of manaslu's extraction records and bato's node/edge data.

### 9.3 Where this lives

Saathi's own backend — not manaslu (stays scan+fill only, not relitigated here), not a new repo. The extraction/ingestion job reuses the exact deployment shape already planned for F4a's knowledge-service ingestion (a Cloud Run Job), rather than inventing a new operational pattern.

### 9.4 Palette

Re-validated: the four categorical colors actually used in the final mockup (`#3987e5, #d95926, #199e70, #c98500`) are the real validated dark-mode values — the drift flagged earlier had already been self-corrected partway through the session without being noted. **However**, checking closely surfaced a separate, real issue: the mockup's `good`/`warning`/`critical` status roles were wired to categorical slot colors (slots 3/4/8) instead of the dedicated status palette (`#0ca30c`/`#fab219`/`#d03b3b`), which violates the rule that status colors must stay distinct from series-identity colors. **Fix before build:** keep the four categorical slots for series identity only (e.g. visa-type color-coding); switch status roles to the real status tokens.

### 9.5 "What this means" insight generation

Same problem bato already solved for its pathway cards — deterministic, templated natural-language generation from structured facts, not an LLM call (`bato/ingest/cards.py` is a working, tested precedent for exactly this: structured node → templated sentence). Approach: a small template library keyed to data conditions (e.g. `ceiling_pct > 75 AND pace_ratio > 1.1` → "Only {left} places left... and it's filling faster than last year"), evaluated against live data at request time. Both projects are Python/FastAPI — worth checking during implementation whether bato's actual card-template code can be adapted rather than rewritten from scratch.

## 10. Success criteria (for whoever builds this)

The build is faithful to this spec if: the personalization selector genuinely changes what's shown (not just cosmetically); every pathway comparison shows time-to-permanence, not just time-to-grant; no copy anywhere states or implies a personalized outcome, eligibility judgment, or recommendation; every stated fact carries a source and freshness date; and the "All..." aggregate views survive alongside the "Your..." personalized ones (this was dropped twice during design and both times had to be restored — worth being deliberate about in implementation too).
