# Manaslu — Form-Filling Investigation: Approaches, Tools & Techniques

**Date:** August 2026 | **Question:** What is the right way to build a service that fills AU immigration forms from user documents?  
**Method:** First-principles analysis of every viable approach, evaluated against real-world constraints.

---

## 1. The Core Problem, Stated Clearly

Manaslu needs to solve this:

```
INPUT:    User documents (passport photo, payslip PDF, bank statement scan)
          + Target form (Form 80, Form 1221, etc.)

PROCESS:  1. Extract structured data from each document
          2. Map extracted data to the correct form fields
          3. Identify what's still missing
          4. Ask the user for missing information (in Nepali)
          5. Fill the form with confirmed values
          6. Handle multiple forms across time (vault reuse)

OUTPUT:   Filled, annotated PDF the user downloads and lodges themselves
```

The hard parts are (1) extraction accuracy from real-world photos, and (2) the mapping from document data to form fields when documents vary wildly in format.

---

## 2. Six Approaches To Building This

### Approach A: Claude Vision Extraction + Manifest Mapping (Manaslu's Current Design)

**How it works:**
```
Document image → Claude Vision (classify → extract_schema → validate)
               → structured JSON per document type
               → ontology keys → form manifest → AcroForm field names
               → fill PDF
```

**What manaslu's docs actually describe:**

| Stage | Tool | Input | Output |
|-------|------|-------|--------|
| classify | Claude Haiku 4.5 | Thumbnail image | Document type (passport/payslip/...) |
| extract_schema | Claude Sonnet 5 | Full image + JSON schema | Structured fields (name: "PRABIN KARKI", dob: "1990-01-01") |
| extract_open | Claude Haiku 4.5 | Full image (no schema) | All readable name-value pairs (recall pass) |
| merge | Python | Schema + open results | Merged, schema wins on conflict |
| validate | Python (deterministic) | Merged fields | MRZ checksum, date plausibility, ABN/BSB format, cross-doc consistency |
| confidence | Python | Validated fields | HIGH (≥85%) / MED (50-84%) / LOW (<50%) / FAIL |

**Strengths:**
- Handles ANY document format — photos, scans, crumpled paper, bad lighting
- Devanagari support is native (Nepali birth certificates)
- No training data needed — works on day one
- Single vendor (Anthropic) for extraction layer
- Confidence scoring built in

**Weaknesses:**
- Cost: ~$0.02-0.03 per document (Claude API pricing)
- Non-deterministic: same document may produce slightly different results
- No fine-tuning: can't improve accuracy on specific AU document formats
- Latency: ~2-3s per extraction call
- Accuracy ceiling: Claude Vision on phone photos of passports may be ~90-95% field-level

**When to use:** When you have diverse, unpredictable document formats and need to ship fast without training data.

---

### Approach B: Document AI Specialized Parsers (Google/AWS/Azure)

**How it works:**
```
Document image → Specialized pre-built model (passport parser, ID parser, form parser)
               → Highly structured output with per-field confidence
               → Map to ontology keys → form manifest → fill
```

**Available specialized parsers:**

| Provider | Parser | What It Extracts | AU Support? | Cost |
|----------|--------|-----------------|-------------|------|
| Google Document AI | Identity Processor | Name, DOB, passport number, address from passports/IDs | Trained on global passports — AU passports supported | $0.10/page |
| Google Document AI | Form Parser | Key-value pairs from structured forms with field labels | Generic — works on any form with labels | $0.05/page |
| AWS Textract | AnalyzeID | Name, DOB, document number, expiry from passports/driver's licenses | AU passports + driver's licenses | $0.005/page |
| AWS Textract | AnalyzeDocument (FORMS) | Key-value pairs + table extraction + checkbox state | Any form | $0.015/page |
| Azure Document Intelligence | Prebuilt-id-document | Passport, driver's license fields | AU supported | $0.01/page |
| Azure Document Intelligence | Prebuilt-layout | Text, tables, selection marks | Any document | $0.01/page |

**Strengths:**
- Cheaper than Claude (10-50× cheaper per page)
- Deterministic — same input = same output
- Built-in per-field confidence scores (not inferred like Claude)
- Production-grade — used by banks, governments, insurance companies
- No prompt engineering needed

**Weaknesses:**
- No Devanagari support (Nepali documents may fail)
- Trained on clean, flat-bed scanned documents — phone photos may degrade accuracy
- Less flexible — can't handle unusual document formats
- Requires separate integration per provider
- Vendor-specific output format — needs normalization layer

