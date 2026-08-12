# Saathi — Complete Infrastructure Plan: GKE vs Cloud Run

**Version:** 1.0 | **Date:** August 2026 | **GCP Region:** australia-southeast1 (Sydney)

---

## 1. Service Inventory (What Needs to Run)

Based on deep review of both repos. Every service, its dependencies, and why it exists.

### From `judasprabin/saathi`

| # | Service | Type | Purpose | Dependencies |
|---|---------|------|---------|-------------|
| S1 | **saathi-web** | Web app | Next.js 14 PWA — all UI (F1-F6), client-side only | None at runtime (static + client-side API calls) |
| S2 | **saathi-api** | API server | FastAPI — F1/F2/F3 CRUD endpoints + F4a RAG queries + manaslu BFF proxy | Cloud SQL, Secret Manager, manaslu (via Cloud Run IAM) |
| S3 | **saathi-knowledge** | Batch job | CronJob — pull Notion registry → chunk → embed (Voyage) → upsert pgvector | Cloud SQL, Notion API, Voyage API, Secret Manager |

### From `judasprabin/manaslu`

| # | Service | Type | Purpose | Dependencies |
|---|---------|------|---------|-------------|
| M1 | **manaslu-agent** | API server | FastAPI — gap-resolution engine: classify/extract/vault/fill + SSE events | Cloud SQL, GCS, Anthropic API, Secret Manager, Cloud Run IAM (S2S) |

### Shared Infrastructure

| # | Resource | Type | Purpose | Used By |
|---|----------|------|---------|---------|
| I1 | **Cloud SQL** | Database | PostgreSQL 16 + pgvector — 2 databases (saathi + manaslu) | S2, S3, M1 |
| I2 | **GCS** | Storage | 2 buckets: user-documents (S2 + M1), filled-artifacts (M1) | S2, M1 |
| I3 | **Identity Platform** | Auth | Email/Google OAuth, JWT issuer, Firebase Admin SDK | S1, S2, M1 |
| I4 | **Secret Manager** | Secrets | Anthropic key, Voyage key, Notion token, DB password, FCM key | S2, S3, M1 |
| I5 | **Artifact Registry** | Container registry | Docker images for all 4 services | CI/CD |
| I6 | **Cloud DNS** | DNS | saathi.app, api.saathi.app | Ingress |
| I7 | **VPC** | Networking | Private IPs, Cloud NAT, Private Google Access | All |

### External APIs (No GCP Resources)

| API | Used By | Purpose | Pricing |
|-----|---------|---------|---------|
| Anthropic (Claude) | M1 (Sonnet extract, Haiku classify) | Document classification + field extraction | Pay-per-token |
| Voyage AI | S3 | `voyage-multilingual-2` embeddings for RAG | Pay-per-token |
| Notion API | S3 | Read AU Visa Source Registry database | Free tier |
| Firebase Cloud Messaging | S2 | Push notifications to PWA | Free tier |

---

## 2. Approach A — GKE Enterprise Architecture

Full Kubernetes cluster. Maximum control, higher baseline cost, production-grade from day one.

### Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                    GCP Project: saathi-prod                      │
│                    Region: australia-southeast1                   │
│                                                                   │
│  VPC: saathi-vpc (10.0.0.0/20)                                   │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  GKE Autopilot Cluster                                    │    │
│  │                                                            │    │
│  │  ┌──────────────────────────────────────────────────┐    │    │
│  │  │  GKE Gateway (single ingress)                     │    │    │
│  │  │  saathi.app → saathi-web:3000                     │    │    │
│  │  │  api.saathi.app → saathi-api:8000                 │    │    │
│  │  └──────────────────────────────────────────────────┘    │    │
│  │                                                            │    │
│  │  namespace: saathi                                         │    │
│  │  ┌────────────────┐  ┌──────────────────┐                 │    │
│  │  │ saathi-web     │  │ saathi-api       │                 │    │
│  │  │ (Next.js PWA)  │  │ (FastAPI)        │                 │    │
│  │  │ 2 replicas     │  │ 2 replicas       │                 │    │
│  │  │ 256m/512Mi req │  │ 512m/1Gi req    │                 │    │
│  │  └────────────────┘  └──────────────────┘                 │    │
│  │                                                            │    │
│  │  ┌────────────────────────────────────┐                    │    │
│  │  │ saathi-knowledge (CronJob)         │                    │    │
│  │  │ daily 7am: pull Notion → embed → DB│                    │    │
│  │  └────────────────────────────────────┘                    │    │
│  │                                                            │    │
│  │  namespace: manaslu                                         │    │
│  │  ┌────────────────────────────────────┐                    │    │
│  │  │ manaslu-agent (FastAPI)            │                    │    │
│  │  │ 1 replica, 1CPU/2Gi req            │                    │    │
│  │  │ NOT publicly invokable              │                    │    │
│  │  │ NetworkPolicy: saathi → manaslu only│                    │    │
│  │  └────────────────────────────────────┘                    │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Cloud SQL (private IP: 10.0.0.0/20)                     │    │
│  │  PostgreSQL 16 + pgvector | 1vCPU/3.75GB | 50GB SSD     │    │
│  │  2 databases: saathi + manaslu                            │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                   │
│  ┌──────────┐  ┌──────────────┐  ┌─────────────────────────┐    │
│  │ GCS      │  │ Secret Mgr   │  │ Identity Platform       │    │
│  │ docs/    │  │ 4 secrets    │  │ Email + Google OAuth    │    │
│  │ artifacts│  └──────────────┘  └─────────────────────────┘    │
│  └──────────┘                                                     │
│                                                                   │
│  Cloud NAT · Cloud DNS · Artifact Registry · Cloud Monitoring    │
└─────────────────────────────────────────────────────────────────┘
```

### GKE Resource Inventory & Costs

| # | Resource | Spec | Purpose | Monthly Cost |
|---|----------|------|---------|-------------|
| G1 | GKE Autopilot (management) | 1 cluster, Regular channel | Orchestration base fee | **$75.00** |
| G2 | saathi-web pods | 2 replicas × 256m CPU / 512Mi RAM | Next.js PWA serving | **$18.00** |
| G3 | saathi-api pods | 2 replicas × 512m CPU / 1Gi RAM | FastAPI CRUD + RAG + BFF | **$36.00** |
| G4 | manaslu-agent pod | 1 replica × 1 CPU / 2Gi RAM | Gap-resolution engine + SSE | **$28.00** |
| G5 | saathi-knowledge CronJob | ~2h/day × 1 CPU / 2Gi | Daily Notion → pgvector sync | **$6.00** |
| G6 | GKE Gateway (Load Balancer) | 1 regional external LB | Ingress for web + API | **$20.00** |
| G7 | Cloud SQL | db-custom-1-3840, 50GB SSD | PostgreSQL + pgvector | **$55.00** |
| G8 | GCS | 2 buckets, ~10GB, AU region | Documents + filled PDFs | **$1.50** |
| G9 | Cloud NAT | 1 NAT gateway | Outbound internet for pods | **$42.00** |
| G10 | Cloud DNS | 1 zone (saathi.app) | DNS for saathi.app + api | **$1.00** |
| G11 | Artifact Registry | 1 repo, ~5GB | Docker images | **$1.50** |
| G12 | Secret Manager | 4 secrets, 0 active versions | API keys + DB password | **$2.40** |
| G13 | Identity Platform | <1K MAU | Email/Google OAuth | **$0.00** (free) |
| G14 | Cloud Monitoring | Basic tier | Logs, metrics, uptime checks | **$5.00** |
| G15 | Cloud Build | ~10 builds/month | CI/CD (GitHub Actions primary) | **$2.00** |
| G16 | External Secrets Operator | Helm chart, minimal resources | Sync secrets to K8s | **$0.00** (runs on GKE) |
| | | | **GKE TOTAL** | **~$293/mo** |

### GKE Pros & Cons

| Pros | Cons |
|------|------|
| Full Kubernetes — portable, industry-standard | $75/mo cluster management fee (wasted at beta) |
| Fine-grained resource control per workload | ~$293/mo total (vs ~$140 for Cloud Run) |
| Namespace isolation (saathi vs manaslu) | Cluster mental load — maintain K8s manifests |
| HPA auto-scaling with custom metrics | Cloud NAT $42/mo is dead weight at <100 users |
| NetworkPolicy enforcement (saathi → manaslu only) | Overkill — manaslu already rejected GKE (doc 09) |
| CronJob native (saathi-knowledge) | CronJob is the only K8s-unique feature here |

---

## 3. Approach B — Cloud Run Lightweight Architecture

Serverless containers. Scale-to-zero. What manaslu already chose. Minimal ops.

### Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                    GCP Project: saathi-prod                      │
│                    Region: australia-southeast1                   │
│                                                                   │
│  VPC: saathi-vpc (10.0.0.0/20)                                   │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Cloud Run Services                                       │    │
│  │                                                            │    │
│  │  ┌──────────────────────┐  ┌────────────────────────┐    │    │
│  │  │ saathi-web           │  │ saathi-api              │    │    │
│  │  │ public, 256m/512Mi   │  │ public, 512m/1Gi        │    │    │
│  │  │ min 0, max 10        │  │ min 0, max 10           │    │    │
│  │  │ timeout: 5m          │  │ timeout: 15m            │    │    │
│  │  └──────────────────────┘  └────────────────────────┘    │    │
│  │                                                            │    │
│  │  ┌──────────────────────────────────────────────────┐    │    │
│  │  │ manaslu-agent                                     │    │    │
│  │  │ NOT public — IAM invoker only                      │    │    │
│  │  │ 1CPU/1Gi, min 0 prod (min 1 if latency matters)  │    │    │
│  │  │ timeout: 60m (SSE sessions)                        │    │    │
│  │  └──────────────────────────────────────────────────┘    │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Cloud Scheduler + Cloud Run Jobs                         │    │
│  │  ┌──────────────────────────────────────────────────┐    │    │
│  │  │ saathi-knowledge (Cloud Run Job)                  │    │    │
│  │  │ Schedule: daily 7am AEST via Cloud Scheduler      │    │    │
│  │  │ 1CPU/2Gi, timeout: 30m                            │    │    │
│  │  │ Pull Notion → embed (Voyage) → upsert pgvector    │    │    │
│  │  └──────────────────────────────────────────────────┘    │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Cloud SQL (private IP: 10.0.0.0/20)                     │    │
│  │  PostgreSQL 16 + pgvector | 1vCPU/3.75GB | 50GB SSD     │    │
│  │  2 databases: saathi + manaslu                            │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                   │
│  ┌──────────┐  ┌──────────────┐  ┌─────────────────────────┐    │
│  │ GCS      │  │ Secret Mgr   │  │ Identity Platform       │    │
│  │ docs/    │  │ 4 secrets    │  │ Email + Google OAuth    │    │
│  │ artifacts│  └──────────────┘  └─────────────────────────┘    │
│  └──────────┘                                                     │
│                                                                   │
│  Cloud DNS · Artifact Registry · Cloud Monitoring               │
│  No Cloud NAT needed — Cloud Run has direct internet egress      │
└─────────────────────────────────────────────────────────────────┘
```

