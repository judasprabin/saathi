# Saathi — Document Scan & Visa Form-Fill Architecture

**Project:** Saathi — AI Settlement Companion for Nepalese Migrants in Australia  
**Date:** July 4, 2026  
**Status:** Architecture draft — feature spec for F4 (Form Helper) doc-scan module  
**Author:** Prabin Karki  

---

## 1. Overview

This document specifies the architecture for Saathi's **Document Scan & Form Fill** capability — the technical pipeline that takes uploaded identity and financial documents (passport, payslips, bank statements, visa grant letters, PDFs, photos) and fills a target Australian immigration form (e.g. Form 80, Form 1221, 485 online sections) with extracted values.

**In-scope:** Upload → Classify → Extract key-value pairs → Validate → Map to form fields → Fill PDF → Human review  
**Out-of-scope:** Form lodgement (ImmiAccount integration), legal advice, eligibility assessment, automated submission

---

## 2. Document Types to Support

| Document | Key Fields to Extract | Format |
|----------|----------------------|--------|
| Passport | Full name, DOB, passport number, nationality, expiry, MRZ | PDF scan / photo |
| Visa Grant Notice | Visa subclass, grant date, expiry date, conditions | PDF (ImmiAccount email attachment) |
| Payslip (1–3 months) | Employer name, ABN, gross pay, pay period, employee name | PDF scan / photo |
| Bank Statement | Account holder, BSB, account number, transaction summaries | PDF |
| Birth Certificate | Full name, DOB, place of birth, parents' names | PDF scan / photo |
| Academic Transcript | Name, institution, degree, completion date | PDF |
| English Test Result (IELTS/PTE) | Test date, overall score, band scores per section | PDF scan / photo |

---

## 3. Stage-by-Stage Option Comparison

### 3.1 Upload & Classification

**Option A — Client-side pre-classification**  
User selects document type before upload. Simple, zero server cost, but introduces user friction and misclassification.

**Option B — Server-side classifier (ML model)**  
A lightweight image classifier (e.g. ResNet/EfficientNet fine-tuned on document types) runs on upload and suggests a type. More robust, enables automatic routing.

**Option C — LLM-based classification (Anthropic Claude)**  
Send a low-resolution thumbnail with a classification prompt. Handles ambiguous cases (e.g. "is this a payslip or a tax summary?"). Most flexible but cost/latency.

**Recommendation:** **Option C (Claude)** for MVP — simplest to build, handles mixed documents (e.g. photo of a passport open to the bio page), and the per-call cost is negligible at MVP scale (~0.1¢/image). Promote to Option B when scale demands it.

---

### 3.2 OCR / Vision (Text Extraction)

| Option | Nepali (Devanagari) | Tables/Layout | Confidence | Cost | Notes |
|--------|---------------------|---------------|------------|------|-------|
| **Claude Vision** (Anthropic) | ✅ Excellent — natively supports Devanagari | ✅ Good — understands layout, columns | ✅ Per-token confidence + bounding boxes | Pay-per-call | **Recommended.** Best-in-class multilingual; no training required |
| AWS Textract | ⚠️ Partial — separate OCR for Devanagari | ✅ Very good — forms/tables | ✅ Built-in confidence scores | Pay-per-page + storage | Strong for structured forms (1098s, tax docs); adds cost and AWS dependency |
| Google Document AI | ✅ Good — supports Devanagari | ✅ Excellent — specialized parsers for many form types | ✅ Built-in | Pay-per-page | Excellent for standardized forms; fewer language options than Claude |
| Tesseract 5 (open) | ⚠️ Fair — requires `--lspnep` model; quality varies | ❌ Poor — no layout understanding | ❌ No per-word confidence in default config | Free | Too brittle for MVP; useful as fallback only |
| Azure AI Document Intelligence | ⚠️ Partial — Devanagari support is limited | ✅ Good | ✅ Good | Pay-per-page | Not recommended for Nepali-first documents |

**Recommendation:** **Claude Vision** — best Nepali support of any commercial model, no training/infrastructure, native bounding boxes enable structured field mapping, confidence scoring via token probabilities.

---

### 3.3 Key-Value Extraction

**Option A — Schema-driven (rule-based + Claude)**  
Define explicit field schemas per document type. Claude receives the image + schema + extraction prompt and returns structured JSON. Strict, predictable, auditable.