**When to use:** When you have documents in known formats, cost matters, and you need deterministic output.

---

### Approach C: Template-Based MRZ/Region Extraction (No AI)

**How it works:**
```
Document image → Pre-process (deskew, contrast) → Crop known regions
               → OCR on specific areas (MRZ zone, name field, date field)
               → Validate (MRZ checksum, format checks)
               → Structured output
```

**What this looks like for specific document types:**

**Passport (MRZ-first):**
```
1. Detect passport bio page (aspect ratio, photo position)
2. Locate MRZ (2 lines of 44 chars at bottom, always same format)
3. OCR the MRZ zone → get: surname, given_names, passport_number, nationality, dob, expiry, sex
4. This is 100% deterministic — MRZ has built-in checksums (ISO 7064 MOD 7/11)
5. For fields NOT in MRZ (place of birth, issuing authority): use region OCR or ask user
```

**Australian Visa Grant Notice:**
```
1. Detect "Visa Grant Notice" header
2. Locate key sections: "Application ID", "Visa Subclass", "Grant Date", "Expiry", "Conditions"
3. These are ALWAYS in the same format (DHA standard template)
4. Region-based OCR → structured output
```

**Australian Payslip:**
```
1. Detect "PAYSLIP" header + ABN format (XX XXX XXX XXX)
2. Locate: employee name, employer name, ABN, pay period, gross pay, net pay
3. Challenging: every employer formats differently
```

**Strengths:**
- Free (Tesseract OCR is open-source)
- 100% deterministic
- Very fast (< 1 second)
- MRZ extraction is effectively 100% accurate when OCR is good
- No API calls, no vendor dependency

**Weaknesses:**
- Brittle — if the document is rotated, folded, or poorly lit, region detection fails
- Doesn't handle Nepali-language documents (Tesseract Devanagari is weak)
- Manual effort to define regions per document type
- Can't handle unusual formats or new document types
- No built-in confidence scoring

**When to use:** As the first extraction pass for KNOWN document types before falling back to Claude. MRZ extraction should ALWAYS be template-based — there's zero reason to use Claude for a deterministic checksum-verified string.

---

### Approach D: Questionnaire-First (No Document Extraction)

**How it works:**
```
User selects form → Questionnaire appears in their language
                 → User types answers directly
                 → Backend maps answers → form fields
                 → Fill PDF
```

**This is what Instafill does.** Instafill doesn't extract from documents. It presents a web form with the same fields as the PDF, the user types their data, and it fills the PDF programmatically. That's it. Simple, reliable, fast.

**What Instafill's Form 80 flow looks like:**
```
Web form with ~45 fields organized by section:
  Section A: Personal Details
    - Family Name: [___________]
    - Given Names: [___________]
    - Date of Birth: [__/__/____]
    - Place of Birth: [___________]
    ...
  Section B: Contact Details
    ...
  Section C: Family Members
    ...
→ User fills all fields → clicks "Generate" → gets filled PDF
```

**Strengths:**
- 100% accurate (user provides all data)
- Zero AI cost
- Works offline
- Fastest possible UX for users who have their data handy
- Competes directly with Instafill on their own terms

**Weaknesses:**
- User must type everything — no time savings from documents
- No vault reuse — user types the same name/DOB for every form
- No bilingual document reading (Nepali birth certificate)
- Doesn't differentiate from competitors

**When to use:** As the fallback path. Always available. If extraction fails or the user prefers to type, they can. This is the baseline experience.

---

### Approach E: LLM-Powered Questionnaire (Boundless-Style)

**How it works:**
```
User describes their situation in free text (or answers structured questions)
→ LLM interprets answers and maps to form fields
→ Asks clarifying questions for missing/ambiguous fields
→ Fills form
```

**Boundless's approach:**
1. User selects immigration process ("I want to apply for a green card through marriage")
2. Questionnaire adapts based on answers (skip irrelevant sections)
3. User uploads documents as supporting evidence (not for extraction)
4. Licensed attorney reviews before submission
5. **Key difference from manaslu:** Boundless doesn't extract from documents. The user provides data. The attorney verifies it.

**Strengths:**
- Adaptive — only asks relevant questions
- Can explain why each question matters
- Works in any language (translate the questionnaire)
- Proven business model (Boundless was acquired by Payoneer)

