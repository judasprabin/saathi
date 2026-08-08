# Saathi — Product Requirements Document

## Visa Utility Tool for Nepalese Migrants in Australia

**Version:** 2.1 — Repositioned post-competitive analysis; unified with manaslu
**Date:** August 8, 2026 (v2.0 was June 28, 2026)
**Status:** Validated scope — ready to build
**Author:** Prabin Karki

> **Scope reset from v1.0:** Previous versions grew to 38 features across 3 phases. v2.0 reset to the 4 core utility features that actually solve the problem: visa tracking, points calculator, document checklist, and form helper. **v2.1** incorporates the July 13, 2026 competitive re-assessment (`research/dispora-nepal/competitive-analysis.md`): mechanical form-filling is already commoditized by real competitors (Instafill has a dedicated Form 80 page; FormMate80 does it free), so Saathi survives only as the wedge product — Nepali-language, persistent profile vault, bilingual explain, compliance trust — not a generic filler. The vault moves from implicit to explicit core scope. F4's scan/fill engine is built in a separate repo, `manaslu`, and consumed via API — not rebuilt here.

---

## 1. What Saathi Is

**"Understand and complete your visa forms in your own language, from your own documents."** Not "AI fills your form" — that's already commoditized (see §2). A focused visa utility tool for Nepalese migrants in Australia that does four things well:

1. **Track** your visa status and upcoming deadlines
2. **Calculate** your skilled migration points score
3. **Generate** a personalised document checklist for your visa
4. **Understand** what Australian immigration forms are asking and how to fill them

All explanations are in **plain Nepali**, sourced from official Australian government pages, and cited. Saathi provides information only — it does not lodge applications or give migration advice.

---

## 2. The Problem (Tight Version)

Nepalese migrants face four recurring, high-friction problems with Australian immigration:

| Problem | Current fix | Why it fails |
|---------|------------|--------------|
| "When does my visa expire and what do I need to do next?" | Calendar reminders, Facebook groups | No structured tracking; miss deadlines |
| "Do I have enough points for skilled migration?" | Agent consult ($200–$500 for a preliminary chat) or confusing ATO/Home Affairs calculators | Expensive or too complex to self-serve |
| "What documents do I actually need for this visa?" | Scattered forum posts, outdated checklists | Out of date, generic, not in Nepali |
| "I don't understand what this form field is asking" | Ask a friend, pay a migration agent | Wrong answers or expensive |

These are mechanical, high-frequency problems. They don't require a platform — they require a well-built tool.

---

## 3. Target User

**One persona for MVP:** A Nepalese person in Australia on a temporary visa (student, graduate, skilled) who is managing their own immigration journey — either because they can't afford a migration agent for every question, or because they want to understand their own situation before engaging one.

- Age: 22–35
- Location: Sydney, Melbourne
- Visa: Student (500), Graduate (485), Skilled (189/190/491), or TSS (482)
- Language: Reads Nepali; understands some immigration English but finds it stressful

---

## 4. The Four Core Features

### F1 — Visa Tracker

Track the user's current visa and surface what matters next.

**What it does:**
- User enters their visa subclass and grant date (or expiry date)
- Saathi shows: visa expiry, days remaining, key conditions (e.g. work hour limits on 500, no-work on some bridging visas), and the natural "next step" for that visa type
- Push reminders at 180, 90, 30, 7 days before expiry

**What it does NOT do:** Check ImmiAccount or query Home Affairs systems. All data is user-entered. Saathi does not connect to the Department of Home Affairs.

**Key visa types to support at MVP:**
- Subclass 500 (Student)
- Subclass 485 (Temporary Graduate)
- Subclass 482 (TSS / Skills in Demand)
- Subclass 189/190/491 (Skilled — permanent/provisional)
- Bridging Visa A/B/C/E

**Acceptance:** User can add a visa, see days remaining, and receive a reminder. Key conditions for each visa type are accurate and cited.

---

### F2 — Points Calculator

Help users understand whether they are competitive for General Skilled Migration (GSM) before spending money on an agent.

**What it does:**
- Step-by-step questionnaire in Nepali: age, English test score, work experience, qualification, Australian study, specialist education, partner skills, state nomination
- Shows total points score, how it compares to recent invitation rounds (from the published SkillSelect data), and a plain-Nepali explanation of each component
- Flags which factors could improve their score and by how much