**Option B — Open extraction (Claude with no schema)**  
Claude returns all readable name-value pairs with no prescribed schema. More flexible but requires post-processing to map to form fields.

**Option C — Fine-tuned extraction model**  
Train a custom NER/extraction model on annotated document images. High accuracy once trained; significant upfront annotation cost; language-specific.

**Recommendation:** **Option A (Schema-driven)** for structured fields (passport number, DOB, etc.) + **Option B (Open)** as a secondary pass to catch unexpected fields. The schema drives the happy path; open extraction handles edge cases and surfaces additional data for the human reviewer.

---

### 3.4 PDF Fill Methods

**Option A — AcroForm field population (pdf-lib / iText)**  
The target PDF has pre-existing form fields (standard for Australian immigration forms). Write values directly to field names using a PDF library. Lossless, reversible, no visual layout changes.

**Option B — PDF overlay (pdftk / reportlab)**  
Render extracted values as positioned text/image on top of the original PDF. Works on any PDF (no fields needed). Risks misalignment, text overflow, and layer conflicts.

**Option C — Flatten + image injection**  
Flatten the form into an image and overlay text. Clean visual output but destroys form field structure — cannot be edited by a migration agent later.

**Option D — pdfrw + reportlab for dynamic PDFs**  
Python-native AcroForm manipulation. Good for dynamically generating PDFs.

**Recommendation:** **Option A (AcroForm)** as primary. Australian immigration forms (Form 80, 1221) use AcroForm fields. Verify field names by opening the PDF in Adobe Acrobat before building. Fall back to Option B only if the target PDF has no form fields.

---

### 3.5 Validation & Confidence Handling

**Critical design rule:** Low-confidence fields **never auto-fill**. They are surfaced to the user for confirmation.

| Confidence Level | Action |
|-----------------|--------|
| ≥ 85% (high) | Pre-fill field; green highlight; user can override |
| 50–84% (medium) | Pre-fill field; amber highlight; explicit user confirm required |
| < 50% (low) | Blank field; red highlight; user must enter manually |
| Extraction failed entirely | Blank + tooltip: "Could not read this field. Please enter manually." |

Claude Vision does not expose per-token confidence scores directly, so confidence is inferred from:
- **Bounding box stability** — multiple coherent reads of the same region
- **Cross-check** — extract the same field from two independent Claude calls; if both agree at ≥85% text match, raise confidence
- **MRZ checksum** — passport numbers validated against ISO 7064 MOD 7/11 algorithms
- **Date plausibility** — extracted dates checked against document type norms (e.g. passport expiry must be >5 years from issue for Australian visa)

---

## 4. Recommended Pipeline

```
Upload → Classify → Extract (Schema) → Extract (Open) → Merge → Validate → 
  [HIGH] ──────────────────→ Auto-fill ──────────────────→ User review ──→ Export
  [MEDIUM] ─────────→ Amber confirm ──────────────────────────────────→ User review ──→ Export
  [LOW/FAIL] ─────→ Red manual entry ──────────────────────────────────→ User review ──→ Export
```

### Pipeline Stage Detail