### Cloud Run Resource Inventory & Costs

| # | Resource | Spec | Purpose | Monthly Cost |
|---|----------|------|---------|-------------|
| C1 | saathi-web (Cloud Run) | 256m CPU / 512Mi, min 0, max 10 | Next.js PWA — scale-to-zero, cold start ~1s | **$3.00** |
| C2 | saathi-api (Cloud Run) | 512m CPU / 1Gi, min 0, max 10 | FastAPI — scale-to-zero, cold start ~2s | **$8.00** |
| C3 | manaslu-agent (Cloud Run) | 1 CPU / 1Gi, min 0 dev / **1 prod**, max 5 | Gap-resolution engine — IAM-only invoker | **$28.00** |
| C4 | saathi-knowledge (Cloud Run Job) | 1 CPU / 2Gi, ~2h/day | Daily Notion → pgvector batch | **$6.00** |
| C5 | Cloud Scheduler | 1 job (daily 7am trigger) | Triggers C4 | **$0.00** (3 jobs free) |
| C6 | Cloud SQL | db-custom-1-3840, 50GB SSD, private IP | PostgreSQL + pgvector | **$55.00** |
| C7 | GCS | 2 buckets, ~10GB, AU region | Documents + filled PDFs | **$1.50** |
| C8 | Serverless VPC Connector | 1 connector (min 2 instances) | Cloud Run → Cloud SQL private IP | **$35.00** |
| C9 | Cloud DNS | 1 zone (saathi.app) | DNS for saathi.app + api | **$1.00** |
| C10 | Artifact Registry | 1 repo, ~5GB | Docker images | **$1.50** |
| C11 | Secret Manager | 4 secrets, 0 active versions | API keys + DB password | **$2.40** |
| C12 | Identity Platform | <1K MAU | Email/Google OAuth | **$0.00** (free) |
| C13 | Cloud Monitoring | Basic tier | Logs, metrics, uptime checks | **$5.00** |
| C14 | Cloud Build | ~10 builds/month | CI/CD (GitHub Actions primary) | **$2.00** |
| | | | **CLOUD RUN TOTAL** | **~$148/mo** |

