# Saathi — Document Scan & Form-Fill Pipeline

**Version:** 2.0 | **Date:** August 8, 2026 | **Feature:** F4 (Form Helper)

Detailed architecture for the document upload → classify → extract → validate → fill pipeline.

---

## Pipeline Overview

```
UPLOAD → CLASSIFY → SCHEMA EXTRACT → OPEN EXTRACT → MERGE → VALIDATE
                                                              │
                    ┌─────────────────────────────────────────┘
                    ▼
            CONFIDENCE TIER → USER REVIEW → PDF FILL → EXPORT
```

---

## Stage 1: Upload

**Endpoint:** `POST /scan/upload`
**Handler:** FastAPI → Supabase Storage

```
Request: multipart/form-data (max 10 files, 20MB each)
Supported: PDF, PNG, JPG, HEIC

Flow:
1. Validate JWT → extract user_id
2. Validate file type (allowlist) + size (≤ 20MB)
3. Generate UUID filename: {user_id}/{uuid}.{ext}
4. Upload to Supabase Storage (user-scoped bucket)
5. Return: [{ file_id, file_url, mime_type, size }]
```

---

## Stage 2: Classify

**Endpoint:** `POST /scan/classify`
**Model:** Claude Vision (Sonnet 4.5)

```
Input: image_url + classification prompt
Prompt: "Classify this document as one of:
  passport, visa_grant, payslip, bank_statement,
  birth_certificate, academic_transcript,
  english_test, other."

Response: { doc_type: "passport", confidence: 0.97 }
```

Document type determines which extraction schema to use in Stage 3.

---

## Stage 3: Schema Extraction

**Endpoint:** `POST /scan/extract`
**Model:** Claude Vision (Sonnet 4.5)

Per-document-type extraction schemas:

**Passport schema:**
```
full_name, date_of_birth, passport_number, nationality,
issuing_country, date_of_issue, date_of_expiry, mrz_text
```

**Visa Grant schema:**
```
visa_subclass, grant_date, expiry_date, conditions[],
visa_grant_number, applicant_name
```

**Payslip schema:**
```
employer_name, abn, employee_name, gross_pay,
pay_frequency, period_start, period_end
```

**Bank Statement schema:**
```
account_holder, bsb, account_number,
statement_period_start, statement_period_end
```

Response: `{ fields: { field_name: { value, confidence } } }`

---

## Stage 4: Open Extraction

**Endpoint:** Same as Stage 3 (second Claude call)

No schema. Claude returns all readable name-value pairs.

**Purpose:** Catch fields not covered by the schema (e.g., middle names, unusual passport fields, additional conditions on visa grant letters).

Response: `{ pairs: [{ key, value, confidence }] }`

---

## Stage 5: Merge & Dedup

- Combine schema + open extraction results
- On conflict: prefer schema value (more reliable)
- Cross-validate: name from passport must match name from payslip
- Flag mismatches for review

---

## Stage 6: Validate

**Validation rules by field type:**

| Field | Rule | Method |
|-------|------|--------|
| Passport number | MRZ checksum (ISO 7064 MOD 7/11) | Algorithmic |
| Date of birth | Plausible range (must be >16 years ago, <100 years ago) | Range check |
| ABN | 11 digits, valid checksum | Algorithmic |
| BSB | 6 digits, valid AU format | Regex |
| Passport expiry | Must be > issue date | Comparison |
| Name consistency | Passport name ≈ payslip/visa name | Fuzzy match |
| English test date | Must be < 3 years ago (for visa purposes) | Range check |

---

## Stage 7: Confidence Tiering

```
≥ 85%  → HIGH    → auto-fill, green highlight
50-84% → MEDIUM  → pre-fill, amber, user must confirm
< 50%  → LOW     → leave blank, red, user enters manually
FAIL   → BLANK   → "Could not read — enter manually"
```

**Confidence sources:**
1. Claude's self-reported confidence
2. Dual-extraction agreement (schema + open)
3. Validation rule pass/fail
4. Cross-document consistency

---

## Stage 8: User Review UI

Side-by-side table:
- Left: Field name (EN+NP label)
- Center: Extracted value with confidence badge
- Right: Source document snippet (zoomable)

User can: confirm (green check), edit (text input), or skip (leave blank).

---

## Stage 9: PDF Fill

**Library:** pdf-lib (JavaScript/AcroForm)
**Source:** Form 80 and Form 1221 official PDFs from immi.homeaffairs.gov.au

- Read field names from form manifest JSON
- Write confirmed values to named AcroForm fields
- Generate annotated output PDF (highlighted fields)
- Store in Supabase Storage

---

## Stage 10: Export & Audit

- Download filled PDF
- Audit log: user_id, doc_type, fields extracted, confidence scores, corrections made
- Stored in `form_extractions` table (RLS enforced)
- User can delete extraction data at any time

---

## Error Handling

| Error | Response | Recovery |
|-------|----------|----------|
| File too large (>20MB) | 400 | Prompt user to compress |
| Unsupported format | 400 | Show supported formats |
| Classification failed | Claude: "unknown" type | Route to manual review |
| Extraction timeout (>30s) | 504 | Retry once; if still fails → manual |
| Both extractions fail | 422 EXTRACTION_FAILED | Route to manual entry form |
| PDF fill error (missing field) | 500 | Log field name; fill what we can |

---

## Cost Model

| Operation | Model | Est. Cost/Call |
|-----------|-------|---------------|
| Classify (low-res thumbnail) | Claude Sonnet 4.5 | ~$0.002 |
| Schema extract (full image) | Claude Sonnet 4.5 | ~$0.01 |
| Open extract (full image) | Claude Sonnet 4.5 | ~$0.01 |
| **Total per document set (3 docs)** | | **~$0.07** |

At 100 users/month scanning 3 docs each: ~$21/month.

---

## Security & Privacy

- All documents stored in user-scoped Supabase Storage buckets
- RLS prevents cross-user access
- Claude API: confirm data retention policy with Anthropic
- Australian Privacy Act 1988: user can request full deletion
- No PII in logs — log user_id + document type only
- Auto-delete policy: 12 months (configurable)

---

*Pipeline architecture compiled: August 8, 2026*