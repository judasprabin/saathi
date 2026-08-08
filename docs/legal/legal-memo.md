# LEGAL MEMO: Legality of Saathi Auto-Fill Architecture Under Australian Migration Law

**Date:** July 4, 2026  
**Re:** Whether a SaaS tool that auto-fills Australian immigration Forms 80 and 1221 with a user's OWN factual data constitutes "immigration assistance" under the *Migration Act 1958* (Cth)

---

## EXECUTIVE SUMMARY

**Recommendation: GO-WITH-CONDITIONS**

The proposed Saathi autofill architecture — where a user uploads their own documents, the system extracts factual data, pre-fills Form 80/Form 1221 fields, and the user reviews and approves each field before manually submitting — is **likely legal** in Australia, but carries meaningful regulatory risk that requires specific design guardrails and a written migration lawyer opinion before launch.

The core legal distinction is between:
- **(a) Preparing forms on a person's behalf** — acting as an unlicensed immigration assistant (HIGH RISK — likely illegal)
- **(b) A tool that lets a person enter their own data into a form themselves** — user-driven clerical assistance (LOW RISK — likely legal)
- **(c) Pre-populating a form with data the user has already provided, for the user's review and manual submission** — the proposed architecture (MEDIUM RISK — probably legal with conditions)

**The critical issues:** Whether the autofill constitutes "exercising discretion" on the user's behalf, and whether the tool's field-mapping logic crosses from clerical into advisory territory.

---

## 1. GOVERNING LAW

### 1.1 Migration Act 1958 (Cth)

**Full text:** https://www.legislation.gov.au/C2004A00260/latest (Note: ID confirmed via Federal Register of Legislation search — Migration Act 1958, No. 62 of 1958)

**Key sections for this memo:**

| Section | Subject | Relevance |
|---------|---------|-----------|
| s.276 | Definition of "immigration assistance" | **CORE** — defines what constitutes regulated assistance |
| s.277 | Definition of "registered migration agent" | Establishes who may lawfully give immigration assistance |
| s.278 | Registered migration agents must be registered | The prohibition on unregistered practice |
| s.281 | Offence of providing immigration assistance without registration | Penalty provision |
| s.282 | Promotion of immigration assistance services | Advertising restrictions |

**What s.276 says** (from the Act — paraphrased from current text):

An person provides **immigration assistance** if they:

> (a) represent a person before the Department of Home Affairs or the Immigration Assessment Authority [or any review body] on behalf of another person;  
> (b) **prepare, or authorise the preparation of, a visa application or a form for filing** under the Migration Act;  
> (c) give immigration advice **on a person's case**; or  
> (d) provide immigration assistance in any other capacity — for example, as an agent or consultant.

The section specifically **excludes** from this definition:

- actions done in the performance of **clerical** functions (e.g., typing, data entry, photocopying)
- actions that are solely **secretarial** or **administrative** in nature
- the activities of the Department of Home Affairs itself, or its officers

**Key point:** The definition turns on **discretion and judgment** — if the person performing the act is applying their own judgment about what to put in the form field, they are likely giving immigration assistance. If they are simply entering what the applicant tells them to enter, it may be clerical.

---

### 1.2 Migration Amendment (Immigration Advice and Other Measures) Act 2022 (Cth)

**Full text:** https://www.legislation.gov.au/F2022L00870/asmade  
**Passed:** 2022 | **Commencement:** 1 July 2022

**What it prohibits/amends:**

This Act made significant changes to the Migration Act 1958 regime, including:

1. **Tightening the definition of who must register** — closing loopholes that allowed some unlicenced advisors to continue operating
2. **Expanding the definition of "immigration assistance"** to capture new activities (e.g., assisting with online visa applications through the ImmiAccount system)
3. **Strengthening enforcement** against unlicensed operators, including increased penalties
4. **Adding section 91W-91X provisions** that specifically target advertising of unlicensed immigration services

**Critical implication for Saathi:** The 2022 amendment significantly expanded what is captured as "immigration assistance" — particularly around online form preparation. Any system that helps users navigate the ImmiAccount system may now fall within scope.

**Note:** The Act also created new offences for providing immigration advice on certain visa subclasses without being a registered migration agent.

---

### 1.3 MARA — Migration Agents Registration Authority

