# Saathi — Product Requirements Document

## Visa Utility Tool for Nepalese Migrants in Australia

**Version:** 2.0 — Scope Reset (Core Utility Focus)
**Date:** June 28, 2026
**Status:** Validated scope — ready to build
**Author:** Prabin Karki

> **Scope reset from v1.0:** Previous versions grew to 38 features across 3 phases. This version resets to the 4 core utility features that actually solve the problem: visa tracking, points calculator, document checklist, and form helper. Everything else is deferred until traction is proven.

---

## 1. What Saathi Is

A focused visa utility tool for Nepalese migrants in Australia. It does four things well:

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

Help users understand what Australian immigration forms are asking and what to enter — field by field.

**What it does:**
- User selects a form (e.g. Form 80, Form 1085, 485 online form sections) or describes what they're filling in
- Saathi explains each field in plain Nepali: what is being asked, why it matters, what a correct answer looks like, and common mistakes
- For ambiguous fields, Saathi explains the options without choosing for the user

**What it does NOT do:** Fill in or submit the form. Does not access ImmiAccount. Does not make eligibility assessments. For any field where the answer depends on the user's specific legal situation, Saathi says "This depends on your specific circumstances — ask a registered migration agent."

**Forms to support at MVP:**
- Form 80 (Personal Particulars for character assessment) — used across many visa types
- Form 1221 (Additional personal particulars) — commonly required
- 485 online application key sections (identity, qualifications, health, character)
- 189/190/491 EOI (Expression of Interest) key sections in SkillSelect

**Acceptance:** Field explanations are accurate; no field explanation constitutes migration advice; disclaimer present on every response; tested with real forms.

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
| Frontend | Next.js PWA | Mobile-first; no app store barrier |
| Backend | FastAPI | Lightweight; async |
| LLM | Claude (claude-sonnet-4-6) | Nepali generation + document explanation |
| Retrieval | RAG over official source corpus | Grounding; citations; "last verified" dates |
| Database | Supabase (Postgres) | Simple; handles auth + user data |
| Hosting | Vercel + Railway | Fast to ship; cheap to validate |

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
- Agent marketplace (Connect)
- Community platform / experiences section
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
| **2.0** | **June 28, 2026** | **Scope reset: 4 core features only (tracker, calculator, checklist, form helper); everything else deferred until traction** |
