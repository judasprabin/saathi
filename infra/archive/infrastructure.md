# Saathi — GKE Infrastructure Architecture

> **ARCHIVED:** superseded by the Cloud Run recommendation in
> `../infrastructure-comparison.md` (§5). Kept for reference only.

**Version:** 1.0 | **Date:** August 2026 | **Cloud:** GCP | **Orchestration:** GKE (Kubernetes)

---

## 1. GCP Project Structure

```
Organization: karki-labs (or personal GCP account)
├── Project: saathi-dev       (development / sandbox)
├── Project: saathi-staging   (pre-production testing)
└── Project: saathi-prod      (production — beta users)
```

**Region:** `australia-southeast1` (Sydney) — lowest latency for AU users  
**Alternative:** `asia-southeast1` (Singapore) if AU region services are limited

---

## 2. GKE Cluster Design

### Cluster Spec

| Attribute | Value | Rationale |
|-----------|-------|-----------|
| Cluster type | GKE Autopilot | Zero node management at beta scale; scales to zero pods |
| Region | australia-southeast1 | AU users — sub-50ms latency |
| Release channel | Regular | Stable, tested releases |
| Network | VPC-native (alias IP) | Pod-to-pod routing, Cloud SQL private IP |
| Workload Identity | Enabled | Pods authenticate to GCP services without key files |
| GKE Gateway | Enabled | Single ingress controller for all services |
| Cost | ~$75/mo (Autopilot base) + pod vCPU/memory | ~$0.10/hour baseline, scales with pods |

### Node Pool Strategy (if GKE Standard is chosen over Autopilot)

```
Pool: default-pool
  - Machine: e2-standard-2 (2 vCPU, 8 GB)
  - Nodes: 2 (min) / 4 (max)
  - Autoscaling: enabled
  - Location: australia-southeast1-a, australia-southeast1-b (multi-zone)
  - Disk: 50 GB balanced persistent disk
```

---

## 3. Namespace & Workload Layout

```
GKE Cluster
├── namespace: saathi
│   ├── Deployment: saathi-web          (Next.js PWA — 2 replicas)
│   │   ├── Container: saathi-web
│   │   ├── Port: 3000
│   │   ├── Resources: 256m CPU / 512Mi (request), 1 CPU / 1Gi (limit)
│   │   ├── Env: NEXT_PUBLIC_API_URL, NEXT_PUBLIC_AUTH_DOMAIN
│   │   └── HPA: min 2, max 6, target 70% CPU
│   │
│   ├── Deployment: saathi-api          (FastAPI — F1/F2/F3 + F4a RAG + BFF)
│   │   ├── Container: saathi-api
│   │   ├── Port: 8000
│   │   ├── Resources: 512m CPU / 1Gi (request), 2 CPU / 2Gi (limit)
│   │   ├── Env: DB_HOST, DB_NAME, VOYAGE_API_KEY, ANTHROPIC_API_KEY
│   │   └── HPA: min 2, max 4, target 70% CPU
│   │
│   ├── Deployment: saathi-knowledge    (RAG ingestion worker — single replica)
│   │   ├── Container: saathi-knowledge
│   │   ├── Port: none (batch job, not service)
│   │   ├── Schedule: CronJob — daily at 7am AEST
│   │   ├── Resources: 1 CPU / 2Gi (request), 2 CPU / 4Gi (limit)
│   │   └── Job: pull from Notion → embed → upsert pgvector
│   │
│   ├── Service: saathi-web-svc         (ClusterIP, port 3000)
│   ├── Service: saathi-api-svc         (ClusterIP, port 8000)
│   │
│   └── ConfigMap: saathi-config
│       ├── API_BASE_URL: https://api.saathi.app
│       ├── WEB_BASE_URL: https://saathi.app
│       └── KNOWLEDGE_REFRESH_INTERVAL: 86400
│
├── namespace: manaslu
│   ├── Deployment: manaslu-agent       (Claude tool-use loop — 1 replica)
│   │   ├── Container: manaslu-agent
│   │   ├── Port: 8000
│   │   ├── Resources: 1 CPU / 2Gi (request), 4 CPU / 4Gi (limit)
│   │   ├── Env: ANTHROPIC_API_KEY, GCS_BUCKET, DB_HOST
│   │   └── HPA: min 1, max 3, target 70% CPU
│   │
│   ├── Service: manaslu-svc            (ClusterIP, port 8000)
│   │
│   └── NetworkPolicy: allow only from saathi namespace
│
├── namespace: monitoring
│   ├── Deployment: cloud-logging-agent (DaemonSet — fluentd/fluent-bit)
│   └── Deployment: metrics-server      (GKE default — HPA support)
│
└── GKE Gateway (single ingress)
    ├── Route: saathi.app → saathi-web-svc:3000
    ├── Route: api.saathi.app → saathi-api-svc:8000 (includes /v1/manaslu/* BFF proxy)
    └── Route: manaslu internal only — no public ingress (saathi-api forwards)
```