**Weaknesses:**
- Still requires user to type everything
- LLM-driven questionnaire can hallucinate questions
- Complex to build — natural language understanding + form mapping
- Boundary between "questionnaire" and "advice" is legally risky

**When to use:** As an enhancement on top of Approach D. After template/Claude extraction fills what it can, an LLM-powered questionnaire fills the remaining gaps with smart, contextual questions in Nepali.

---

### Approach F: Hybrid — Template First, AI Fallback, Smart Questionnaire

**How it works:**
```
Step 1: For each document, try TEMPLATE extraction first
        - Passport → MRZ region OCR → 100% accurate for MRZ fields
        - Visa Grant → region OCR on known DHA format
        - If template succeeds: mark high confidence, skip AI
        - If template fails (rotated image, unknown format): go to Step 2

Step 2: For failed templates, try DOCUMENT AI specialized parser
        - AWS Textract AnalyzeID for passports
        - Azure Document Intelligence for forms
        - If Document AI succeeds: mark confidence from API
        - If Document AI fails or unsupported: go to Step 3

Step 3: For everything else, use CLAUDE VISION
        - classify → extract_schema → extract_open
        - Most expensive, slowest, but handles anything

Step 4: VALIDATE everything
        - MRZ checksum, date plausibility, ABN/BSB format
        - Cross-document consistency (passport name vs payslip name)
        - Confidence tiering

Step 5: MAP to form via manifest
        - ontology keys → AcroForm field names
        - Provenance chain (every value traceable to source)

Step 6: IDENTIFY GAPS
        - manifest.required_keys - known_keys = gaps

Step 7: SMART QUESTIONNAIRE for gaps
        - Each gap → bilingual question + explanation
        - User can: provide value, upload different document, skip, or request help
        - Free-text answers → intent_router → structured action

Step 8: FILL + VERIFY
        - pypdf AcroForm population
        - Post-fill verification pass
        - Audit annex generation
```

**Cost comparison per document:**

| Document Type | Template (free) | Document AI ($0.01) | Claude ($0.03) | Hybrid (expected) |
|---------------|-----------------|---------------------|-----------------|-------------------|
| Passport (MRZ) | ✅ Free | Not needed | Not needed | **$0.00** |
| Visa Grant (DHA format) | ✅ Free | Not needed | Not needed | **$0.00** |
| Payslip (AU format) | ✅ Free if ABN detected | $0.01 if template fails | $0.03 if both fail | **~$0.002** |
| Bank Statement | ❌ No template | $0.01 | $0.03 | **~$0.015** |
| Birth Certificate (Nepali) | ❌ Tesseract NP is weak | ❌ No NP support | $0.03 (Claude NP) | **$0.03** |
| Academic Transcript | ❌ No template | $0.01 | $0.03 | **~$0.015** |
| English Test Result | ❌ No template | $0.01 | $0.03 | **~$0.015** |

**Average cost per 5-document session: hybrid = ~$0.05 vs pure Claude = ~$0.15 (67% cheaper)**

---

## 3. The Mapping Problem: How To Actually Fill Form Fields

Extraction solves "what's in the document." Mapping solves "where does it go in the form." This is harder than it looks.

### 3.1 The AcroForm Approach (Manaslu's Current Design)

Australian immigration PDFs (Form 80, Form 1221) are AcroForms — they have named, programmatically-writable form fields.

**How to discover field names:**

```bash
# D3 Dump Script (manaslu's 'start-now task 1')
python3 -c "
from pypdf import PdfReader
reader = PdfReader('form80.pdf')
for page in reader.pages:
    for annot in page.annotations or []:
        obj = annot.get_object()
        if obj.get('/FT') == '/Tx':  # Text field
            print(f'{obj.get("/T")}: {obj.get("/TU", "")}')
"
```

**What you'll get:**
```
AT01_GivenNames: 
AT02_FamilyName: 
AT03_OtherNames: 
AT04_DateOfBirth_dd: 
AT04_DateOfBirth_mm: 
AT04_DateOfBirth_yyyy: 
...
```

**The problem:** AU immigration form field names are cryptic. `AT01_GivenNames` doesn't tell you what ontology key maps to it. You need a human to annotate: `AT01_GivenNames → identity.given_names`.