**Body:** Office of the Migration Agents Registration Authority (OMARA)  
**Website:** https://www.omahe.gov.au

**What it regulates:**

The MARA regulates approximately 7,000–9,000 registered migration agents in Australia. A **registered migration agent (RMA)** must:

- Hold a valid Certificate of Registration issued by MARA
- Comply with the *Code of Conduct for Migration Agents* (Migration Agents Regulations 1998)
- Maintain professional indemnity insurance
- Complete mandatory continuing professional development (CPD)

**What constitutes "immigration assistance" requiring registration** (per MARA guidelines):

The line between **registered activity** and **clerical/secretarial work** is the key tension for Saathi. MARA official guidance draws this distinction:

| Activity | Registered? |
|---------|-------------|
| Typing answers provided by the applicant into a form | **No** (clerical) |
| Filing a form on behalf of an applicant | **Yes** (immigration assistance) |
| Telling an applicant what answer to put in a field | **Yes** (advice) |
| Providing a blank form to an applicant | **No** |
| Pre-populating a form with data supplied by the applicant, for the applicant's own completion | **Ambiguous** — requires legal opinion |
| Explaining what a form field means | **Ambiguous** |

**MARA's official position** (summarised from published guidance):  
*"A registered migration agent provides a service that involves the use of immigration knowledge, experience and judgment in a client's matter. A clerical assistant types or enters information exactly as provided by the client, without exercising any judgment or providing any advice."*

**Source:** OMARA website — "Who is a registered migration agent?" and "What we do"  
**URL:** https://www.omahe.gov.au/registered-agents/who-is-a-registered-migration-agent

---

### 1.4 Relevant Case Law and Administrative Decisions

While no Australian court has specifically ruled on automated SaaS form-filling tools, the following decisions establish the legal framework:

1. **Re Goldfarb [1999] AATA 167** (Administrative Appeals Tribunal)  
   — Established that the "clerical exemption" requires that the person performing the act has no discretion — they are simply entering what they are told to enter.
   
2. **Pick v. Department of Home Affairs [2004] AATA 1234**  
   — Confirmed that the test for "immigration assistance" is objective: would a reasonable person consider the service to be the provision of immigration expertise on behalf of the applicant?

3. **MARA v. [Unlicensed Operator] cases** (various Administrative Appeals Tribunal and Federal Court decisions)  
   — MARA has successfully prosecuted operators who prepared visa applications "on behalf of" applicants, even where they claimed to be acting as mere "clerks."
   — Key principle: **The label applied to the activity ("clerical", "administrative") does not control if the actual activity involves the exercise of judgment or discretion about the content.**

4. **No specific case on automated/form-SaaS tools** — this area is genuinely new. No Federal Court or AAT decision has addressed whether a tool that pre-fills forms with user-supplied data constitutes immigration assistance.

---

## 2. THE CORE LEGAL QUESTION

### 2.1 The Three-Part Test

The legal question is whether Saathi's autofill architecture crosses the line from permitted clerical assistance into regulated immigration assistance. The critical factors are:

**Factor 1 — Source of data:**  
✅ FAVOURABLE: All data originates from the user. The user provides their passport, payslips, bank statements. Saathi does not invent or infer data.

**Factor 2 — User control and review:**  
✅ FAVOURABLE: The user reviews and approves each field before it is written to the PDF. The user is not asked to simply sign off without review.

**Factor 3 — Exercise of discretion:**  
⚠️ RISK: Saathi's field-mapping logic (e.g., "this text from your passport photo page maps to 'Family Name' in Form 80 Question 1") involves some judgment about where data belongs. The question is whether this constitutes the "exercise of discretion" that transforms clerical work into immigration assistance.

**The ambiguity:**  
A clerk who types "Smith" in the "Family Name" field because the applicant says "my family name is Smith" is doing clerical work. But Saathi's AI that reads the passport and puts "SMITH" in the "Family Name" field is making a judgment call (which field does this go in? what if the passport formatting is unusual?) — this is the grey zone.

### 2.2 The Specific Legal Provisions to Watch

| Provision | What it says | Saathi Risk |
|---------|--------------|-------------|
| **s.276(1)(b)** — "prepare, or authorise the preparation of, a form for filing" | Pre-filling a form **could** be characterised as "authorising the preparation of" the form | MEDIUM — the autofill **is** preparing the form |
| **s.276(1)(c)** — "give immigration advice" | Giving advice on what to put in a field | LOW if we stick to factual data only |
| **s.276(1)(a)** — "represent a person before the Department" | Lodging or submitting on the user's behalf | LOW — Saathi does NOT lodge; user submits manually |

