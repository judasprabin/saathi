# Saathi — Market & Competitive Analysis

**Project:** Saathi — Visa Utility Tool for Nepalese Migrants in Australia
**Consolidated:** August 8, 2026
**Status:** Governing research reference — supersedes `market-research.md` (Jun 28) and `deep-market-analysis.md` (Aug 8), both archived under `docs/research/archive/`
**Governing framing from:** [`research/dispora-nepal/competitive-analysis.md`](../../../research/dispora-nepal/competitive-analysis.md) (Jul 13, 2026) — the pivotal strategic re-assessment. That document remains the source of truth for the wedge decision and is not duplicated in full here; treat it as authoritative if this summary ever drifts from it.

---

## 0. Why this document exists

Three research efforts happened at different times and never fully reconciled: a broad product-analog survey (Jun 28), a deep TAM/SAM/SOM and corridor-data pass (Aug 8), and — in between, on Jul 13 — a competitive re-assessment that discovered ten live visa-AI tools and materially changed the product's positioning. This document merges all three into one coherent, correctly-positioned reference, with the Jul 13 pivot governing the framing throughout.

---

## 1. Positioning: The Wedge (read this first)

**The mechanical form-fill is already commoditized.** "AI extracts your documents and fills a visa PDF" exists today at every price point, including free:

