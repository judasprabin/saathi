# Infrastructure Repo — Prioritized Task List

**Repo:** `karki-labs-infra` | **Tool:** Terraform (GCP provider) | **CI:** GitHub Actions (WIF, keyless)

> **Compute platform: Cloud Run, not GKE.** `infrastructure-comparison.md` did the
> full comparison and recommends Cloud Run — matching manaslu's own doc 09 (which
> already rejected GKE) and saving ~$145/mo at beta scale. See `README.md` for the
> resource summary and `infrastructure-comparison.md` §3/§5 for the full Cloud Run
> resource inventory and deployment matrix this task list builds toward.

---

## Phase 0 — GCP Foundation (Week 1) | Priority: P0

| # | Task | Est. | Depends |
|---|------|------|---------|
| I0.1 | GCP project creation: `saathi-prod`, `saathi-staging`, `saathi-dev` | 0.5d | — |
| I0.2 | Enable APIs: Cloud Run, Cloud SQL, GCS, Identity Platform, Secret Manager, Artifact Registry, Cloud DNS, Cloud Scheduler, Cloud Monitoring | 0.5d | I0.1 |
| I0.3 | Service account creation: one per Cloud Run service (saathi-web, saathi-api, manaslu-agent, saathi-knowledge), minimum IAM roles each | 0.5d | I0.2 |
| I0.4 | GitHub Actions Workload Identity Federation: keyless GCP auth from GitHub Actions (no service account keys) | 0.5d | I0.3 |
| I0.5 | Terraform state: GCS backend bucket + state locking (no local state) | 0.5d | I0.1 |
| I0.6 | Terraform workspace structure: dev/staging/prod workspaces with variable files | 0.5d | I0.5 |

---

## Phase 1 — Core Infrastructure (Week 1-2) | Priority: P0

| # | Task | Est. | Depends |
|---|------|------|---------|
| I1.1 | VPC + subnet: `saathi-vpc`, australia-southeast1 subnet (10.0.0.0/20) | 1d | I0.4 |
| I1.2 | Serverless VPC Connector: Cloud Run → Cloud SQL private IP (Cloud Run has direct internet egress — no Cloud NAT needed) | 0.5d | I1.1 |
| I1.3 | Cloud SQL: PostgreSQL 16 instance, private IP, 1vCPU/3.75GB, pgvector extension | 1d | I1.1 |
| I1.4 | Cloud SQL databases: `saathi` + `manaslu` databases, users, schema grants | 0.5d | I1.3 |
| I1.5 | GCS bucket: `saathi-user-documents`, AU region, lifecycle policy (12mo auto-delete), uniform ACL | 0.5d | I0.3 |
| I1.6 | Secret Manager: anthropic-api-key, voyage-api-key, notion-token, db-password | 0.5d | I0.3 |
| I1.7 | Artifact Registry: `saathi-docker` repository, AU region | 0.5d | I0.2 |

---

## Phase 2 — Cloud Run Services (Week 2) | Priority: P0

| # | Task | Est. | Depends |
|---|------|------|---------|
| I2.1 | Cloud Run services: saathi-web, saathi-api (public), manaslu-agent (IAM-only invoker, not publicly invokable) | 1d | I1.1, I1.2 |
| I2.2 | Cloud Run Job: saathi-knowledge (Notion → chunk → embed → pgvector), triggered by Cloud Scheduler | 0.5d | I1.4 |
| I2.3 | Domain mapping: `saathi.app` → saathi-web, `api.saathi.app` → saathi-api (Cloud Run automatic HTTPS) | 0.5d | I2.1 |
| I2.4 | Secret Manager → Cloud Run: mount secrets as env vars/volumes directly (native Cloud Run integration, no operator needed) | 0.5d | I1.6, I2.1 |
| I2.5 | IAM invoker bindings: saathi-api service account granted `roles/run.invoker` on manaslu-agent only; manaslu-agent trust boundary is IAM, not a network policy | 0.5d | I0.3, I2.1 |
| I2.6 | Cloud Monitoring: Cloud Run dashboard (request latency, error rate, instance count), log-based metrics, uptime checks | 0.5d | I2.1 |