**The solution:** The form manifest. A human-curated YAML file that maps:
```yaml
fields:
  - acroform_name: AT01_GivenNames
    ontology_key: identity.given_names
    label_en: Given Names
    label_np: दिएको नाम
    explanation_np: तपाईंको पासपोर्टमा भएको नाम लेख्नुहोस्।...
    type: text
    max_length: 50
    charset: latin
    required: true
    source_priority: [passport.bio_page, birth_certificate, user_entry]
```

### 3.2 The Overlay Approach (Alternative)

Instead of AcroForm field population, render values as positioned text on top of the PDF. Works on ANY PDF — no form fields needed. Used by Instafill.

**How it works:**
1. For each field, know its X, Y position on the PDF page
2. Render the value at that position using a PDF library (reportlab, pypdf)
3. The original PDF is untouched underneath

**Pros:** Works on any PDF. No AcroForm dependency.  
**Cons:** Alignment issues. Can't be edited after. Looks slightly "off."

**When to use:** Fallback for forms that DON'T have AcroForm fields. Form 80/1221 DO have AcroForm fields — so AcroForm is preferred.

### 3.3 The FDF/XFDF Approach (Alternative)

Generate an FDF (Forms Data Format) file — an XML sidecar that maps field names to values. Open the PDF + FDF together in Adobe Reader and the form populates.

**Pros:** Standard format. No PDF manipulation needed.  
**Cons:** Requires Adobe Reader. User must open both files. Not a clean UX.

**When to use:** Not suitable for a web service. Rejected.

### 3.4 Recommendation: AcroForm Primary, Overlay Fallback

This is already manaslu's design (doc 03). No change needed. The key is getting the field manifest right.

---

## 4. The Gap Resolution Problem: How To Figure Out What's Missing

This is where manaslu's gap-resolution engine is clever but potentially insufficient.

### Manaslu's approach:

```python
def resolve_form(uid, form_id):
    manifest = manifests.get(form_id)
    required = manifest.required_ontology_keys   # e.g., [identity.given_names, ...]
    known = vault.recall(uid, required)           # what we already know
    gaps = required - known                        # what's missing
    
    # Extract from new documents to fill gaps
    for doc in session.new_docs:
        extracted = extract(doc)
        vault.save_facts(uid, extracted)
        gaps -= extracted.keys
    
    # Ask user for remaining gaps
    if gaps:
        pause_and_ask(gaps)
```

### The problem with this approach:

**It treats all gaps as equal.** But not all gaps are the same:

| Gap Type | Example | Best Resolution |
|----------|---------|-----------------|
| **Simple fact** | "What is your date of birth?" | Ask directly — user knows this |
| **Ambiguous fact** | "Place of birth: city or district?" | Explain what the form wants, show examples |
| **Document-dependent** | "Employment history for last 10 years" | User needs to reference their CV |
| **Legal judgment** | "Have you ever been convicted?" | Handoff to MARA agent — Saathi can't answer |
| **Conditional** | "Partner details" (only if married) | Skip if condition not met |
| **Multi-source** | "Current address" (on passport? payslip? neither?) | Show all possible sources, let user pick |

**The gap engine should categorize gaps and handle each type differently:**

```python
def categorize_gap(field, manifest):
    if manifest.fields[field].category == "legal_judgment":
        return GapAction.HANDOFF_TO_MARN  # "This needs a registered agent"
    elif manifest.fields[field].category == "conditional":
        return GapAction.ASK_CONDITION    # "Are you married? → if yes, ask partner details"
    elif field_has_hint_in_documents(field):
        return GapAction.SHOW_SOURCES     # "We found 'Sydney NSW 2000' on your bank statement — use this?"
    else:
        return GapAction.ASK_SIMPLE       # "What is your place of birth?"
```

### The manifest should carry this categorization:

```yaml
fields:
  - acroform_name: AT01_GivenNames
    ontology_key: identity.given_names
    gap_category: simple_fact          # ← NEW
    gap_question_en: "What is your given name as shown on your passport?"
    gap_question_np: "तपाईंको पासपोर्टमा भएको नाम के हो?"
    gap_example: "e.g., PRABIN (not Prabin Kumar — middle names go in a separate field)"
    
  - acroform_name: AT67_CriminalCharges
    ontology_key: character.criminal_history
    gap_category: legal_judgment       # ← NEVER ask the user — always handoff
    handoff_message_en: "This question requires legal judgment. Consult a registered migration agent (MARN: verify at mara.gov.au)."
    handoff_message_np: "..."
    
  - acroform_name: AT31_PartnerName
    ontology_key: family.partner.name
    gap_category: conditional          # ← Only ask if user has a partner
    condition: family.marital_status in ["married", "de_facto"]
```

