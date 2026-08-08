# F4 — Manaslu Integration (saathi's side)

**Status:** Active | **Replaces:** `scan-pipeline.md`, `scan-pipeline-v1.md` (archived out — see below)

The document scan → classify → extract → validate → transliterate → fill
pipeline is **not built in this repo**. It's owned end-to-end by `manaslu`, a
separate headless backend, and documented there:

- `manaslu/docs/architecture/02-scan-extraction.md` — classify + extract + validate
- `manaslu/docs/architecture/03-form-fill-engine.md` — AcroForm mapping + fill
- `manaslu/docs/architecture/04-transliteration.md` — Devanagari → Latin, MRZ-first
- `manaslu/docs/architecture/06-service-api.md` — the API contract summarized below
- `manaslu/docs/architecture/11-mvp-build-plan.md` — manaslu's own M0–M4 build timeline

This doc covers only **saathi's side**: the API contract we consume, what our
review UI must render, identity forwarding, and the one piece of F4 that *is*
still saathi's own build (F4a, knowledge-service/RAG).

## What saathi consumes: manaslu's API (v1)

REST + SSE. Saathi's API layer creates a session, then streams and reacts to
events — it never re-implements extraction or fill logic.

```
POST /v1/sessions                              → {session_id}
POST /v1/sessions/{id}/messages                → SSE event stream
POST /v1/sessions/{id}/documents                → upload (signed-URL handshake)
POST /v1/sessions/{id}/confirmations            → resume a paused session
GET  /v1/sessions/{id}                          → state incl. pending review items
GET  /v1/sessions/{id}/artifacts/{artifact_id}  → filled PDF / annex (signed URL)
```

Events saathi must handle:

| Event | Saathi's obligation |
|-------|----------------------|
| `message.delta` | Render agent narration text |
| `tool.started` / `tool.finished` | Progress indicator (classifying, extracting, filling…) |
| `extraction.ready` | Render the review UI: fields + confidence tiers + source-region crop refs |
| `review.required` | Pause UI on the item needing confirmation (field value, transliteration choice, unsourced field); eventually POST a confirmation with the `request_id` |
| `fill.completed` | Show/download the filled PDF + audit annex artifact ids |
| `session.done` / `session.error` | Terminal state handling |

Full contract detail (versioning, idempotency, RFC 7807 errors, quotas) lives
in manaslu doc 06 — not duplicated here.

## What saathi's review UI must render

Manaslu supplies the data; saathi owns presentation only:

- **Side-by-side review**: extracted value ↔ source document region crop, with confidence badge (HIGH/MED/LOW/FAIL per manaslu doc 02's tiers)
- **Bilingual labels**: EN/NP labels and short explanations ship in the field manifest payload — saathi renders them, never hardcodes form knowledge
- **Transliteration picker**: a sub-screen for `review.required` items where the field is a Devanagari value needing a Latin rendering — presents candidate spelling(s), lets the user pick or type their own (the user's passport spelling is authoritative, never an algorithm's)
- **Confirm/edit/skip** actions per field, matching the confirmations endpoint's shape

See `ui-ux-flows.md` for the screen-by-screen spec.

## Identity: forwarding the end-user JWT

Manaslu is a resource server (manaslu doc 07) — it doesn't own end-user
identity. Every call from saathi's API layer to manaslu must carry:

1. A **Google-signed Cloud Run ID token** proving saathi is the calling service (IAM invoker — manaslu is not publicly invokable)
2. The **end-user's GCP Identity Platform JWT**, forwarded as-is in `Authorization`

The browser never calls manaslu directly — it goes through saathi's own API
layer (BFF), which holds the service-to-service credential and simply passes
the user's token through.

## F4a — knowledge-service / RAG (this IS saathi's own build)

Manaslu is scoped to scan + fill, not explain. Bilingual field-by-field
*explanation* (distinct from the fill flow) is saathi's responsibility:
pgvector ingestion of Home Affairs pages, chunking + embeddings, retrieval,
and a Claude explanation call. See `docs/BUILD-SCHEDULE.md` Phase 4a for the
task breakdown — this piece has no manaslu dependency.