### Cloud Run Pros & Cons

| Pros | Cons |
|------|------|
| ~$148/mo total — 50% cheaper than GKE | Cold start latency (~1-2s) on scale-to-zero |
| Scale-to-zero on dev (near-zero idle cost) | Serverless VPC Connector $35/mo is the single biggest line item |
| Zero cluster management — no K8s manifests | Manaslu min-instances=1 costs $28/mo (always-on for SSE sessions) |
| Cloud Run IAM = built-in S2S auth (manaslu's model) | CronJob → Cloud Scheduler + Cloud Run Jobs (extra component) |
| Manaslu already chose this — no split-brain infra | 60m request timeout ceiling (fine for SSE sessions) |
| Direct internet egress — no Cloud NAT needed | No NetworkPolicy equivalent (trust boundary is IAM, not network) |
| Automatic HTTPS + custom domain mapping | Less portable than K8s if multi-cloud ever needed |

---

## 4. Side-by-Side Comparison

### Cost Comparison

| Category | GKE Enterprise | Cloud Run Light | Delta |
|----------|---------------|-----------------|-------|
| Compute (web + api + agent + knowledge) | $163/mo | $45/mo | **-$118** |
| Database (Cloud SQL) | $55/mo | $55/mo | Same |
| Networking (NAT vs VPC Connector) | $42/mo | $35/mo | -$7 |
| Orchestration (GKE fee vs none) | $75/mo | $0/mo | **-$75** |
| DNS, Registry, Secrets, Monitoring | $12/mo | $12/mo | Same |
| Identity Platform | $0/mo | $0/mo | Same |
| **TOTAL INFRA** | **~$293/mo** | **~$148/mo** | **-$145/mo (49% less)** |

### Feature Comparison

| Feature | GKE | Cloud Run | Winner |
|---------|-----|-----------|--------|
| Scale-to-zero (dev/staging cost) | ❌ (base fee always) | ✅ (0 pods = $0) | Cloud Run |
| Cold start latency | N/A (always warm) | ~1-2s first request | GKE |
| Native CronJob | ✅ (K8s CronJob) | ⚠️ (Scheduler + Jobs) | GKE |
| Network isolation between services | ✅ (NetworkPolicy) | ⚠️ (IAM only) | GKE |
| Portability (multi-cloud) | ✅ | ❌ (locked to Cloud Run) | GKE |
| S2S auth built-in | ⚠️ (needs setup) | ✅ (Cloud Run IAM) | Cloud Run |
| HTTPS + custom domain out of box | ⚠️ (needs Gateway) | ✅ (automatic) | Cloud Run |
| Long-running SSE (manaslu) | ✅ (no timeout) | ✅ (60m timeout, sufficient) | Tie |
| Manaslu already chose this | ❌ (doc 09 rejects GKE) | ✅ (doc 09 recommends) | Cloud Run |
| Ops burden (solo dev) | High (K8s manifests) | Low (gcloud deploy) | Cloud Run |
| Cost at <100 users | $293/mo | $148/mo | Cloud Run |
| Cost at 10K users | Similar (pods scale) | Similar (instances scale) | Tie |

### Scalability Comparison

| Scale | GKE | Cloud Run |
|-------|-----|-----------|
| 0-100 users | Overkill — $293/mo minimum | Perfect — $148/mo, scale-to-zero on dev |
| 100-1,000 users | Comfortable — HPA auto-scales | Comfortable — auto-scales instances |
| 1,000-10,000 users | Add node pools, Redis cache | Add min-instances, Redis, Cloud CDN |
| 10,000+ | K8s full power unlocked | May hit Cloud Run limits → migrate to GKE |

---

## 5. Recommendation: Cloud Run (Lightweight)

**Manaslu already chose Cloud Run for documented reasons** (doc 09 explicitly rejects GKE). Saathi should follow the same pattern for consistency. The $145/mo saving at beta scale matters for a solo-built product.

### Cloud Run Deployment Matrix

| Service | Image | CPU/Mem | Min | Max | Concurrency | Timeout | Public? |
|---------|-------|---------|-----|-----|-------------|---------|---------|
| saathi-web | `saathi-web:latest` | 256m / 512Mi | 0 | 10 | 80 | 5m | ✅ Yes |
| saathi-api | `saathi-api:latest` | 512m / 1Gi | 0 | 10 | 40 | 15m | ✅ Yes |
| manaslu-agent | `manaslu-agent:latest` | 1 / 1Gi | 1 (prod) | 5 | 20 | 60m | ❌ IAM only |
| saathi-knowledge | `saathi-knowledge:latest` | 1 / 2Gi | Job | Job | 1 | 30m | ❌ IAM only |

### Environment Strategy

```
saathi-dev:     all services min=0, Cloud SQL smallest tier, $40/mo (mostly idle)
saathi-prod:    manaslu min=1 (SSE latency), others min=0, $148/mo
```

### Migration Path: Cloud Run → GKE (if needed)

| Trigger | When | Migration Step |
|---------|------|---------------|
| >10K MAU sustained | Cloud Run concurrency limits hit | Convert Cloud Run YAML → K8s Deployment YAML (similar shape) |
| Need NetworkPolicy | Multi-tenant concerns arise | Containerize same images into GKE pods |
| Cold start unacceptable | 90th percentile latency >3s | Set min-instances > 0 (Cloud Run) before migrating |
| Multi-cloud required | Business requirement, not technical | GKE → GKE-compatible distro (Anthos/EKS/AKS) |

### What You DON'T Need Yet

| Resource | Why Not Now | When To Add |
|----------|-------------|-------------|
| Cloud CDN | <100 users, static assets small | 1K+ MAU |
| Redis (Memorystore) | No caching layer needed at beta | 1K+ MAU, API response caching |
| Cloud Load Balancer (standalone) | Cloud Run provides ingress | 10K+ MAU, multi-region |
| Cloud Armor (WAF/DDoS) | Only trusted users at beta | Public launch |
| BigQuery (cost analytics) | Premature — Claude usage log is sufficient | When AI costs > $100/mo |
| Apigee / Cloud Endpoints | Only 1 consumer (Saathi) | When manaslu has 2+ consumers |
| Cloud SQL read replica | Write volume negligible | 1K+ MAU, read-heavy queries |
| VPC Service Controls | Data residency enforced by region | When compliance audit requires it |
