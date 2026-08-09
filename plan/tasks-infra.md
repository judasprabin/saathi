# Infrastructure Repo — Prioritized Task List

**Repo:** `karki-labs-infra` | **Tool:** Terraform (GCP provider) | **CI:** Cloud Build / Terraform Cloud

---

## Phase 0 — GCP Foundation (Week 1) | Priority: P0

| # | Task | Est. | Depends |
|---|------|------|---------|
| I0.1 | GCP project creation: `saathi-prod`, `saathi-staging`, `saathi-dev` | 0.5d | — |
| I0.2 | Enable APIs: GKE, Cloud SQL, GCS, Identity Platform, Secret Manager, Artifact Registry, Cloud DNS, Cloud Build, Cloud Monitoring | 0.5d | I0.1 |
| I0.3 | Service account creation: sa-api, sa-knowledge, manaslu-agent with minimum IAM roles | 0.5d | I0.2 |
| I0.4 | Workload Identity setup: link GCP SAs to GKE K8s SAs | 0.5d | I0.3 |
| I0.5 | Terraform state: GCS backend bucket + state locking (no local state) | 0.5d | I0.1 |
| I0.6 | Terraform workspace structure: dev/staging/prod workspaces with variable files | 0.5d | I0.5 |

---

## Phase 1 — Core Infrastructure (Week 1-2) | Priority: P0

| # | Task | Est. | Depends |
|---|------|------|---------|
| I1.1 | VPC + subnet: `saathi-vpc`, australia-southeast1 subnet (10.0.0.0/20), secondary ranges for pods/services | 1d | I0.4 |
| I1.2 | Cloud NAT + Private Google Access: outbound internet for pods | 0.5d | I1.1 |
| I1.3 | Cloud SQL: PostgreSQL 16 instance, private IP, 1vCPU/3.75GB, pgvector extension | 1d | I1.1 |
| I1.4 | Cloud SQL databases: `saathi` + `manaslu` databases, users, schema grants | 0.5d | I1.3 |
| I1.5 | GCS bucket: `saathi-user-documents`, AU region, lifecycle policy (12mo auto-delete), uniform ACL | 0.5d | I0.4 |
| I1.6 | Secret Manager: anthropic-api-key, voyage-api-key, notion-token, db-password | 0.5d | I0.3 |
| I1.7 | Artifact Registry: `saathi-docker` repository, AU region | 0.5d | I0.2 |

---

## Phase 2 — GKE Cluster (Week 2) | Priority: P0

| # | Task | Est. | Depends |
|---|------|------|---------|
| I2.1 | GKE Autopilot cluster: australia-southeast1, Regular release channel, VPC-native, Workload Identity | 1d | I1.1 |
| I2.2 | Namespaces: `saathi`, `manaslu`, `monitoring` | 0.5d | I2.1 |
| I2.3 | GKE Gateway: single ingress controller, HTTPRoute for saathi.app + api.saathi.app | 1d | I2.1 |
| I2.4 | External Secrets Operator: install via Helm, ClusterSecretStore pointing to GCP Secret Manager | 0.5d | I2.1 |
| I2.5 | Workload Identity bindings: K8s SAs → GCP SAs with IAM roles | 0.5d | I0.4 |
| I2.6 | NetworkPolicy: manaslu namespace only accepts from saathi namespace | 0.5d | I2.2 |
| I2.7 | Cloud Monitoring: GKE dashboard, log-based metrics, uptime checks | 0.5d | I2.1 |

---

## Phase 3 — DNS, TLS, CI/CD (Week 2-3) | Priority: P0

| # | Task | Est. | Depends |
|---|------|------|---------|
| I3.1 | Cloud DNS: `saathi-zone` with A records for saathi.app, api.saathi.app pointing to GKE Gateway IP | 0.5d | I2.3 |
| I3.2 | Managed certificates: Google-managed SSL for saathi.app, api.saathi.app (auto-renew) | 0.5d | I3.1 |
| I3.3 | Domain registration: saathi.app → point NS to Cloud DNS | 0.5d | I3.1 |
| I3.4 | Cloud Build trigger: connect GitHub repos (saathi, manaslu) → Cloud Build | 1d | I2.1 |
| I3.5 | CI/CD service account: Cloud Build → GKE deploy + Artifact Registry push permissions | 0.5d | I3.4 |
| I3.6 | Identity Platform: email/password + Google OAuth providers, JWT config, web client ID | 1d | I0.2 |
| I3.7 | Firebase Cloud Messaging: project setup, server key → Secret Manager | 0.5d | I3.6 |
| I3.8 | End-to-end test: deploy dummy pod → verify it can reach Cloud SQL, GCS, Secret Manager | 1d | I2.5 |

---

## Phase 4 — IaC Quality & Documentation (Week 3) | Priority: P1

| # | Task | Est. | Depends |
|---|------|------|---------|
| I4.1 | Terraform lint: `terraform fmt`, `tflint`, `tfsec` (security scanning) | 0.5d | All |
| I4.2 | Terraform docs: `terraform-docs` auto-generated README per module | 0.5d | All |
| I4.3 | Variable files: per-environment (dev.tfvars, staging.tfvars, prod.tfvars) with descriptions | 0.5d | All |
| I4.4 | State locking test: verify only one terraform apply can run at a time | 0.5d | I0.5 |
| I4.5 | Disaster recovery plan: Cloud SQL backup restore, GKE cluster rebuild, DNS failover | 1d | All |
| I4.6 | Cost budget: GCP billing budget + alert at 50%/80%/100% of $500/month | 0.5d | I0.1 |

---

## Terraform Module Structure

```
karki-labs-infra/
├── modules/
│   ├── gke/              # GKE cluster + node pools + namespaces
│   ├── cloud-sql/        # PostgreSQL instance + databases + users
│   ├── networking/       # VPC, subnets, NAT, firewall rules
│   ├── storage/          # GCS buckets + lifecycle policies
│   ├── auth/             # Identity Platform + Workload Identity
│   ├── secrets/          # Secret Manager + External Secrets Operator
│   ├── dns/              # Cloud DNS + managed certificates
│   ├── monitoring/       # Cloud Monitoring dashboards + alerts + uptime
│   └── cicd/             # Cloud Build triggers + Artifact Registry
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
| GKE | Autopilot, min 1 pod each | Autopilot, min 2 pods | Autopilot, min 2 pods |
| Cloud SQL | 1 vCPU, 25 GB | 1 vCPU, 50 GB | 1 vCPU, 50 GB + backups |
| GCS lifecycle | 30 days | 90 days | 12 months |
| Domain | dev.saathi.app | staging.saathi.app | saathi.app |
| Monitoring alerts | Email only | Email + Discord | Email + Discord + PagerDuty |