```
┌─────────────────────────────────────────────────────────────────────┐
│  1. UPLOAD                                                          │
│     User selects file(s) — max 10, max 20MB each                    │
│     Supported: PDF, PNG, JPG, HEIC                                  │
│     Client-side: virus scan (ClamAV via API), file type validation  │
└──────────────────────────┬──────────────────────────────────────────┘
                           ▼
┌─────────────────────────────────────────────────────────────────────┐
│  2. CLASSIFY (Claude Vision — classification prompt)                │
│     Output: { doc_type: "passport" | "payslip" | "bank_statement"  │
│                           | "visa_grant" | "birth_certificate" |    │
│                             "academic_transcript" | "english_test"  │
│                           | "unknown" }                             │
│     Confidence score attached                                        │
└──────────────────────────┬──────────────────────────────────────────┘
                           ▼
┌─────────────────────────────────────────────────────────────────────┐
│  3. SCHEMA EXTRACTION (Claude Vision — per document-type schema)     │
│     Passport schema: name, dob, passport_no, nationality, expiry,    │
│                      issuing_country, mrz_text                       │
│     Payslip schema: employer_name, abn, gross, pay_frequency,        │
│                     period_start, period_end, employee_name           │
│     ... per document type                                            │
│     Output: structured JSON + per-field confidence estimate          │
└──────────────────────────┬──────────────────────────────────────────┘
                           ▼
┌─────────────────────────────────────────────────────────────────────┐
│  4. OPEN EXTRACTION (Claude Vision — no schema)                      │
│     Second pass: extract ALL readable name-value pairs              │
│     Output: flat JSON of all fields found                            │
└──────────────────────────┬──────────────────────────────────────────┘
                           ▼
┌─────────────────────────────────────────────────────────────────────┐
│  5. MERGE & DEDUP                                                   │
│     Combine schema + open extraction; prefer schema value on conflict│
│     Cross-validate where possible (e.g. name from passport vs. name  │
│     from payslip must be consistent — flag mismatch for review)      │
└──────────────────────────┬──────────────────────────────────────────┘
                           ▼
┌─────────────────────────────────────────────────────────────────────┐
│  6. VALIDATE                                                        │
│     MRZ checksum, date plausibility, ABN format, BSB format        │
│     Cross-document consistency checks                               │
│     Output: per-field confidence tier (HIGH / MEDIUM / LOW / FAIL)  │
└──────────────────────────┬──────────────────────────────────────────┘
                           ▼
┌─────────────────────────────────────────────────────────────────────┐
│  7. MAP TO FORM FIELDS                                              │
│     Source field → Target form field mapping table (per form type)   │
│     e.g. "passport.full_name" → "Form80.AT01_GivenNames"           │
│     Only HIGH-confidence mappings auto-fill; MEDIUM/LOW go to review │
└──────────────────────────┬──────────────────────────────────────────┘
                           ▼
┌─────────────────────────────────────────────────────────────────────┐
│  8. HUMAN REVIEW UI                                                 │
│     Side-by-side: extracted value (left) ↔ source image snippet (right)
│     User confirms, corrects, or enters manually per field           │
│     Bilingual labels: field name in English + Nepali explanation    │
│     "Was this extraction correct?" — one-click confirm per field    │
└──────────────────────────┬──────────────────────────────────────────┘
                           ▼
┌─────────────────────────────────────────────────────────────────────┐
│  9. PDF FILL (AcroForm via pdf-lib)                                 │
│     Write confirmed values to pre-named form fields                 │
│     Field name mapping sourced from field manifest (see §7.1)       │
│     Generate annotated PDF (highlighted fields, audit trail)        │
└──────────────────────────┬──────────────────────────────────────────┘
                           ▼
┌─────────────────────────────────────────────────────────────────────┐
│  10. EXPORT & AUDIT LOG                                             │
│      Download filled PDF                                            │
│      Store: source docs + extracted JSON + reviewer actions +       │
│             filled PDF (all in Supabase, user-scoped bucket)        │
│      Audit log entry: who, what, when, confidence tier              │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 5. Bilingual (English / Nepali) Handling

### 5.1 Document Language

Australian immigration documents are English-language only. Passport MRZ, payslips, bank statements are all in English. **Nepali text does not appear in source documents.**

The Nepali layer applies to:
1. **Field labels** in the review UI (e.g. "Passport Number / पासपोर्ट नम्बर")
2. **Explanation text** for what each immigration form field means
3. **Tooltip text** for unfamiliar fields (e.g. "ABN — Australian Business Number")
4. **Confirmation prompts** ("Does this expiry date look correct? / यो म्याद सही छ?")

### 5.2 Nepali in Review UI

- All UI labels are bilingual (EN primary / NP secondary)
- Extraction confidence messages shown in both languages
- User can toggle NP-only mode in settings

### 5.3 Devanagari OCR

If a user uploads a Nepali-language document (e.g. Nepali birth certificate), Claude Vision handles Devanagari natively. The extraction schema remains the same; the output values are in Devanagari, which is stored as-is in Supabase and rendered in the review UI with a Devanagari-to-Latin transliteration suggestion.

---

## 6. Form Field Manifest (Per-Form)

Before building any form fill, produce a **field manifest** by opening the target PDF in Adobe Acrobat, listing every form field: name, type, default value, allowed characters, max length. This becomes the source-of-truth mapping table.

| Form | Fields to Map | AcroForm Status |
|------|--------------|-----------------|
| Form 80 | ~45 fields | Partially filled AcroForm; some fields are flat text, others are editable |
| Form 1221 | ~30 fields | Editable AcroForm |
| 485 Online Sections | Dynamic (online form, not PDF) | N/A — API integration required; out of scope for doc-scan module |

---

## 7. Build Order

### Phase 1 — Foundation (Weeks 1–3)
1. **Form field manifest** — open Form 80 + Form 1221 in Acrobat; export all field names, types, constraints
2. **Upload API** — FastAPI endpoint, file validation, Supabase storage (user-scoped bucket)
3. **Classification prompt** — build and test Claude classification for 7 document types; measure accuracy on 50 samples

### Phase 2 — Extraction Core (Weeks 3–6)
4. **Schema extraction pipeline** — per-document-type Claude prompts; structured JSON output; confidence tier assignment
5. **Open extraction pass** — second Claude call with no schema; merge logic with dedup
6. **Validation layer** — MRZ checksum, date plausibility, ABN/BSB format checks; cross-document name consistency check

### Phase 3 — Fill & Review (Weeks 6–9)
7. **PDF fill (AcroForm)** — pdf-lib integration; field write; generate annotated output PDF
8. **Review UI** — side-by-side extraction display; per-field confirm/correct UX; bilingual labels
9. **Audit log** — Supabase audit trail for all extraction events; GDPR-compliant retention

### Phase 4 — Polish (Weeks 9–12)
10. **Confidence calibration** — measure actual accuracy rate from review UI data; tune thresholds
11. **Error analysis** — sample failed extractions; improve prompts; add new document types
12. **Performance optimization** — reduce per-call latency; batch multi-page PDFs; cache common field extractions

---

## 8. Infrastructure Summary

| Component | Choice | Notes |
|-----------|--------|-------|
| Upload API | FastAPI + Supabase Storage | User-scoped buckets; 20MB limit; virus scan |
| LLM (Vision + Extraction) | Claude Sonnet 4 | Per-call; track spend per user |
| PDF Manipulation | pdf-lib | AcroForm read/write; TypeScript/JS SDK |
| Database | Supabase (Postgres) | Extraction audit log; user-scoped doc storage |
| Review UI | Next.js PWA | Mobile-first; bilingual (EN/NP toggle) |
| Hosting | Vercel (frontend) + Railway (FastAPI) | Fast to ship; cheap to validate |

---

## 9. Open Questions

1. **AcroForm field names for Form 80** — need to verify the actual field names by inspecting the PDF. Some Australian immigration PDFs use non-obvious field names (e.g. `AT01_GivenNames` not `given_names`). This is a prerequisite to building the fill step.

2. **Cross-document consistency enforcement** — if the name on the passport is "Prabin Karki" but the payslip says "P. Karki", should this block fill or warn? Policy decision needed: flag as mismatch → user resolves → proceed.

3. **Retention policy for source documents** — uploaded documents contain PII (passport numbers, financial data). GDPR-equivalent (Australian Privacy Act) requires a clear retention and deletion policy. What is the retention period? Who can access?

4. **Whether to support 485 online (dynamic) form** — the 485 is submitted through ImmiAccount (web-based, not PDF). True form-fill for 485 would require browser automation (Playwright) against ImmiAccount, which has anti-bot protections. Out of scope for now, but confirm with legal before promising this later.

5. **Claude data retention / HIPAA-equivalent** — Australian Privacy Act + the migration context means users are submitting highly sensitive personal data. Confirm with Anthropic what data retention applies and whether a DPA (Data Processing Agreement) is needed for the MVP.

6. **Cost model at scale** — per-document extraction costs ~$0.01–0.05 at Claude pricing. If the product goes viral in the Nepalese community (Facebook-first GTM), cost could spike. When to introduce caching, batch processing, or a cost ceiling?

7. **Nepali-script field values in PDFs** — if a user corrects an extraction with a Nepali-language value (e.g. Nepali name from a Nepali birth certificate), does the target AcroForm support Unicode? Most Australian government forms expect Latin-1 only. May need transliteration requirement in the UI.

---

## 10. References

- Saathi PRD — `PRD.md` (June 28, 2026)
- Saathi Market Research — `docs/MARKET-RESEARCH.md` (June 28, 2026)
- Anthropic Claude API — https://docs.anthropic.com/en/api
- pdf-lib (AcroForm) — https://pdf-lib.org/
- Australian Form 80 — https://immi.homeaffairs.gov.au/forms/documents/80.pdf
- Australian Form 1221 — https://immi.homeaffairs.gov.au/forms/documents/1221.pdf
- MRZ checksum — ISO 7064 MOD 7/11
- Nepalese MRP passport MRZ format — ICAO Doc 9303

---

*Architecture compiled: July 4, 2026*