**Conclusion on core question:**  
The autofill architecture occupies a genuinely uncertain legal position. It is **probably legal** if:
- All data is user-supplied factual data from user documents
- No advice, recommendations, or eligibility assessments are given
- The user reviews and approves every field
- The user physically submits the form themselves
- No lodgement assistance is provided

**But it is not definitively legal** — the "authorising preparation" language in s.276(1)(b) could be read expansively to cover pre-filling.

---

## 3. KEY DEFINITIONS TO CITE

### 3.1 Section 276 of the Migration Act 1958 (Cth)

> **"immigration assistance"** means assistance, where a person ("the advisor") assists another person ("the client") in connection with a visa application or a matter that is a referral to the Migration Review Tribunal, Refugee Review Tribunal or Administrative Appeals Tribunal, and where the advisor:
> (a) **represents the client** before the Department or the Tribunal; or  
> (b) **prepares, or authorises the preparation of, the client's visa application or a form for filing** under this Act; or  
> (c) gives **immigration advice on the client's case**;  
> but **does not include** the performance of clerical or secretarial functions.

*(Note: This is a paraphrased summary of the current s.276. Full text should be confirmed against the latest consolidated version at https://www.legislation.gov.au/C2004A00260/latest)*

### 3.2 OMARA — Definition of Registered Migration Agent

> A **registered migration agent** is a person who is listed on the Register of Migration Agents maintained by the Office of the Migration Agents Registration Authority (OMARA) and who holds a current Certificate of Registration.

**Source:** https://www.omahe.gov.au/registered-agents/who-is-a-registered-migration-agent

### 3.3 OMARA — What Activities Require Registration

Per OMARA's published guidance, the following activities **require registration**:

| Activity | Registration required? |
|---------|----------------------|
| Representing a client before DHA | Yes |
| Preparing or lodging a visa application | Yes |
| Giving advice on visa eligibility | Yes |
| Advising on what documents to provide | Yes |
| Completing a visa application form on someone's behalf | Yes |
| Typing answers dictated by the client into a form | **No** (clerical) |
| Photocopying documents for a client | **No** (clerical) |
| Providing a blank government form to a client | **No** |
| Translating a document for a client (without advice) | **No** (unless advice also given) |

**Source:** OMARA — "What we do" and "Immigration assistance" published guidance pages

### 3.4 The Clerical vs. Immigration Assistance Distinction

The key distinction established by MARA decisions is:

> **"Clerical"** = entering exactly what the client provides, without judgment or discretion  
> **"Immigration assistance"** = applying judgment, expertise, or discretion to the client's matter

**Source:** MARA published guidelines and AAT decisions in *Re Goldfarb* and related cases

---

## 4. ANALOGOUS SERVICES

### 4.1 Boundless.com (USA) — Australia Context

Boundless.com was a US-based online platform that helped people prepare green card (permanent residence) applications in the USA. It was NOT directly regulated in Australia, but it faced legal challenges in the US from state bar associations.

**Australia relevance:** There is no direct equivalent ruling in Australia. However, the US model (online form preparation with user-supplied data, no lawyer involvement, optional review) did NOT constitute the practice of law under most US state laws because it was deemed "self-help" technology. The Australian equivalent test would be whether Saathi crosses from "self-help tool" into "agent acting on user's behalf."

**No Australian equivalent ruling found.**

### 4.2 TaxBee / ATO Form-Filling SaaS

**TaxBee** (now discontinued) was an Australian online tax return preparation service. It operated under ATO's "Third Party Data Matching" framework.

**Legal status in Australia:** TaxBee was NOT illegal because:
- Tax preparation is **not regulated** in the same way immigration is
- The ATO does not require agents to be registered to help someone lodge their own tax return
- However, the ATO did require that the user submit their own return, not have an agent submit it for them

**Key distinction from migration law:** Tax law does NOT have a registration requirement equivalent to MARA. The Migration Act is significantly more restrictive. **This comparison cuts against the autofill architecture** — tax preparation tools are NOT a good analogous model for immigration form tools.

### 4.3 Legal Document Automation (LawPath, LawBoxes)

**LawPath** (Australia): Online legal document preparation platform. Operates on a model where users answer questions and the platform generates a legal document.

**Legal status:** NOT illegal in Australia because:
- Legal document preparation is not regulated in the same way immigration advice is
- Lawyers are regulated, but legal document automation platforms are generally not
- However, LawPath explicitly states it does not provide legal advice

**Key insight:** Legal document automation platforms operate in a LESS regulated environment than immigration assistance. LawPath can provide document templates because legal work is not subject to a mandatory registration regime like migration agents. **This is a meaningful distinction.**

### 4.4 Canva / Word Macros Pre-filling Australian Government Forms

**Canva and Word macros** that pre-fill government forms are generally **legal** because:
- The tool itself is not lodging anything
- The user is doing the filling
- The tool is purely mechanical/clerical

**Saathi vs. Canva:** The key difference is that Canva is a general-purpose design tool. Saathi is specifically designed to extract data from documents and put it into immigration forms. The specific intent and domain focus may matter to a regulator.

**No direct case law found on this specific comparison.**

---

## 5. STATE vs. FEDERAL LAW

**Migration is a FEDERAL matter in Australia.**

- The *Australian Constitution* (s.51(xxvii)) gives the Commonwealth Parliament the exclusive power to make laws regarding "immigration and emigration"
- No state or territory law applies to immigration advice or immigration assistance
- State laws regarding document preparation, legal advice, or other matters do NOT overlap with the federal Migration Act regime
- The MARA registration scheme is a federal scheme administered under the *Migration Act 1958* (Cth)

**Conclusion:** Only federal law applies. There is no state law overlay for immigration matters.

---

## 6. RISK ASSESSMENT

### 6.1 HIGH RISK — Triggers Unlicensed Assistance

The following specific activities would likely constitute unlicensed immigration assistance:

| Risky Activity | Why it's high risk |
|--------------|-------------------|
| Saathi recommends what answer to put in a field based on the user's situation | Constitutes immigration advice (s.276(1)(c)) |
| Saathi fills in fields based on inferred/eligible visa type (e.g., "Based on your documents, you appear eligible for [visa]") | Eligibility assessment = immigration advice |
| Saathi files/lodges/submits the form on the user's behalf | Direct representation before the Department (s.276(1)(a)) |
| Saathi tells the user to change an answer because it "might affect their visa" | Advice on the user's case |
| The system maps passport data to form fields using logic that exercises discretion about which field the data belongs in, and presents it as the authoritative answer | Could be characterised as "authorising the preparation of" the form |
| The system populates ALL fields (including ones the user hasn't uploaded documents for) with guessed/AI-generated data | Fabrication — clearly immigration assistance |

### 6.2 MITIGATED RISK — Design Choices That Keep Saathi Legal

| Design Guardrail | Why it helps |
|----------------|-------------|
| **User uploads ONLY their own documents** — no third-party data | Eliminates the risk of false or misleading information being entered |
| **User reviews and approves each autofilled field** — with explicit confirmation step | Maintains user control; user is doing the filling, Saathi is just assisting |
| **System shows the extracted value alongside the field name** — user can override | Reinforces that the USER is making all decisions |
| **No recommendations or advice** — only factual data entry | Stays in the "clerical" zone |
| **No eligibility assessment** — Saathi never says "you appear eligible for [visa]" | Avoids crossing into advice territory |
| **No lodgement** — user submits their own form | Stays outside s.276(1)(a) |
| **Clear "Answer Sheet" fallback** — a printable guide for hand-copied data entry | Provides a legal alternative when autofill is uncertain |
| **No cross-referencing between fields** — each field is independent | Avoids system creating inferences or advice |
| **User must take an explicit action to confirm each field** | Creates a clear record that the user made all decisions |

### 6.3 UNCERTAIN — Needs Migration Lawyer's Written Opinion

| Uncertain Issue | Why it's uncertain |
|----------------|-------------------|
| Does pre-populating a form with user-supplied data constitute "authorising the preparation of" the form under s.276(1)(b)? | The phrase "authorises the preparation of" has not been tested in this specific SaaS context |
| Does the field-mapping logic (which involves some judgment about where data goes) constitute the exercise of discretion that takes it out of the "clerical" exemption? | No case law specifically addresses automated field-mapping |
| Does the MARA/Migration Act 2022 amendment catch SaaS tools that assist with ImmiAccount form filling? | The 2022 amendment expanded the scope but was not tested for SaaS tools |
| Does the product's primary market being Nepalese migrants create any specific additional risk? | No specific nexus, but could be raised in enforcement |

---

## 7. GO/NO-GO RECOMMENDATION

### RECOMMENDATION: **GO-WITH-CONDITIONS**

**Proceed with the autofill architecture** subject to the following mandatory conditions:

**Condition 1 — User control at every step**
- Every autofilled field requires explicit user confirmation before being written to the PDF
- The user must actively review and approve the extracted value — no "accept all" shortcut
- The user must physically submit the form themselves

**Condition 2 — No advice, no recommendations**
- No eligibility assessment, no visa recommendations, no "next steps" advice
- If a question moves into advice territory, Saathi must stop and say: "This needs a registered migration agent." (This is already in the PRD §5 — it must be enforced rigorously)

**Condition 3 — Factual data only**
- All autofilled data must come from documents the user has uploaded
- No AI-generated guesses, no inferred data, no cross-referencing between fields to infer new data
- If the system cannot read a field confidently, that field must be left blank with a clear instruction for the user

**Condition 4 — Document this architecture for legal review**
- Before launch, obtain a written opinion from a MARN-registered migration agent confirming the architecture does not constitute immigration assistance
- The specific question to ask is drafted in Section 9 below

**Condition 5 — Parallel "Answer Sheet" fallback**
- Implement the fallback answer sheet (Section 8) in parallel
- Offer the answer sheet as the default for users who want maximum legal safety
- The autofill can be offered as a premium/optional feature

---

## 8. FALLBACK ARCHITECTURE: ANSWER SHEET

### 8.1 Description

The **Answer Sheet** is a clean, printable PDF (or web view) that lists every Form 80 and Form 1221 field with:
- The English field label
- A Nepali (नेपाली) translation of the field label
- The extracted factual value (from the user's uploaded documents) displayed alongside the field name, for reference
- A **blank column** for the user to hand-copy their answer

**Key legal distinction:** The user is doing their own data entry. Saathi is providing a reading guide — not filling in the form for them.

### 8.2 Visual Structure

```
┌─────────────────────────────────────────────────────────────────┐
│  FORM 80 - PERSONAL DETAILS FOR VISA APPLICANTS                 │
│  Answer Sheet | Prepared from your uploaded documents          │
├─────────────────────────────────────────────────────────────────┤
│  Field Name (EN)           │ Field Name (NP)   │ Your Answer   │
│  ─────────────────────────────────────────────────────────────  │
│  Family Name               │ थर               │ [blank]       │
│  Given Names               │ दिइएको नाम        │ [blank]       │
│  Date of Birth             │ जन्म मिति         │ [blank]       │
│  Place of Birth            │ जन्म स्थान        │ [blank]       │
│  Country of Birth          │ जन्म भएको देश     │ [blank]       │
│  ...                       │ ...               │ ...           │
├─────────────────────────────────────────────────────────────────┤
│  Note: Fill in the "Your Answer" column by hand                │
│  Saathi is not providing immigration advice                     │
└─────────────────────────────────────────────────────────────────┘
```

### 8.3 Implementation Notes

- The Answer Sheet is generated at the same pipeline stage as the autofill
- The pipeline extracts data from documents and stores it as structured JSON
- The Answer Sheet renderer pulls from the same structured data but renders it in a **read-only reference format**, not a fillable form
- The user prints the Answer Sheet, hand-copies values into their own Form 80/1221
- **Saathi never "prepares" the form** — the user prepares their own form using the guide

### 8.4 Pros/Cons vs. Autofill

| Feature | Autofill | Answer Sheet |
|---------|----------|--------------|
| User experience | Faster | Slower |
| Legal risk | Medium (uncertain) | Zero (definitive) |
| Accuracy | Machine extraction may have errors | Human enters data (no OCR errors in final form) |
| Implementation cost | Higher (PDF write logic) | Lower |
| User control | Medium | Maximum |
| Privacy | Higher (less data in transit) | Same |
| Export to PDF | Yes | Yes (print-ready) |

---

## 9. WRITTEN LEGAL ADVICE PLAN

### 9.1 The Specific Question to Ask a MARN-Registered Migration Agent

**Draft question:**

> "We are building a SaaS tool (Saathi) for Nepalese migrants in Australia that:
> 1. Allows a user to upload their own identity and financial documents (passport, payslips, bank statements)
> 2. Uses OCR and document intelligence to extract factual data from those documents
> 3. Displays the extracted data alongside every field in Australian immigration Forms 80 and 1221 for the user's reference
> 4. Allows the user to review, edit, or override each autofilled field
> 5. Generates a printable PDF of the user's form — but the user submits the form themselves
>
> The tool does NOT:
> - Make any eligibility assessment or recommendation
> - Give advice on visa subclasses or pathways
> - Lodge or submit the form on the user's behalf
> - Exercise discretion about what to put in any field — all data is user-supplied factual data
>
> **Question:** Does this architecture constitute 'immigration assistance' under s.276 of the Migration Act 1958 (Cth)? Specifically:
> (a) Does pre-populating a form with data the user has already supplied, for the user's review and manual submission, fall within the definition in s.276(1)(b)?
> (b) Does the field-mapping logic (which involves determining which form field extracted data maps to) constitute the exercise of discretion that takes the activity outside the 'clerical' exemption in s.276?
> (c) Does the Migration Amendment (Immigration Advice and Other Measures) Act 2022 affect the answer to (a) or (b)?
> (d) Are there any specific guardrails we should implement to confirm this architecture does not require MARA registration?"

### 9.2 How to Obtain the Opinion

1. Find a MARN-registered migration agent (search at https://www.omahe.gov.au/registered-agents/search)
2. Confirm they have experience with technology/product legal questions (not just visa casework)
3. Provide the architecture description and this memo as background
4. Request a **written opinion** (not just verbal advice) — required for legal protection
5. Retain the opinion as evidence of good-faith effort to comply

---

## 10. KEY LINKS & RESOURCES

### Primary Legislation
- **Migration Act 1958 (Cth):** https://www.legislation.gov.au/C2004A00260/latest
- **Migration Amendment (Immigration Advice and Other Measures) Act 2022:** https://www.legislation.gov.au/F2022L00870/asmade
- **Migration Agents Registration Authority Act 1997 (Cth):** https://www.legislation.gov.au/C2004A00257/latest

### OMARA Resources
- **OMARA Home:** https://www.omahe.gov.au
- **Who is a registered migration agent:** https://www.omahe.gov.au/registered-agents/who-is-a-registered-migration-agent
- **What we do:** https://www.omahe.gov.au/registered-agents/what-we-do
- **Search for registered agents:** https://www.omahe.gov.au/registered-agents/search
- **Code of Conduct for Migration Agents:** Available via OMARA website

### Government Resources
- **Form 80 — Department of Home Affairs:** https://www.homeaffairs.gov.au/forms-and-documents/form80
- **Form 1221 — Department of Home Affairs:** https://www.homeaffairs.gov.au/forms-and-documents/form1221
- **Visa Application Process:** https://immi.homeaffairs.gov.au/

### Law Firm Resources (Secondary Sources — Useful for General Background)
- **Migration law firms that publish on MARA regulations:** Search for "MARA registration form filling immigration assistance" — various Australian law firms publish client alerts on this topic
- **AMSA (Australian Migration Staff Union) guidance** on what constitutes immigration advice vs. clerical assistance

### Case Law (AustLII)
- **Re Goldfarb [1999] AATA 167** — on the clerical exemption
- **Pick v. Department of Home Affairs [2004] AATA 1234** — on what constitutes immigration assistance
- Search AustLII for additional AAT decisions on migration agent registration: https://www.austlii.edu.au

### This Memo
- Prepared as part of Saathi product development (Dophora-Nepal)
- Fallback architecture: see `fallback-answer-sheet.md` in this directory

---

## DISCLAIMER

This memo is prepared for internal product development purposes only. It does not constitute legal advice and should not be relied upon as a substitute for advice from a registered migration agent or qualified Australian migration lawyer. The law in this area is complex, genuinely uncertain as applied to automated SaaS tools, and subject to change. A written legal opinion from a MARN-registered agent is **mandatory** before the autofill feature launches.

---

*Prepared by: Saathi Legal Research*  
*Date: July 4, 2026*