---

## 4. Networking & Security

### VPC Design

```
VPC: saathi-vpc
├── Subnet: australia-southeast1-subnet  (10.0.0.0/20)
│   ├── Primary IP range: 10.0.0.0/20 (4,096 IPs)
│   ├── Pod IP range:    10.4.0.0/18 (16,384 IPs) — VPC-native
│   └── Service IP range: 10.0.32.0/22 (1,024 IPs) — VPC-native
│
├── Cloud NAT (for outbound internet — pods call Anthropic, Voyage AI, Notion)
├── Private Google Access (for GCP APIs without external IPs)
│
├── Cloud SQL (PostgreSQL + pgvector)
│   ├── Private IP only (10.0.0.0/20 subnet)
│   ├── No public IP — accessed via VPC peering or Cloud SQL Auth Proxy
│   └── Connection: workloads connect via private IP (no proxy needed in GKE)
│
├── GCS Bucket: saathi-user-documents
│   ├── Location: australia-southeast1
│   ├── Access: Workload Identity (saathi-api + manaslu service accounts)
│   ├── Lifecycle: auto-delete objects > 12 months
│   └── Encryption: Google-managed (default) — CMEK optional for compliance
│
├── Secret Manager
│   ├── anthropic-api-key (for saathi-api + manaslu)
│   ├── voyage-api-key (for saathi-knowledge)
│   ├── notion-integration-token (for saathi-knowledge)
│   ├── db-password (for Cloud SQL)
│   └── Mounted via: External Secrets Operator (ESO) or CSI Secret Store driver
│
├── Identity Platform (Auth)
│   ├── Email/password provider
│   ├── Google OAuth provider
│   ├── JWT issuer: https://securetoken.google.com/saathi-prod
│   └── manaslu configured as resource server (verifies same JWTs)
│
└── IAM (Service Accounts)
    ├── sa-web@saathi-prod.iam.gserviceaccount.com
    │   └── Roles: none (web is client-side — no service account needed)
    ├── sa-api@saathi-prod.iam.gserviceaccount.com
    │   └── Roles: Cloud SQL Client, GCS Object User, Secret Manager Accessor,
    │              Cloud Run Invoker (to call manaslu)
    ├── sa-knowledge@saathi-prod.iam.gserviceaccount.com
    │   └── Roles: Cloud SQL Client, Secret Manager Accessor
    └── manaslu-agent@saathi-prod.iam.gserviceaccount.com
        └── Roles: Cloud SQL Client, GCS Object User, Secret Manager Accessor
```

---

## 5. Cloud SQL (PostgreSQL + pgvector)

| Attribute | Value |
|-----------|-------|
| Instance type | db-custom-1-3840 (1 vCPU, 3.75 GB RAM) — smallest that supports pgvector |
| Storage | 50 GB SSD (auto-grow 10 GB increments, max 200 GB) |
| Version | PostgreSQL 16 |
| Extensions | pgvector, pg_cron, uuid-ossp |
| Backups | Daily automated, 7-day retention |
| High availability | No at beta scale — add read replica at 1K+ MAU |
| Private IP | 10.0.0.0/20 subnet, no public IP |
| Connections | 50 max (sufficient for 2-3 workloads) |

### Database Layout (logical databases)

```
Instance: saathi-db
├── Database: saathi
│   ├── Schema: saathi_public
│   │   ├── users, visas, checklist_sessions, points_results
│   │   ├── knowledge_base (pgvector — F4a RAG embeddings)
│   │   └── audit_log
│   └── RLS: owner_uid on all user-scoped tables
│
├── Database: manaslu
│   ├── Schema: manaslu_public
│   │   ├── sessions, documents, profile_facts, extractions, fills
│   │   └── audit_log
│   └── RLS: session ownership (saathi-api passes user identity)
│
└── 2 databases on 1 instance at beta scale — separate at 1K+ MAU
```

---

## 6. CI/CD Pipeline

