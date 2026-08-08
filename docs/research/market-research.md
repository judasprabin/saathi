# Saathi — Market Research Document

**Project:** AI Settlement & Immigration Companion for the Nepalese Diaspora in Australia
**Date:** June 28, 2026
**Author:** Prabin Karki
**Status:** Research phase — feature & scope decision support

---

## 1. Purpose & Scope of This Document

This document catalogues existing products — apps, websites, services, and platforms — that Saathi can learn from, draw inspiration from, or differentiate against. It is organized by category and concludes with a **Feature Blueprint** summarizing what Saathi should copy, adapt, or avoid.

**Research scope:** Immigration tech, diaspora fintech, document AI, community platforms, settlement services, and professional services marketplaces serving immigrant communities globally. Products span US, UK, Australia, and India markets.

---

## 2. Category I — Immigration & Visa Assistance Platforms

### 2.1 Boundless (USA) — The Closest Analog ⭐⭐⭐⭐⭐

**Website:** https://www.boundless.com
**What it is:** Immigration form preparation + attorney handoff platform. Sends questionnaires in the user's language, fills out immigration forms, and hands regulated legal work to licensed attorneys.
**Pricing:** ~$750 marriage green card, ~$395 citizenship (flat-rate)
**Funding:** ~$45M raised. **Acquired by Payoneer, January 2026.**
**Target:** US family and employment immigrants