---

## 5. The Multi-Form Problem: How The Vault Actually Works

The vault stores facts as `(uid, ontology_key, value, source, confidence)`. When a new form is requested, the engine queries: "what does this form need?" minus "what do we already know?"

### The missing piece: ontology key versioning and evolution

Form 80 might need `identity.given_names`. Form 1221 might need `identity.given_names` too. Form 1085 might need `applicant.given_name` (different key for the same concept).

**Without a canonical ontology, the vault breaks across forms.**

Manaslu's plan mentions `packages/schemas/ontology.yaml` but doesn't specify:

1. **How keys are structured:**
```
identity.given_names        # Good
identity.address.current.line1  # Good — hierarchical
applicant_given_name         # Bad — inconsistent with identity.given_names
```

2. **How aliases work:**
```
identity.given_names:
  aliases: [applicant.given_name, primary_applicant.name.given]
  description: "The given (first) name of the visa applicant"
  source_documents: [passport.bio_page, birth_certificate]
```

3. **How the vault handles updates:**
```
User changes address → new fact written
Old address → marked as stale (not deleted — provenance chain)
Form 80 fill → uses current (active) address
Audit log → shows previous address was used for Form 80 on Jan 2026
```

### Recommendation: Build the ontology FIRST.

Before a single extraction function, define the ~60-80 keys that Form 80 and Form 1221 need. This is the contract between extraction and mapping. Get it right once.

---

## 6. How Instafill Actually Works (Competitive Intelligence)

Instafill's Form 80 page asks the user to TYPE data into a web form, then generates a PDF. They do NOT extract from documents.

Their competitive advantage:
- Instant (no AI latency)
- Free
- Works on mobile
- 150+ country forms

Their weakness:
- User must type everything
- No document intelligence
- No vault/reuse across forms
- English-only
- No bilingual explanations

**Manaslu can beat Instafill IF:**
1. The extraction from documents saves more time than typing
2. The vault makes the second form dramatically faster
3. The NP explanations genuinely help users who struggle with English forms

**Manaslu loses to Instafill IF:**
1. Extraction is slow (30 seconds of AI latency vs 2 minutes of typing — who wins?)
2. Users have to correct too many extracted fields (correction time > typing time)
3. Only one form is supported (no vault demo)
4. Users don't fill multiple forms (vault is wasted)

---

## 7. Recommended Architecture (Revised)

### Tiered Extraction Pipeline:

```
Document Upload
    │
    ├── Is it a passport? → MRZ template extraction (Tesseract + region crop)
    │   ├── MRZ found + checksum passes → 100% confidence, skip AI
    │   └── MRZ not found → Document AI AnalyzeID
    │       ├── Confidence ≥ 95% → accept
    │       └── Confidence < 95% → Claude Vision
    │
    ├── Is it a DHA visa grant notice? → Template extraction (known format)
    │   ├── Key fields found → accept
    │   └── Not found → Claude Vision
    │
    ├── Is it an AU payslip? → ABN detection + region OCR
    │   ├── ABN found + key fields → accept
    │   └── Not found → Claude Vision
    │
    └── Everything else → Claude Vision (primary), Document AI (fallback)
```

### Gap Categorization:

```
After extraction + validation:
    │
    ├── Gap is simple_fact → ask directly (NP question + example)
    ├── Gap is conditional → check condition, skip or ask
    ├── Gap is document_dependent → prompt to upload different document
    ├── Gap is legal_judgment → MARN handoff (never ask user)
    ├── Gap is multi_source → show all possible values, let user pick
    └── Gap is ambiguous → explain what the form wants, show examples
```

### Form Manifest (Enhanced):