```
GitHub (source) → Cloud Build (build + push) → GKE (deploy)

saathi repo:
  push to main → trigger cloudbuild.yaml
    → Build saathi-web Docker image → push to Artifact Registry
    → Build saathi-api Docker image → push to Artifact Registry
    → Build saathi-knowledge Docker image → push to Artifact Registry
    → kubectl apply -f k8s/ (deploy to GKE)
    → Run DB migrations (Cloud SQL Proxy sidecar in Cloud Build)

manaslu repo:
  push to main → trigger cloudbuild.yaml
    → Build manaslu-agent Docker image → push to Artifact Registry
    → kubectl apply -f k8s/ (deploy to GKE)
    → Run DB migrations

karki-labs-infra repo:
  push to main → trigger Terraform Cloud / Cloud Build
    → terraform plan → manual approval → terraform apply
```

### Artifact Registry

```
Location: australia-southeast1
Repository: saathi-docker
├── saathi-web:latest + git-sha tags
├── saathi-api:latest + git-sha tags
├── saathi-knowledge:latest + git-sha tags
└── manaslu-agent:latest + git-sha tags
```

---

## 7. DNS & TLS

| Record | Points to | TLS |
|--------|-----------|-----|
| `saathi.app` | GKE Gateway public IP | Google-managed certificate (auto-renew) |
| `api.saathi.app` | GKE Gateway public IP | Google-managed certificate |
| `*.saathi.app` | GKE Gateway public IP | Wildcard certificate |

Domain registrar: Namecheap (or Cloud Domains) — point NS to Cloud DNS.

```
Cloud DNS: saathi-zone
├── saathi.app.         A      → GKE Gateway IP
├── api.saathi.app.     A      → GKE Gateway IP
├── www.saathi.app.     CNAME  → saathi.app.
└── _acme-challenge     TXT    → (for certificate provisioning)
```

---

## 8. Observability

| Layer | Tool | What It Tracks |
|-------|------|---------------|
| Logs | Cloud Logging (stdout → structured JSON) | Request logs, Claude API calls, errors |
| Metrics | Cloud Monitoring (GKE built-in) | CPU, memory, request latency, error rate |
| Traces | Cloud Trace (OpenTelemetry SDK) | End-to-end request tracing across services |
| Alerts | Cloud Monitoring Alerting | P95 latency > 500ms, error rate > 5%, pod restart > 3/hour |
| Dashboards | Cloud Monitoring Custom | GKE workload view, API latency, Claude API usage, cost |
| Error tracking | Cloud Error Reporting | FastAPI exceptions, manaslu agent failures |
| Uptime | Cloud Monitoring Uptime Checks | `api.saathi.app/health` every 60s |

### Key Alerts (beta scale)

```
1. P95 latency > 500ms for POST /scan/fill → manaslu under load
2. Error rate > 5% on saathi-api → investigate
3. Claude API 429 (rate limit) → backoff or upgrade tier
4. Knowledge base staleness > 48h → crawler failure
5. Pod restart count > 3/hour → crash loop
6. Cloud SQL storage > 80% → expand
7. Cloud SQL connections > 80% of max → connection leak
```

---

## 9. Cost Estimate (Monthly — Beta Scale, < 500 Users)

| Resource | Spec | Est. Cost/Month |
|----------|------|----------------|
| GKE Autopilot (management fee) | — | $75 |
| GKE Pods (~6 pods × 0.5-1 vCPU) | Variable | $40-80 |
| Cloud SQL (1 vCPU, 3.75 GB) | db-custom-1-3840 | $55 |
| GCS (user documents, 50 GB) | Standard storage | $2 |
| Cloud DNS | 1 zone | $1 |
| Artifact Registry | 50 GB | $3 |
| Cloud Load Balancer (GKE Gateway) | 1 LB | $20 |
| Cloud NAT | 1 NAT gateway | $42 |
| Cloud Logging/Monitoring | Basic tier | $5 |
| Secret Manager | ~4 secrets | $2 |
| Identity Platform | < 1K MAU | $0 (free tier) |
| **TOTAL INFRA** | | **~$230-280** |

AI costs (separate — Claude API, Voyage embeddings): **~$30-50/month at beta scale**

**Grand total: ~$260-330/month at beta (0-500 users)**

---

## 10. Scaling Thresholds

| User Count | Change Required | Est. Cost/Month |
|------------|----------------|-----------------|
| 0-500 | Current setup (Autopilot + 1vCPU DB) | ~$260 |
| 500-2,000 | Add Cloud SQL read replica; bump DB to 2 vCPU | ~$450 |
| 2,000-10,000 | Add Redis (Cloud Memorystore) for caching; horizontal pod scaling | ~$800 |
| 10,000-50,000 | Dedicated DB per service; separate GKE node pool; Cloud CDN | ~$2,500 |
| 50,000+ | Multi-region (Sydney + Singapore); message queue (Pub/Sub); CDN | ~$6,000+ |
