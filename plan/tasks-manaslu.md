# Manaslu Repo — Prioritized Task List

**Repo:** `judasprabin/manaslu` | **Stack:** FastAPI + Claude (tiered: Opus/Sonnet/Haiku) + Cloud Run  
**Scope:** Headless scan/extract/form-fill agent service. No UI. Saathi is the only consumer at MVP.

> Infra decision: Cloud Run per doc 09 (GKE explicitly rejected there) — see
> `../saathi/infra/infrastructure-comparison.md` §5 for the cross-repo comparison.

---

## M0 — Foundation (Week 1-2) | Priority: P0

| # | Task | Est. | Depends |
|---|------|------|---------|
| M0.1 | Project scaffold: FastAPI app, Dockerfile, requirements.txt | 0.5d | — |
| M0.2 | Cloud SQL schema: sessions, documents, profile_facts, extractions, fills, audit_log | 1d | — |
| M0.3 | Claude client wrapper: Sonnet primary, Opus for hard cases, Haiku for classification | 1d | M0.1 |
| M0.4 | GCS client: upload/download, signed URL generation, user-scoped bucket structure | 0.5d | M0.1 |
| M0.5 | Identity Platform verification: validate forwarded JWT as resource server | 1d | M0.1 |
| M0.6 | Cloud Run service config: `service.yaml`, ingress = internal (IAM-only invoker, not publicly invokable — trust boundary is IAM, not a K8s NetworkPolicy) | 0.5d | M0.1 |
| M0.7 | GitHub Actions (WIF) pipeline + Artifact Registry push | 0.5d | M0.6 |

**M0 exit:** Agent boots, accepts JWT, writes to DB, deploys to Cloud Run from CI.

---

## M1 — Document Classification (Week 2-3) | Priority: P0

| # | Task | Est. | Depends |
|---|------|------|---------|
| M1.1 | `/sessions` endpoint: create session, associate user, return session_id | 0.5d | M0.3 |
| M1.2 | `/documents` upload: accept multipart, validate type/size, store in GCS | 1d | M0.4 |
| M1.3 | Classification prompt: Claude Haiku classifies into 7 doc types (passport, visa_grant, payslip, bank_statement, birth_cert, transcript, english_test) | 1d | M1.2 |
| M1.4 | Classification confidence + "unknown" handling → route to manual | 0.5d | M1.3 |
| M1.5 | Test: 50 sample documents, ≥95% classification accuracy | 1d | M1.4 |

**M1 exit:** Upload a document → agent correctly classifies it → stores in GCS + records in DB.

---

## M2 — Extraction Engine (Week 3-5) | Priority: P0

| # | Task | Est. | Depends |
|---|------|------|---------|
| M2.1 | Per-document-type extraction schemas (7 JSON schemas) | 1d | M1.3 |
| M2.2 | Schema extraction: Claude Sonnet + doc image + schema → structured JSON | 2d | M2.1 |
| M2.3 | Open extraction: second Claude call (no schema) → catch unexpected fields | 1d | M2.2 |
| M2.4 | Merge + dedup: prefer schema value, flag conflicts, cross-document consistency | 1d | M2.3 |
| M2.5 | Validation: MRZ checksum (ISO 7064), date plausibility, ABN/BSB format | 1.5d | M2.4 |
| M2.6 | Confidence tiering: HIGH ≥85% / MED 50-84% / LOW <50% / FAIL | 0.5d | M2.5 |
| M2.7 | SSE event stream: `extraction.ready`, `review.required`, `fill.completed` | 1d | M2.6 |
| M2.8 | Test: golden document set (10 passports, 5 payslips, 5 bank statements) | 1.5d | M2.7 |

**M2 exit:** Upload passport → agent returns extracted fields with confidence scores via SSE.

---

## M3 — Profile Vault (Week 5-6) | Priority: P0

| # | Task | Est. | Depends |
|---|------|------|---------|
| M3.1 | `profile_facts` table: canonical user facts with provenance (source doc, extraction date, confidence) | 1d | M2.8 |
| M3.2 | Vault upsert logic: new extraction → compare with existing → keep higher confidence → log diff | 1d | M3.1 |
| M3.3 | Cross-extraction consistency: name from passport must match name from payslip → flag mismatch | 0.5d | M3.2 |
| M3.4 | "Fill once, reuse" contract: subsequent sessions pre-populate from vault → reduces Claude calls | 1d | M3.2 |
| M3.5 | Test: scan passport twice (same user) → second session uses vault, not re-extraction | 0.5d | M3.4 |

**M3 exit:** User scans passport once → vault stores facts → future forms auto-fill from vault.

---

## M4 — Form Fill Engine (Week 6-8) | Priority: P0

| # | Task | Est. | Depends |
|---|------|------|---------|
| M4.1 | Form 80 field manifest: open PDF → extract AcroForm field names → JSON mapping | 1d | — |
| M4.2 | Form 1221 field manifest: same as above | 0.5d | M4.1 |
| M4.3 | pdf-lib integration: read AcroForm, write values from vault/extraction, generate annotated PDF | 2d | M4.1 |
| M4.4 | Field mapping engine: vault/extraction field → AcroForm field name table | 1d | M4.3 |
| M4.5 | Confidence-aware fill: only HIGH fields auto-fill; MED fields flagged; LOW/FAIL left blank | 1d | M4.4 |
| M4.6 | Audit annex: generate sidecar PDF showing source document crops per filled field | 1d | M4.5 |
| M4.7 | Artifact endpoints: GET filled PDF + GET audit annex (signed URLs) | 0.5d | M4.6 |
| M4.8 | Test: end-to-end — upload passport + payslips → review → download filled Form 80 + audit | 2d | M4.7 |

**M4 exit:** Upload 3 documents → review extractions → download filled Form 80 with audit trail.

---

## M5 — Polish & Hardening (Week 8-10) | Priority: P1

| # | Task | Est. | Depends |
|---|------|------|---------|
| M5.1 | Rate limiting: per-user session caps, per-session document caps, Claude API quota tracking | 1d | M1.1 |
| M5.2 | Error recovery: retry logic, dead-letter queue, manual intervention endpoint | 1d | M2.7 |
| M5.3 | Performance: Claude prompt caching (60-90% cost reduction on repeated system prompts) | 1d | M0.3 |
| M5.4 | Cost tracking: per-session Claude spend, user-level quota alerts | 1d | M5.1 |
| M5.5 | Observability: structured logging, Cloud Trace integration, extraction accuracy metrics | 1d | — |
| M5.6 | Documentation: API contract v1, field manifests, extraction schemas | 1d | M4.8 |
| M5.7 | Security audit: JWT verification, GCS ACLs, DB RLS, no PII in logs | 1d | M0.5 |

---

## Always-On Tasks

| Task | Cadence | Owner |
|------|---------|-------|
| Golden document set accuracy | Every PR touching extraction | CI |
| Form field manifest updates (Home Affairs changes) | Monthly review | Engineering |
| Claude model version testing | When Anthropic releases new models | Engineering |
| Cost-per-session monitoring | Weekly dashboard review | Engineering |