**What it does NOT do:** Tell the user they are eligible or recommend a specific visa subclass. Output always includes: "This is an estimate for your information only. For a formal assessment, consult a registered migration agent (MARN)."

**Points criteria to implement:**
- Age (18–44 only; 0–30 points)
- English language ability (0–20 points; IELTS/PTE/TOEFL/OET)
- Skilled employment — outside Australia (0–15 points)
- Skilled employment — in Australia (0–20 points)
- Educational qualifications (10–20 points)
- Australian study requirement (5 points)
- Specialist education qualification (5 points)
- Accredited community language (5 points)
- Study in regional Australia (5 points)
- Partner skills (0–10 points)
- Professional Year in Australia (5 points)
- State/territory nomination (5 points) or family in regional AU (15 points)

**Acceptance:** Calculator produces a correct score for a test set of 10 profiles; recent SkillSelect round data is shown with source + date; disclaimer is always visible.

---

### F3 — Document Checklist

Generate a personalised document checklist for any supported visa type.

**What it does:**
- User selects their visa type and answers 3–5 branching questions (e.g. "Do you have a partner to include?", "Did you study in Australia?", "Are you applying onshore or offshore?")
- Saathi generates a checklist of required documents: what each document is called, what it needs to show, where to get it, and common mistakes
- Each checklist item is explained in plain Nepali
- Checklist is printable / shareable

**Supported visa types at MVP:**
- 485 (Temporary Graduate) — both Graduate Work and Post-Study Work streams
- 500 (Student) — extension or fresh application
- 189 (Skilled Independent)
- 190 (Skilled Nominated)
- 491 (Skilled Work Regional)
- 482 (TSS / Skills in Demand) — employer-sponsored

**Acceptance:** Checklist for each supported visa matches current Home Affairs requirements; each item is cited with the source page and "last verified" date; branching logic produces different checklists for different scenarios.

---

### F4 — Form Helper

Help users understand what Australian immigration forms are asking, and — the actual moat — fill forms from a persistent profile built from their own documents, reused across their multi-year visa journey.

**What it does:**
- **F4a — Explain (built here):** user selects a form (e.g. Form 80, Form 1085, 485 online form sections) or describes what they're filling in. Saathi explains each field in plain Nepali: what is being asked, why it matters, what a correct answer looks like, and common mistakes. For ambiguous fields, Saathi explains the options without choosing for the user.
- **F4b — Scan & Fill (built by `manaslu`, a separate headless agent service, consumed here via API):** user uploads documents; the agent classifies, extracts, and validates them, transliterates Devanagari names to Latin with the user's confirmation, and fills the AcroForm PDF from **confirmed, provenance-tracked values only**. Values are also saved to a persistent **profile vault** — the second form a user fills (e.g. Form 1221 after Form 80) arrives already ~pre-filled from data captured the first time, with every value still traceable to its source document. This vault is the product's actual differentiator; see `research/dispora-nepal/competitive-analysis.md`.

**What it does NOT do:** Submit the form. Does not access ImmiAccount. Does not make eligibility assessments. For any field where the answer depends on the user's specific legal situation, Saathi says "This depends on your specific circumstances — ask a registered migration agent."

**Forms to support at MVP:**
- Form 80 (Personal Particulars for character assessment) — used across many visa types
- Form 1221 (Additional personal particulars) — commonly required
- 485 online application key sections (identity, qualifications, health, character)
- 189/190/491 EOI (Expression of Interest) key sections in SkillSelect

**Acceptance:** Field explanations are accurate; no field explanation constitutes migration advice; disclaimer present on every response; tested with real forms.

---

### F5 — News, Seminars & Opportunities (Phase 2 — after F1–F4 traction)

A curated Home feed: immigration/visa news with Nepali AI summaries, plus community seminars, education expos, scholarships, and info sessions — aggregated from other sites, never republished.