**What Saathi should copy:**
- The **exact mechanic**: in-language questionnaire → form preparation → handoff to licensed professional
- Flat-rate transparent pricing (even though Saathi's MVP is free, the pricing model validates willingness to pay)
- **Boundless was acquired** — this proves the "information + referral, not advice" model works commercially and can exit

**Key features observed:**
- Process explainers (family immigration, work visas, citizenship)
- Document checklist per visa type
- Attorney matching and case status tracking
- Educational content (U.S. Immigration 101, guides)
- Visa bulletin tracking
- Form preparation tools

**What Saathi should NOT copy:**
- US-specific workflow (very different from Australian immigration system)
- Full form preparation (regulated activity in AU without a registered agent license)
- The attorney employment model (Saathi is solo/small team, not an employer of attorneys)

**Relevance to Saathi:** Direct mechanic match. The most important precedent for the product architecture.

---

### 2.2 Immihelp (USA)

**Website:** https://www.immihelp.com
**What it is:** Long-standing community-driven US immigration information portal with forums, visa trackers, document checklists, and a marketplace for immigration services.
**Pricing:** Free content + paid services through partners
**Target:** Global immigrants to the US (diverse nationalities)

**What Saathi should copy:**
- **Community experiences section** — real user stories about visa processes, timelines, interview experiences. This is extremely high-value content that no AI tool has replicated well
- **Visa processing tracker** — users can track case status
- **Document checklist by visa type** — comprehensive, community-vetted
- **Multilingual support** — Chinese and Spanish editions demonstrate the in-language model
- **Forum/community Q&A** — peer-to-peer help at the bottom of the pyramid (non-professional, non-legal)
- **Blog with current immigration news** — keeps content fresh, SEO-friendly

**What Saathi should NOT copy:**
- The forum moderation burden (requires significant community management)
- General US immigration coverage (too broad — Saathi should stay niche)

**Relevance to Saathi:** Best-in-class content architecture for immigration information. The community experiences model is directly applicable to the Nepalese community.

---

### 2.3 Leverage Edu (India) — Edtech with AI Coaching

**Website:** https://leverageedu.com
**What it is:** Indian study abroad platform covering admissions, financing, housing, and career outcomes. Recently mentioned in TechCrunch (Oct 2025) for rerouting students affected by visa crackdowns.
**Pricing:** Free tier + paid services
**Target:** Indian students seeking international education (Australia, UK, Canada, US, Europe)

**What Saathi should copy:**
- **"LE AI" — AI coaching chatbot** for study abroad questions (this is the direct parallel to Saathi's AI explainer)
- University/course matching algorithm (analogous to Saathi's process-matching)
- End-to-end lifecycle: admissions → visa → housing → career outcomes (Saathi should think about this full lifecycle for Nepalese migrants)
- **"Talk to an Expert" CTA** — hybrid AI + human model
- Emerging destinations feature (alternative country routing when primary visa is rejected — interesting for Saathi's "what are my options?" use case)

**What Saathi should NOT copy:**
- University commission model (different monetization)
- Broad course database (not relevant to Saathi's scope)

**Relevance to Saathi:** The AI coach feature is the closest edtech parallel to Saathi's Q&A explainer. The full-lifecycle approach validates Saathi's expansion beyond just the 485 visa.

---

### 2.4 RapidVisa (USA)

**What it is:** Online visa and green card application service for US immigration. Covers fiancé visas, spouse visas, and adjustment of status.
**Pricing:** Flat fee per application stage
**Target:** US family-based immigrants

**What Saathi should copy:**
- **Step-by-step guided process** with clear progress indicators
- **Document upload and organization** feature
- Plain-language explainers at each step
- Mobile-responsive PWA approach

**Relevance to Saathi:** The UX pattern of "here's where you are, here's what comes next, here's what you need" is exactly what Saathi's checklist and deadline tracker should do.

---

### 2.5 Path / Pathgrade (USA)

**What it is:** Immigration tech startup focused on H-1B and employment-based green card processes. Uses AI to screen candidates for visa eligibility.
**Target:** Indian tech workers in the US on H-1B

**What Saathi should copy:**
- **Skills assessment matching** — connecting people to visa categories they may be eligible for
- **Timeline projection** — based on current processing times (extremely valuable for Nepalese 485 → PR pathway planning)
- **Document requirement engine** — personalized based on employment history and education

**Relevance to Saathi:** The timeline projection feature for the Australian PR pathway would be extremely valuable. The Nepalese community faces long wait times for skilled migration — projecting realistic timelines would be a high-demand feature.

---

### 2.6 VisaCoach (Various)

**What it is:** A US-based subscription service offering guided visa application assistance. Offers video courses, document templates, and coaching calls.
**Pricing:** Subscription model ($29-$99/month)
**Target:** Various visa categories

**What Saathi should copy:**
- **Subscription model** for premium features (Saathi's freemium approach)
- Video explainers (Saathi could do short Nepali-language video explainers for key processes)
- Document template library (Saathi could offer Nepali-language document templates)

---

### 2.7 Australian-Specific Resources (Reference Only)

**immi.gov.au** — Official Australian immigration website. Not a competitor but the source of truth Saathi must cite. The site is dense, English-only, and notoriously hard to navigate. **This is Saathi's actual competitive space** — making Home Affairs content accessible is the core value proposition.

**Key takeaway:** The Australian government's own tools are English-only and confusing. A Nepali-language explainer layer on top of official Australian processes is genuinely unoccupied territory.

---

## 3. Category II — Diaspora Fintech & Financial Services

### 3.1 Majority (USA) — The Community-Bank Model ⭐⭐⭐⭐

**Website:** https://www.getmajority.com
**What it is:** Migrant neobank launched by serving one specific community (Nigerian immigrants in Houston) before expanding community by community. $5.99/month membership.
**Funding:** ~$83.5M raised
**Target:** Migrants in the US (specific communities first, then expansion)

**What Saathi should copy:**
- **Start with ONE community deeply, then expand** — the most important go-to-market lesson for Saathi. Own the Nepalese-in-Sydney market before trying to own all diaspora communities
- **In-community distribution** — advisers and meet-up centres within the community (Saathi's FB group strategy maps directly to this)
- **Membership model** — $5.99/month as a signal that migrants will pay for community-specific value
- **Community word-of-mouth** as primary acquisition channel
- **Verified community advisers** as a trust layer (analogous to Saathi's "registered agent referral")

**What Saathi should NOT copy:**
- Banking/financial product complexity (regulatory overhead)
- US-specific financial products

**Relevance to Saathi:** The most important go-to-market template. The founder story (built for their own community) mirrors Saathi directly.

---

### 3.2 Bloom Money (UK) — The Diaspora ROSCA App ⭐⭐⭐⭐

**Website:** https://www.bloommoney.com
**What it is:** Digital "money circle" (ROSCA) app for African and Asian diaspora communities "straddling two countries." Started in the UK.
**Funding:** ~£1.5M pre-seed (modest), later opened investment to its own users
**Target:** African and Asian diaspora in the UK

**What Saathi should copy:**
- **Founder shape match** — solo-ish, culturally rooted, niche community focus. Not trying to be a unicorn
- **"Straddling two countries" product framing** — directly maps to Nepalese migrants straddling Nepal and Australia
- **Community-specific financial products** — building products FOR the community rather than for general market
- **Platform investment model** — opening investment to community members (could inspire Saathi's agent marketplace design)

**Relevance to Saathi:** The closest founder/market shape match. Bloom Money proves that small, culturally specific fintech for diaspora works even with modest funding.

---

### 3.3 Comun (USA) — Immigrant Neobank

**Website:** https://www.comun.life
**What it is:** Neobank focused on immigrant communities in the US. Raised ~$21.5M Series A.
**Target:** Latino and other immigrant communities in the US

**What Saathi should copy:**
- **Community-specific onboarding** — Spanish-first, culturally relevant UX
- **Bilingual support** — full Spanish + English (Saathi should be full Nepali + English)
- **Government benefit access** — helping immigrants access EITC, SNAP, etc. (analogous to Saathi helping navigate ATO, Medicare, Services Australia)

**Relevance to Saathi:** The bilingual government services navigation model maps directly to Saathi's Australian bureaucracy explainer.

---

### 3.4 Welcome Tech / SABEResPODER (USA) — In-Language Community Platform ⭐⭐⭐⭐

**Website:** https://saberespoder.com
**What it is:** Spanish-language community platform for Latino immigrants in the US. Started as an information resource and grew to paid services.
**Funding:** ~$70M total
**Target:** Latino immigrants in the US

**What Saathi should copy:**
- **"Trusted in-language information first, monetise adjacent services later"** — the exact Saathi arc
- **Health, financial, and educational services marketplace** (health → immigration is the same adjacency pattern as tax → immigration)
- **Earn money by sharing experience** — user-generated content model where community members contribute experiences in exchange for rewards (directly applicable to Saathi's community experiences section)
- **Knowledge center with educational content** — content hub model
- **Community events and workshops** — in-person trust building (Saathi could host Nepalese community events in Sydney)
- **Freemium membership** — free basic access, premium for discounted services

**What Saathi should NOT copy:**
- Health services marketplace (too regulated for early Saathi)
- The breadth of services (SABEResPODER is broad; Saathi should stay focused)

**Relevance to Saathi:** The product evolution arc (information → community → marketplace) is the playbook Saathi should follow. SABEResPODER proves this model is investable and scalable.

---

## 4. Category III — AI Document Reading & Explanation Tools

### 4.1 Claude / GPT-4V — Document Understanding (The Infrastructure Layer)

**What it is:** Underlying LLM vision + language capabilities that power document understanding.
**Relevance to Saathi:** These are the building blocks for Saathi's document reader (F2 in the MVP). The key challenge is:
- Nepali text extraction from documents (Devanagari script OCR quality)
- Structured extraction (what fields to pull from a visa grant letter)
- Citation grounding (extracting source text to cite)

**What Saathi should do:** Build a thin wrapper around Claude's vision API with:
- Custom prompt engineering for Australian immigration/ATO documents
- Nepali Devanagari handling
- Source-citation extraction pipeline
- Quality confidence scoring

---

### 4.2 Abe (USA) — AI Immigration Assistant

**Website:** https://abe.ai (referenced in immigration tech space)
**What it is:** AI-powered immigration assistant using conversational AI to answer immigration questions and guide users through processes.
**Target:** US immigrants

**What Saathi should copy:**
- **Conversational UI** — ask questions in natural language (directly applicable)
- **Process guidance** — not just Q&A but active guidance through steps
- **Handoff to attorney** — when the AI can't answer, route to a professional

**Relevance to Saathi:** The conversational model is the core UX pattern. The "explain → guide → handoff" flow is Saathi's product loop.

---

### 4.3 Document AI / Legal Document Readers (Generic)

Products like **FileNet**, **Kira Systems**, and **Luminance** use AI to review contracts and legal documents. Key lessons for Saathi:

- **Confidence scoring** — showing users when the AI is uncertain is critical in legal/immigration contexts
- **Highlight the relevant text** — when explaining a document, show the exact source text that supports the explanation
- **Version tracking** — documents change; showing "this explanation was based on version X, dated Y" builds trust
- **Human-in-the-loop** — for high-stakes extractions (e.g., extracting visa conditions from a grant letter), require human confirmation

---

## 5. Category IV — Community & Settlement Platforms

### 5.1 Meetaway (USA)

**What it is:** A platform connecting new migrants to local communities and settlement services. Uses matching algorithms to connect newcomers with established community members.
**Target:** New migrants in the US and Canada

**What Saathi should copy:**
- **Community matching** — connecting Nepalese students with established Nepalese migrants in Australia (mentorship model)
- **Local event discovery** — cultural events, community gatherings (could be integrated into Saathi's GTM)
- **Service provider directory** — vetted local service providers (migration agents, accountants, doctors who serve the Nepalese community)

---

### 5.2 Buddy HQ (Various)

**What it is:** Settlement platform for migrants in Australia and Canada. Offers housing, employment, and social integration services.
**Target:** New migrants in Australia/Canada

**What Saathi should copy:**
- **Housing guidance** — rental applications, bond, tenancy laws (a major pain point for Nepalese migrants in Australia)
- **Employment preparation** — resume localization, credential recognition (relevant to skilled migrant persona)
- **Banking and finance basics** — opening a bank account, TFN, superannuation explanation (directly overlaps with Saathi's P1 scope)

---

### 5.3 Reddit r/immigration / Facebook Groups (Community Intelligence)

**Platforms:** Reddit r/immigration, r/AusVisa, various Facebook groups (large Nepali-in-Australia groups)
**What they are:** Peer-to-peer immigration Q&A communities

**What Saathi should copy:**
- **Real question harvesting** — use these groups to find what people actually ask (the PRD already mentions this)
- **Community answer format** — how people explain things to each other in plain language (Saathi should match this register in Nepali)
- **Warning: unreliable advice** — the reason these groups exist is that nothing better does. This is Saathi's entry point.

**What Saathi should NOT copy:**
- The misinformation risk — Saathi's entire value prop is being more reliable than these groups

---

## 6. Category V — Government & Public Service Simplification

### 6.1 UK GOV.UK / Australia myGov (Reference)

These are the official portals Saathi must integrate with and explain. Key insight:
- **They are functional but not user-friendly** — extremely dense, bureaucratic, English-only
- **They are the source of truth** — Saathi's citations must point here
- **They have APIs** — some have machine-readable data (visa processing times, etc.) that Saathi should use directly

**myGov (Australia) specific:** Extremely low user satisfaction. The gap between what myGov offers and what users need is enormous. Saathi can fill this gap for the Nepalese community specifically.

---

## 7. Category VI — Professional Services Marketplaces

### 7.1 Zocdoc / Thumbtack / Lawyered — Marketplace Patterns

**Zocdoc** (healthcare): Find and book verified doctors, real-time availability, reviews. Lessons:
- **Verification as trust** — every professional is vetted (Saathi must verify MARN for migration agents)
- **Real-time availability** — valuable but hard to implement for immigration agents
- **Reviews from real patients** — community experiences feed into agent quality

**Thumbtack** (local services): Instant matches, quotes from multiple providers. Lessons:
- **Quote request flow** — Saathi's Connect C1 feature (quote request) maps directly here
- **Multi-agent comparison** — showing 2-3 agent options with pricing
- **Take-rate risk** — Thumbtack takes a cut, faces disintermediation. The PRD's warning about this is correct.

**Lawyered** (legal services marketplace): Connect with immigration lawyers. Directly relevant.

**What Saathi should copy from marketplace patterns:**
- **Verification-first** — MARN check at onboarding and on a schedule
- **Quote comparison UI** — simple, comparable quotes from verified agents
- **Case status visibility** — what Thumbtack doesn't do well (and what Saathi's Connect C4 should do well)
- **Escrow/hold** — payment only released when case milestones hit (prevents disintermediation)

---

## 8. Synthesis — Feature Blueprint for Saathi

### 8.1 Copy These Features (Direct Adoption)

| Feature | Source | Why It Fits Saathi |
|---------|--------|-------------------|
| Process explainer in plain language + citations | Boundless | Core product mechanic |
| Document checklist per process | Boundless, Immihelp | High value, low build |
| Professional handoff with disclaimer | Boundless, Abe | Regulatory necessity |
| Visa/process tracker | Immihelp | Deadline tracking MVP |
| Community experiences section | Immihelp | Trust building + content |
| "Grounded or silent" answering | Boundless | Critical for trust + legality |
| Community word-of-mouth GTM | Majority | Founder's unfair advantage |
| Timeline projection for PR pathway | Pathgrade | High demand for Nepalese migrants |
| Multilingual content hub | SABEResPODER, Immihelp | Knowledge center model |
| User-generated experiences with rewards | SABEResPODER | Community content flywheel |

### 8.2 Adapt These Features (Modify for Saathi)

| Feature | Source | Adaptation |
|---------|--------|-----------|
| Agent marketplace with quotes | Thumbtack, Zocdoc | Narrow to MARN-verified migration agents in AU |
| Subscription model | VisaCoach, Majority | Freemium: free 485 explainer, paid vault + multi-process |
| Conversational Q&A | Abe, Leverage Edu LE AI | Nepali-language, grounded, cited |
| Timeline projections | Pathgrade | Australian PR pathway wait times |
| Document reader | LLM vision + custom prompts | Nepali Devanagari + Australian immigration docs |
| Settlement services directory | Buddy HQ | Nepalese community-vetted agents, accountants, doctors |

### 8.3 Avoid These Features (Don't Copy)

| Feature | Source | Why Avoid |
|---------|----------------|---------|
| Form preparation/lodgement | Boundless | Regulated activity in AU without license |
| Medical/health services marketplace | SABEResPODER | Too regulated for MVP stage |
| Banking products | Majority, Comun | Heavy regulatory overhead |
| Broad immigration coverage | Immihelp | Too broad — stay niche on Nepalese in AU |
| General chatbot without grounding | ChatGPT | Dangerous for high-stakes immigration content |
| Commission-only model | Thumbtack | High disintermediation risk |

### 8.4 Unique Opportunities (Saathi-Specific, No Direct Competitor)

These are genuinely unoccupied spaces that Saathi could own:

1. **Nepali Devanagari document OCR + explanation** — no product does this for Australian immigration documents
2. **"Which visa can I get?" eligibility triage** — general info only, not advice, but genuinely helpful
3. **485 Graduate Visa explainer in Nepali** — this specific wedge is completely unoccupied
4. **Nepalese community-vetted Australian service providers** — a directory that the community trusts
5. **"Straddling two countries" financial explainer** — remittance, superannuation portability, Nepal-Australia tax
6. **Cultural integration explainer** — Australian workplace culture, rental norms, healthcare system — in Nepali

---

## 9. Market Gaps — Where No Product Exists

Based on this research, the following are genuinely unoccupied:

1. **Nepali-language Australian immigration explainer** — no product explains Australian visa processes in Nepali. This is the core wedge.
2. **Australian immigration AI companion with grounded citations** — generic LLMs (ChatGPT etc.) are dangerous here; a grounded, cited, Nepali-language AI is a step-change.
3. **Nepalese diaspora settlement app in Australia** — no product serves this community specifically in AU.
4. **Community-experiences immigration content for Nepalese in AU** — real stories from real Nepalese migrants about Australian immigration processes in Nepali.
5. **NRN (Non-Resident Nepali) financial integration tool** — explaining how Nepalese migrants can manage finances across Nepal and Australia.

---

## 10. Go-to-Market Validation from Research

The research validates the GTM strategy in the PRD:

1. **Community-first distribution works** — Majority, SABEResPODER, and Bloom Money all used community word-of-mouth as primary channel. The Nepali Facebook group strategy is correct.

2. **In-language trust layer is the moat** — SABEResPODER, Majority, and Comun all built in-language products that generic fintech/immigration products couldn't replicate. Saathi's Nepali-language, grounded, cited explainer is defensible.

3. **Professional handoff is the business model** — Boundless and Lawyered both show that the "AI handles information, humans handle regulated work" model is commercially viable and legally sound.

4. **Small community, deep ownership** — Majority proves you don't need to serve all immigrants to build a valuable business. Owning the Nepalese-in-Sydney market deeply is sufficient.

---

## 11. Sources & References

| Source | URL | Notes |
|--------|-----|-------|
| Boundless Immigration | https://www.boundless.com | Acquired by Payoneer Jan 2026 |
| Immihelp | https://www.immihelp.com | Long-standing US immigration portal |
| Leverage Edu | https://leverageedu.com | Edtech AI coach, TechCrunch Oct 2025 |
| Majority | https://www.getmajority.com | Migrant neobank, $83.5M raised |
| Bloom Money | https://www.bloommoney.com | ROSCA for diaspora, UK |
| SABEResPODER | https://saberespoder.com | Welcome Tech, $70M raised |
| Comun | https://www.comun.life | Immigrant neobank, $21.5M Series A |
| MoveBuddha | https://www.movebuddha.com | Comparison platform (moving, not immigration) |
| Abe AI | https://abe.ai | Immigration conversational AI |
| Immihelp multilingual | https://www.immihelp.com (Chinese, Spanish editions) | Multilingual model |
| Australian Home Affairs | https://www.immi.gov.au | Source of truth (blocked bot access) |
| myGov | https://my.gov.au | Official AU government portal (dense, English-only) |

---

## 12. Next Steps

1. **Validate the wedge** — 485 Graduate Visa transition is the most acute pain point (deadline pressure, high agent cost, no Nepali-language resource)
2. **Build the thin prototype** — grounded Q&A + document checklist + professional handoff, Nepali language, cited
3. **Harvest real questions** — spend 1-2 weeks in Nepali Facebook groups gathering actual questions people ask
4. **Talk to 3-5 migration agents** — validate referral appetite and willingness to pay for the Connect marketplace
5. **Build the knowledge base** — ingest Home Affairs, ATO, and Services Australia content for the 485 process

---

*Research compiled: June 28, 2026*
*For internal use only — decision support for Saathi product development*