```yaml
form_id: form80
form_title_en: "Form 80 — Personal Particulars for Character Assessment"
form_title_np: "फारम ८० — चरित्र मूल्याङ्कनको लागि व्यक्तिगत विवरण"
version: 1
last_verified: 2026-08-12
source_url: https://immi.homeaffairs.gov.au/form-listing/forms/80.pdf
total_fields: 45
required_ontology_keys:
  - identity.given_names
  - identity.family_name
  - identity.date_of_birth
  - identity.passport_number
  # ... ~60 keys total

fields:
  - acroform_name: AT01_GivenNames
    ontology_key: identity.given_names
    gap_category: simple_fact
    label_en: "Given Names"
    label_np: "दिएको नाम"
    explanation_en: "Your given (first) name as shown on your passport."
    explanation_np: "तपाईंको पासपोर्टमा भएको नाम।"
    gap_question_en: "What is your given name as shown on your passport?"
    gap_question_np: "तपाईंको पासपोर्टमा भएको नाम के हो?"
    gap_example_en: "e.g., PRABIN (not middle names)"
    gap_example_np: "उदाहरण: PRABIN"
    type: text
    max_length: 50
    charset: latin
    required: true
    source_priority:
      - passport.mrz
      - passport.visual_zone
      - birth_certificate
      - user_entry
    validation:
      - not_empty
      - match_passport_name
```

---

## 8. Tools Comparison Matrix

| Tool | Best For | Cost | Accuracy (Passport) | Devanagari | AU Forms |
|------|----------|------|---------------------|------------|----------|
| **Tesseract OCR + MRZ** | Passport MRZ zone | Free | 99%+ (with checksum) | ❌ | N/A |
| **AWS Textract AnalyzeID** | Passports, IDs | $0.005/page | 95%+ | ❌ | ✅ |
| **Google Document AI Identity** | Passports, IDs | $0.10/page | 96%+ | ❌ | ✅ |
| **Azure Document Intelligence ID** | Passports, IDs | $0.01/page | 95%+ | ❌ | ✅ |
| **Claude Vision (Haiku)** | Classification | $0.001/img | N/A (classify only) | ✅ | ✅ |
| **Claude Vision (Sonnet)** | Extraction from any image | $0.02/img | 90-95% (phone photos) | ✅ | ✅ |
| **pypdf** | AcroForm fill | Free | N/A | N/A | ✅ |
| **pikepdf (qpdf)** | PDF repair | Free | N/A | N/A | ✅ |
| **reportlab** | Audit annex + overlay | Free | N/A | ✅ (Unicode) | ✅ |
| **Aksharamukha** | Devanagari→Latin translit | Free | N/A | ✅ | N/A |

---

## 9. What Manaslu's Plan Is Missing — And What To Add

### Already in the plan:
- ✅ Claude Vision tiered extraction
- ✅ Form manifest (YAML)
- ✅ Gap-resolution engine
- ✅ Confidence tiering
- ✅ Vault (profile_facts)
- ✅ Provenance chain
- ✅ Audit annex

### Missing from the plan:

| Missing | Why It Matters | How To Add |
|---------|---------------|------------|
| **Template extraction for MRZ** | Claude is burning credits on checksum-verified strings | Add before M1 — 1 day of work, saves $0.03/passport forever |
| **Document AI as REAL fallback (not documented)** | If Anthropic is down, manaslu is dead | Implement AWS Textract AnalyzeID before M3 |
| **Gap categorization in manifest** | Not all gaps should be handled the same way | Add `gap_category` field to manifest YAML |
| **Ontology with aliases** | Vault breaks if keys aren't consistent across forms | Build ontology.yaml BEFORE extraction code |
| **Smart questionnaire for gaps** | "What is your DOB?" is different from "Criminal history?" | Gap-specific UI in saathi, driven by manifest categories |
| **Benchmark extraction accuracy before launch** | Can't claim accuracy without measuring it | Collect 50-doc golden set during M1 |

---

## 10. Final Recommendation

**The architecture is 80% correct.** The gap-resolution engine, vault, and provenance chain are sound. What's missing:

1. **Template extraction as the first pass** — MRZ, DHA visa notices, and AU payslip ABNs should never touch Claude. They're deterministic formats with known regions.

2. **Document AI as a REAL second pass** — not "documented fallback" but an implemented, tested, CI-verified fallback path. Use AWS Textract (cheapest, best AU support).

3. **Gap categorization** — the manifest must tell the engine HOW to handle each missing field. Not all gaps are "ask user."

4. **Ontology with aliases and versioning** — the contract between extraction and mapping. Build this before code.

5. **Accuracy benchmarking** — a 50-doc test set with CI-enforced accuracy targets. Build this during M1, not after.

**The core insight:** Manaslu's biggest architectural risk is not Claude extraction accuracy — it's that the plan uses the most expensive tool (Claude Vision) for tasks that can be done with free, deterministic tools (MRZ OCR, template matching). A hybrid approach saves 67% on extraction costs AND improves reliability by reducing non-deterministic calls.

---

*Investigation compiled: August 2026*