**What it does:**
- **News:** allowlisted sources (Home Affairs, SkillSelect, state programs, SBS Nepali, vetted sector press) ingested on a schedule; each item = headline + labelled AI-generated Nepali summary (≤2 sentences) + source + date + link out to the original. Personalised by visa subclass ("you're on 500 → 485 news pinned"), with the why shown.
- **Seminars & events:** human-curated listings (city + online filters) with date, venue, free/paid, audience tags. Registration always happens on the organiser's site — Saathi holds no attendee data or payments. Optional reminders (FCM) and calendar export.
- **Student corner:** scholarship deadlines, intake dates, PY/NAATI info sessions, university open days — deadline-first, every listing sourced and last-verified.

**Trust rules (non-negotiable):**
- Migration-topic seminars list **only MARN-verified presenters** (number shown, linked to the mara.gov.au register). Unregistered "agents" running seminars are the community's biggest scam vector; refusing them is part of the moat.
- Copyright: aggregation only — headline + short summary + attribution + link out. No full-text republishing.
- AI summaries are always labelled, with "the linked source is authoritative."
- No course/college recommendations — listings only, disclaimer in both languages.

**Why Phase 2, not MVP:** this is the retention/daily-touch layer and the natural future surface for the MARN-referral monetisation (§7) — but it's content ops, not product validation. It ships only after the four core tools prove traction (§9's discipline). The news half reuses the ingestion worker already planned for tracker rule-change detection, so the marginal engineering is small; the real cost is curation time.

**Acceptance (when built):** feed loads with ≥5 fresh items/week; every item links out and carries source + date; every migration seminar shows a verifiable MARN; zero full-text reproductions.

---

### F6 — Connect to an Agent (Phase 2 — the monetisation surface, traction-gated)

Operationalises the MARN handoff: every "consult a registered agent" line in the app becomes a tappable entry into a directory of **MARA-registered agents only**, with enquiry and request-a-call flows. **English-only UI** by product decision (agent correspondence happens in English); the bilingual handoff lines elsewhere remain the entry points.

**What it does:**
- **Directory:** MARN-verified agents with specialisations, languages ("Speaks Nepali" is a first-class filter), location/online, upfront consult fees, response times. Sorted by specialisation match — **never pay-to-rank**.
- **Enquiry / request a call:** topic + optional short message + contact preference and call-time windows.
- **Consent review before send:** item-by-item opt-in of exactly what the agent sees (contact + enquiry pre-ticked; visa/points/checklist summaries **off by default**). Consent is per-agent, per-enquiry, revocable, audit-logged.
- **My Enquiries:** status tracking (Sent → Viewed → Responded), accept a proposed call time, revoke shared data (agent is notified with a deletion obligation).

**Trust rules (non-negotiable):**
- Agents listed only with a MARN checked against the MARA register at onboarding and re-verified quarterly; lapsed registration = auto-delisted.
- Referral-fee disclosure on the agent profile **and** at the point of send: "Saathi may receive a fee if you engage this agent — your price is unaffected."
- **Documents and filled forms never flow through F6** — manaslu's vault has no API path to it, structurally. Users bring documents to the consult themselves.
- Saathi brokers the introduction only: the advice relationship is directly client ↔ agent; Saathi cannot see the conversation.
- No public star-ratings at launch (defamation/gaming risk on regulated professionals); private "responded professionally?" feedback feeds delisting decisions instead.

**Why Phase 2:** this is PRD §7's referral-fee monetisation — it only works once user volume exists, and it shares F5's trust prerequisites (MARA verification ops). Sharing user PII with third parties also requires the privacy policy and consent framework from Phase 5's legal review to be in place first.

**Acceptance (when built):** every listed agent's MARN independently verifiable; zero data shared without the itemised consent record; revocation round-trip works; referral disclosure present on profile + send screens.

---

## 5. Regulatory Guardrail

Every feature is bounded by this:

- ✅ **Allowed:** explaining what a form field means; showing what documents are required; calculating a points score for information purposes; tracking dates the user provides.
- ❌ **Not allowed:** assessing a person's eligibility for a specific visa; recommending a visa pathway; lodging or preparing forms on a person's behalf; telling someone they will or won't get a visa.
- 🔁 **Handoff:** whenever a question moves into advice territory, Saathi stops and says: "This needs a registered migration agent. You can verify an agent's registration at mara.gov.au using their MARN."

Get written legal advice confirming the information-only boundary before launch.

---

## 6. Technical Stack (Minimal)