---

## Phase 3 — DNS, TLS, CI/CD (Week 2-3) | Priority: P0

| # | Task | Est. | Depends |
|---|------|------|---------|
| I3.1 | Cloud DNS: `saathi-zone` with records for saathi.app, api.saathi.app pointing to Cloud Run domain mapping | 0.5d | I2.3 |
| I3.2 | Managed certificates: automatic via Cloud Run domain mapping (no separate cert management) | — | I3.1 |
| I3.3 | Domain registration: saathi.app → point NS to Cloud DNS | 0.5d | I3.1 |
| I3.4 | GitHub Actions workflow: connect GitHub repos (saathi, manaslu) → build → push Artifact Registry → `gcloud run deploy` | 1d | I0.4, I2.1 |
| I3.5 | CI service account permissions: WIF-bound identity → Artifact Registry push + Cloud Run deploy roles | 0.5d | I3.4 |
| I3.6 | Identity Platform: email/password + Google OAuth providers, JWT config, web client ID | 1d | I0.2 |
| I3.7 | Firebase Cloud Messaging: project setup, server key → Secret Manager | 0.5d | I3.6 |
| I3.8 | End-to-end test: deploy a dummy Cloud Run revision → verify it can reach Cloud SQL, GCS, Secret Manager | 1d | I2.4 |

---

## Phase 4 — IaC Quality & Documentation (Week 3) | Priority: P1

| # | Task | Est. | Depends |
|---|------|------|---------|
| I4.1 | Terraform lint: `terraform fmt`, `tflint`, `tfsec` (security scanning) | 0.5d | All |
| I4.2 | Terraform docs: `terraform-docs` auto-generated README per module | 0.5d | All |
| I4.3 | Variable files: per-environment (dev.tfvars, staging.tfvars, prod.tfvars) with descriptions | 0.5d | All |
| I4.4 | State locking test: verify only one terraform apply can run at a time | 0.5d | I0.5 |
| I4.5 | Disaster recovery plan: Cloud SQL backup restore, Cloud Run redeploy from Artifact Registry image, DNS failover | 1d | All |
| I4.6 | Cost budget: GCP billing budget + alert at 50%/80%/100% of ~$150/month (Cloud Run beta-prod estimate — see README.md) | 0.5d | I0.1 |

---

## Terraform Module Structure

```
karki-labs-infra/
├── modules/
│   ├── cloud-run/        # Cloud Run services + Job + IAM invoker bindings
│   ├── cloud-sql/        # PostgreSQL instance + databases + users
│   ├── networking/       # VPC, subnet, Serverless VPC Connector
│   ├── storage/          # GCS buckets + lifecycle policies
│   ├── auth/             # Identity Platform
│   ├── secrets/          # Secret Manager
│   ├── dns/              # Cloud DNS + domain mapping
│   ├── monitoring/       # Cloud Monitoring dashboards + alerts + uptime
│   └── cicd/             # GitHub Actions WIF + Artifact Registry
├── environments/
│   ├── dev/              # dev.tfvars + main.tf
│   ├── staging/          # staging.tfvars + main.tf
│   └── prod/             # prod.tfvars + main.tf
├── terraform.tf          # Backend config + provider
└── variables.tf          # Shared variable definitions
```

---

## Key Environment Differences

| Resource | dev | staging | prod |
|----------|-----|---------|------|
| Cloud Run | All services min=0 (scale-to-zero) | All services min=0 | manaslu-agent min=1 (SSE latency); web/api min=0 |
| Cloud SQL | 1 vCPU, 25 GB | 1 vCPU, 50 GB | 1 vCPU, 50 GB + backups |
| GCS lifecycle | 30 days | 90 days | 12 months |
| Domain | dev.saathi.app | staging.saathi.app | saathi.app |
| Monitoring alerts | Email only | Email + Discord | Email + Discord + PagerDuty |