- [Instafill.ai](https://instafill.ai/) has a **dedicated Form 80 page** — 840 fields, 19 pages, "filled in under a minute," with per-field source and reasoning.
- [FormMate80](https://formmate80.com.au/) does the same AU Form 80 fill **for free**.
- [formli.ai](https://formli.ai/en/forms/australia-form80) offers a step-by-step Form 80 guide; generic PDF fillers (pdfFiller, PDFliner, SignNow) all carry Form 80 templates too.

**Any version of Saathi whose headline value is "we fill the form" is dead on arrival.** Competing on fill speed, form breadth, or price is a race against better-resourced horizontal players, and the price floor is already $0.

**What's unoccupied:** a Nepali-language layer, community-trust distribution, bilingual field-by-field explanation, AU-migration depth (MRZ/cross-doc validation, Home Affairs field semantics, cited freshness), and — the actual moat — a **persistent migrant profile vault** reused across the multi-year visa journey (500 → 485 → 189/190), so the *second* form a user fills arrives already ~80% pre-filled. No competitor has this. Every one of them is a transactional, English-only, journey-blind one-shot.

**Headline:** *"Understand and complete your visa forms in your own language, from your own documents"* — not "AI form filler." The vault and bilingual explanation are core MVP, not a v2 nice-to-have.

| Differentiator | Where it lives | Who else has it |
|---|---|---|
| Persistent provenance-tracked profile (fill once, next form ~free) | Profile-facts vault (manaslu docs 03/08) | Nobody — all competitors are one-shot |
| Bilingual explain-while-fill (every field in Nepali at confirmation) | Bilingual field manifests + review events (manaslu docs 03/06) | Nobody |
| AU-migration depth | MRZ/ABN/BSB validators, cross-doc consistency, Form-80 semantics | Generic fillers can't justify single-country depth |
| Trust/compliance posture | Fill-only tool contracts, audit annex, MARN handoff | Structurally absent in "recommendation" products |
| Distribution | Nepali FB groups, community word-of-mouth | Unreachable for horizontal tools |

**Compliance is a moat, not a constraint.** Eligibility-assessment tools (AustraliaVisa.ai, Famsia) operate near the Migration Act's definition of "immigration assistance" without apparent MARA registration. Saathi's fill-only / no-suggestions / MARN-handoff design is a position a community can trust, a regulator can't easily kill, and "recommendation"-structured competitors cannot copy without rebuilding.

**Explicitly not doing:**
- English-first horizontal form filler.
- Eligibility scoring / visa recommendations (legal boundary + already crowded).
- Competing on number of supported forms.
- Per-fill monetization — the price floor is $0 (Instafill, FormMate80 prove it). It's the journey/vault that's monetizable later: free first form, premium vault/multi-form/history, MARN-agent referrals downstream.

---

## 2. Competitive Landscape (the real threat set)

This is the corrected competitor set. It replaces the diaspora-fintech and generic US/India immigration-platform list (Boundless, Immihelp, RapidVisa, etc.) as the *primary* competitive section — those products don't operate in this corridor and aren't the actual threats. See §6 for what remains useful about them as UX analogs.

| Tool | What it actually is | Market | Threat |
|---|---|---|---|
| [Instafill.ai](https://instafill.ai/) | Generic AI PDF filler with a **dedicated Form 80 page**; extracts from source docs, reports per-field source & reasoning | Global, B2B+B2C | 🔴 Highest — commoditizes the mechanical fill, including our exact form |
| [Migraide](https://migraide.com) | Upload passport/docs → AI fills forms + cover letters + checklists. Schengen today, **Australia planned** | Schengen → expanding | 🟠 Closest business-model match; wrong forms so far |
| [Visix](https://visix.me/) | Generic visa form automation, 150+ countries, $0/$29/$99 tiers | Global | 🟠 Breadth play, shallow per-form depth |
| [AustraliaVisa.ai](https://australiavisa.ai/) | AU-specific eligibility checker — scored assessments | AU | 🟡 Different product; does what Saathi legally refuses to do |
| [Famsia](https://famsia.ai/) | Eligibility checks 190+ countries, itineraries, cover letters | Global | 🟡 Thin/generic |
| [Fillwise](https://fillwise.ai/) | Chrome extension for travel agents: Schengen/UK/ESTA/eTA forms, batch mode | Travel-agent B2B | 🟡 Short-stay visas, agent workflow — not AU migration |
| [Visas.AI](https://visas.ai/) | Platform for US immigration attorneys; AI paralegal, briefs, RFEs | US legal B2B | ⚪ Different country & buyer |
| [Torly.ai](https://torly.ai/) | UK Innovator Founder Visa business-plan builder | UK founders | ⚪ Unrelated niche |
| [Quillio](https://quillio.au/) | AI legal assistant for AU lawyers — not a visa tool | AU legal B2B | ⚪ Not a competitor |
| nuronai.org | Dead (404) | — | ⚪ — |

Also present but not in the primary table: [FormMate80](https://formmate80.com.au/) (free AU Form 80 filler) and [formli.ai](https://formli.ai/en/forms/australia-form80) (Form 80 guide) — both reinforce F1 above.

**Confirmed absent, checked directly:** any Nepali-language AU-immigration product, anywhere. The only official Nepali-language touchpoints found are Home Affairs' [digital assistant](https://immi.homeaffairs.gov.au/help-support/tools/digital-assistant) (TIS interpreter phone access) and [AUSCO settlement PDFs](https://immi.homeaffairs.gov.au/settling-in-australia/ausco/information-in-your-language/nepali) — static, not interactive, not form-specific.

---

## 3. Market Sizing — TAM / SAM / SOM

### Corridor context

| Metric | Value | Source |
|---|---|---|
| Nepali ancestry in Australia (2021 census) | 138,463 (0.54% of AU pop) | ABS Census 2021 |
| 5-year growth (2016→2021) | +120% | ABS |
| Nepal-born residents (DFAT est.) | ~200,000 | DFAT Nepal Country Brief |
| Nepal as source of AU international students | #3, after China/India | DFAT, 2024 |
| Remittances AU→Nepal (2021) | US$466.6M | DFAT |
| Migration agent fees paid (typical range) | $800–$8,000+ AUD per case | Community research |

Key pain points feeding the wedge: government services are English-only (no Nepali on myGov, immi.homeaffairs.gov.au, SkillSelect, VEVO); "ghost college" crackdowns and elevated refusal rates for students from "high-risk" countries (Nepal included); heavy reliance on Facebook groups/YouTube/word-of-mouth for guidance, with real misinformation risk; and document-verification complexity across Nepali educational, police, and health records.

### TAM / SAM / SOM

| Layer | Estimate |
|---|---|
| **TAM — free tier** | ~250,000 people (all Nepali-speakers interacting with AU immigration systems; 2021 census 138,463, extrapolated to ~180,000+ by 2026 plus students/new arrivals) |
| **TAM — premium tier ($7.99/mo)** | ~180,000 active immigration navigators (engaged in a visa process over a 3-year window) |
| **SAM — free tier** | ~108,000–150,000 users over 3 years (95% smartphone penetration × ~80% app comfort × ~60% would use a Nepali-language tool) |
| **SAM — premium tier** | ~30,000–50,000 paying users at $7.99/mo |
| **SOM Year 1 — free** | 13,000–23,000 users (FB group referrals, student associations, agent partnerships, organic, YouTube collabs) |
| **SOM Year 1 — paid** | 400–1,150 subscribers (3–5% conversion estimate) |

Revenue at these SOM levels is modest ($38K–$110K/year from subscriptions alone) — the point of these numbers is validation of addressable scale, not a funding case. Treat every figure above as an estimate pending the primary research in §7; none of it is validated with real user interviews yet.

### Migration agent (MARA/OMARA) market — referral economics

~5,000–7,000 registered migration agents nationally; an estimated 200–400 serve the Nepali community, of which ~50–100 are Nepali-speaking. Typical fees run $800–$2,000 for a 485, $3,000–$7,000 for skilled-visa categories. Rough total addressable agent-fee market for the Nepal-AU corridor: **~$30M–$45M/year** (30,000–45,000 new arrivals × ~40% using agents × ~$2,500 avg fee). A referral model at 20–100 referrals/month and $100–$200/referral yields $24K–$240K/year — real but secondary to the vault/subscription path, and it depends on the MARN-handoff design in §1 staying structurally compliant.

---

## 4. Nepali-Language Gap — Qualitative Confirmation

Both the Jun 28 and Aug 8 research passes independently confirm the same finding later validated in the Jul 13 competitive re-assessment (§2): **no Nepali-language immigration product exists for the Australia corridor**, from any source — indirect immigration-tech competitors (Boundless, Immihelp, RapidVisa) are US-focused and English-only; Nepal-focused consumer apps (Hamro Patro, eSewa, Khalti, Daraz Nepal) have no immigration features; and government tools (myGov, immi.homeaffairs.gov.au, SkillSelect, VEVO) are English-only with no immigration-specific guidance layer.

Information-seeking behavior reinforces the distribution channel choice: Facebook groups and word-of-mouth are the dominant, highest-trust channels (100k+ members across groups like "Nepalese in Australia," "Nepalese Students in Australia"); YouTube is high-reach but low-to-medium trust; government websites are used directly by under 20% of the community because they're too complex. Bilingual code-switching is the norm — technical terms ("subclass 500," "EOI," "SkillSelect") stay in English even inside Nepali conversation, which is why Saathi's bilingual-not-Nepali-only approach is correct.

---

## 5. Go-to-Market Patterns from Diaspora Fintech

These are **not competitors** — they don't touch immigration or the AU-Nepal corridor — but their community-first playbooks are the most relevant GTM precedent available, and they showed up independently in both earlier research passes.

| Company | Model | Growth lever | Application to Saathi |
|---|---|---|---|
| **Majority** (Nigerian immigrants → US, ~$83.5M raised) | Migrant neobank, one community first (Houston) then expanded | Extreme community specificity, physical + digital hubs, remittance pain-point wedge | Own Nepali-in-Sydney deeply before expanding; immigration anxiety is Saathi's equivalent wedge |
| **Welcome Tech / SABEResPODER** (Latino immigrants → US, ~$70M raised) | Spanish-language info platform, expanded to services marketplace | Trusted in-language content first, monetize adjacent services later; UGC rewards for shared experience | The exact arc Saathi should follow: information → community → marketplace |
| **Comun** (Latino immigrants → US banking, ~$21.5M Series A) | Spanish-first neobank, accepts foreign IDs | Community ambassadors, bilingual support, alternative-ID acceptance | Accept Nepali documents; Nepali-speaking community ambassadors as a channel |
| **Bloom Money** (diaspora ROSCA app, UK, modest ~£1.5M pre-seed) | Digital money-circle app for diaspora communities "straddling two countries" | Cultural practice digitized, community leaders as distribution | "Straddling two countries" framing maps directly to Nepal↔Australia migrants; NRNA chapters as leaders |
| **Leverage Edu** (India study-abroad, $62M+ raised) | AI coaching chatbot + free tools, premium upsell | Free AI tools as lead magnet, native-language content marketing | Free Points Calculator/checklist as lead magnet → premium vault upsell |

**Key lessons, consolidated:** narrow focus wins (one corridor, deeply, before expanding); language is the trust layer that generic competitors structurally can't replicate; community ambassadors (FB group admins, NRNA chapters) are the distribution channel, not paid acquisition; freemium free-tool-then-premium is the proven monetization shape; and the founder-shape match (solo/small team building for their own community) is itself part of the credibility story, as it was for Majority and Bloom Money.

---

## 6. Additional Product/UX Analogs (reference only, not competitors)

The Jun 28 survey covered several US/India immigration and marketplace products that are **not part of the competitive threat set** (wrong country, wrong regulatory regime, wrong language) but still contain reusable product patterns:

| Pattern | Source | Note |
|---|---|---|
| In-language questionnaire → form prep → licensed-professional handoff | Boundless (acquired by Payoneer, Jan 2026) | Validates "information + referral, not advice" as a model that can exit |
| Community-experiences content section | Immihelp | High-value, no AI tool replicates this well yet |
| "Grounded or silent" answering (never guess) | Boundless | Directly applicable to Saathi's citation discipline |
| Conversational Q&A with professional handoff when stuck | Abe.ai | Core UX loop pattern |
| Verification-first, quote comparison, escrow-on-milestone | Zocdoc / Thumbtack marketplace patterns | Relevant if/when a MARN-agent marketplace is built; take-rate/disintermediation risk noted as real |

None of these should appear in competitive positioning copy — they're UX inspiration, not threats, and conflating them with the real competitor set in §2 is exactly the framing error this document corrects.

---

## 7. Kill Criteria and Immediate Next Step: 2-Week Cheap Validation

These are carried forward from `competitive-analysis.md` §5–6 as **the current action item**, not background reading. Nothing beyond wedge-critical components (vault, manifest, bilingual explain data model) should get more build time until this validation runs.

### Kill criteria (honest tripwires)

1. **Community validation fails** — the Nepali landing-page/FB-group test below produces fewer than ~100 genuinely interested users, or no engagement on real questions → stop or pivot audience.
2. **Instafill or Migraide ships a Nepali-localized AU-migration flow with a persistent profile** → the wedge is gone; do not fight it.
3. **Legal review concludes even fill-only transcription is MARA-restricted** → product as designed is not shippable; only the explain/checklist layer survives.
4. **Unit economics fail** — cost per completed form-fill session persistently above ~$0.50 with no willingness to pay for premium → engine is a hobby, not a business.

### 2-week validation plan (≪ $500)

1. Nepali landing page ("फारम भर्न सहयोग, आफ्नै भाषामा") + waitlist, seeded via small FB groups. Target: 100+ signups.
2. Two weeks of question harvesting in Nepali-in-Australia FB groups — count Form-80/document questions specifically.
3. Five concierge form-fills done manually with real community members — measures trust, extraction difficulty, and time-to-value before any automation exists.
4. Three to five MARN agent conversations — validates the referral leg of monetization (§3).

---

## 8. Verification Flags — Check Before Any External Use

Two items surfaced while consolidating this research that were **not independently verified** and should be checked before any of the surrounding documents are shown outside the team:

1. **AATA case citations in `docs/legal/legal-memo.md`** — *Re Goldfarb [1999] AATA 167* (on the clerical exemption) and *Pick v. Department of Home Affairs [2004] AATA 1234* (on what constitutes immigration assistance) are cited as precedent for the legal risk assessment. These have not been independently confirmed against an AustLII/AATA case lookup. Verify both citations resolve to real, correctly-summarized decisions before the legal memo's conclusions are relied on externally.
2. **`omahe.gov.au` domain** — `docs/legal/legal-memo.md` cites `https://www.omahe.gov.au` repeatedly (OMARA home, registered-agent lookup, "who is a registered migration agent") as the OMARA reference domain. This does not look like a domain that should be silently trusted or copied forward — it should almost certainly be a real OMARA/Home Affairs domain, but the correct one hasn't been confirmed here. (Note: the brief that scoped this consolidation flagged this domain as appearing in `fallback-answer-sheet.md`; on inspection it does not appear there — `fallback-answer-sheet.md` only references "a registered migration agent (MARN-registered)" with no URL. The domain was actually found in `legal-memo.md`, at the locations above.) Flag both files for a domain audit; do not propagate `omahe.gov.au` into new user-facing copy until the correct OMARA domain is confirmed.

---

## 9. Sources

**Competitive landscape (§1–2):** visas.ai, famsia.ai, fillwise.ai, instafill.ai (+ /forms/80), australiavisa.ai, visix.me, quillio.au, nuronai.org (dead), torly.ai, migraide.com, formmate80.com.au, formli.ai, immi.homeaffairs.gov.au (digital assistant, Nepali AUSCO resources). Fetched/searched July 13, 2026.

**Market sizing (§3):** Australian Bureau of Statistics (ABS) 2021/2016 Census; DFAT Nepal Country Brief; Department of Education (Australia) international student data; OMARA agent registry structure; Wikipedia (Nepalese Australians, Ghost colleges in Australia, International students in Australia). Note limitations: no direct API access to immi.homeaffairs.gov.au visa-grant data by nationality, no programmatic MARA registry query, no primary user survey yet — see §7 for the plan to close this gap.

**GTM patterns (§5) and UX analogs (§6):** getmajority.com, saberespoder.com (Welcome Tech), comun.life, bloommoney.com, leverageedu.com, boundless.com, immihelp.com, abe.ai, plus general marketplace pattern review (Zocdoc, Thumbtack).

**Governing document:** `research/dispora-nepal/competitive-analysis.md` (Jul 13, 2026) — read that document in full for the wedge decision, differentiator rationale, and full source list behind §1–2 and §7.