| Layer | Choice | Reason |
|-------|--------|--------|
| Frontend | Next.js PWA on Cloud Run | Mobile-first; no app store barrier; one deploy pattern with the API |
| Backend (F1-F3, F4a) | FastAPI on Cloud Run | Lightweight; async; same pattern manaslu uses |
| F4b (scan/extract/fill) | **manaslu** — separate headless agent repo, consumed via REST+SSE API | Already built, GCP-native, purpose-designed around the vault moat — not rebuilt here |
| LLM | Claude Sonnet 5 | Nepali generation + document explanation |
| Retrieval | RAG (pgvector + Voyage AI embeddings) over official source corpus | Grounding; citations; "last verified" dates; Voyage chosen over OpenAI for Nepali retrieval quality |
| Database | Cloud SQL for PostgreSQL | Same engine as manaslu; one auth system (GCP Identity Platform) across both services |
| Hosting | GCP (Cloud Run + Cloud SQL), Terraform in `karki-labs-infra` | One cloud, reuses infra already provisioned for manaslu |

**Knowledge freshness:** Official source pages (Home Affairs, ATO, SkillSelect) are ingested on a schedule with change detection. Every checklist item and calculator component carries a "last verified" date. Stale data is more dangerous than no data.

---

## 7. Monetisation (Simple)

| Model | When |
|-------|------|
| Free (all 4 features) | Launch — grow the user base |
| Referral fee from MARN-verified migration agents | Once user volume is there; agents pay for qualified leads |
| Optional premium: saved profiles, unlimited checklists, PDF export | After retention is proven |

No marketplace, no subscriptions at launch. Validate demand first.

---

## 8. Success Metrics

| Metric | Target |
|--------|--------|
| Checklist completions | Users generate and save a checklist |
| Points calculator completions | User gets a score and shares/saves it |
| Form Helper sessions | User asks about ≥3 fields per session |
| Visa Tracker DAU | Users return to check their tracker |
| "Was this helpful?" rating | ≥70% positive |
| 30-day retention | Users return within 30 days of first use |

---

## 9. What's Explicitly Out of Scope

The following will NOT be built until the four core features have traction:

- Mental health / crisis support
- Workplace rights / Fair Work
- Agent marketplace (Connect) — *exception: F6 (§4), the enquiry/referral flow, is now specced as a Phase 2 follow-on; still gated on core-feature traction + the Phase 5 legal/privacy review. The full marketplace (payments, bookings, case management) remains out of scope*
- Community platform / experiences section — *exception: F5 (§4), the curated news/events feed, is now specced as the planned Phase 2 follow-on; it is still gated on core-feature traction*
- Skills assessment deep-dive
- Dual-country financial tools
- Voice input
- CV localisation
- Any B2B features

---

## 10. Build Sequence

1. **Week 1–2:** Document Checklist (F3) — highest immediate value; pure content + branching logic; no LLM needed for MVP version
2. **Week 2–3:** Points Calculator (F2) — rule-based; deterministic; easy to test
3. **Week 3–5:** Visa Tracker (F1) — user accounts + notifications; slightly more infra
4. **Week 5–8:** Form Helper (F4) — requires RAG pipeline + LLM; most complex; build last

---

## Document History

| Version | Date | Changes |
|---------|------|---------|
| 0.1–0.3 | June 10, 2026 | Pre-validation draft; market scoping |
| 1.0 | June 28, 2026 | Research-complete; 38 features; scope too broad |
| 2.0 | June 28, 2026 | Scope reset: 4 core features only (tracker, calculator, checklist, form helper); everything else deferred until traction |
| 2.1 | August 8, 2026 | Repositioned per July 13 competitive re-assessment (wedge/vault framing, corrected competitor set); F4b (scan/fill) unified with the `manaslu` repo instead of being rebuilt in saathi; stack consolidated to GCP-native to match manaslu |
| 2.2 | August 8, 2026 | Added F5 — News, Seminars & Opportunities (curated aggregation feed + Home digest) as an explicitly Phase 2, traction-gated feature with MARN-verification and no-republishing trust rules |
| **2.3** | **August 8, 2026** | **Added F6 — Connect to an Agent (MARA-verified directory + enquiry/request-a-call + itemised share consent, English-only UI) as the Phase 2 monetisation surface; full marketplace still out of scope** |
