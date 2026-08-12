# Saathi — Terraform Infrastructure Resource Map (GKE Enterprise)

**Version:** 1.0 | **Target:** `karki-labs-infra` repo  
**Region:** australia-southeast1 | **Tool:** Terraform (GCP provider)  
**Approach:** GKE Standard (not Autopilot — for full control + cost visibility)

---

## How to Use This Document

Every table = a Terraform resource you create. Follow top-to-bottom — earlier resources unblock later ones. Each table has the exact `google_*` resource name, key settings, and money-saving alternatives.

**Money-saving icons:**
- 💰 = Significant cost impact — evaluate carefully
- 🆓 = Free tier available — use it
- ⚡ = Can defer to Phase 2

---

## Phase 0 — GCP Project & APIs

### P0.1 Project

| Terraform Resource | Setting | Value | Notes |
|---|---|---|---|
| `google_project` | project_id | `saathi-prod` | Or `karki-labs-saathi` — pick one and lock it |
| | name | `Saathi Production` | |
| | billing_account | Your billing account ID | 💰 Billing alert at $200/mo — set BEFORE creating resources |
| | auto_create_network | `false` | Never use default VPC |

```hcl
# terraform.tf
provider "google" {
  project = "saathi-prod"
  region  = "australia-southeast1"
}

terraform {
  backend "gcs" {
    bucket = "saathi-tfstate"  # create this manually first
    prefix = "terraform/state"
  }
}
```

### P0.2 APIs to Enable

| Terraform Resource | API | Why |
|---|---|---|
| `google_project_service` | `compute.googleapis.com` | VPC, GKE nodes, Cloud NAT |
| | `container.googleapis.com` | GKE cluster |
| | `sqladmin.googleapis.com` | Cloud SQL |
| | `sql-component.googleapis.com` | Cloud SQL (newer) |
| | `storage.googleapis.com` | GCS buckets |
| | `secretmanager.googleapis.com` | API keys + DB passwords |
| | `artifactregistry.googleapis.com` | Docker images |
| | `cloudbuild.googleapis.com` | CI/CD |
| | `dns.googleapis.com` | DNS zone |
| | `monitoring.googleapis.com` | Logs, metrics, alerts |
| | `logging.googleapis.com` | Structured logging |
| | `identitytoolkit.googleapis.com` | Auth |
| | `cloudscheduler.googleapis.com` | Knowledge CronJob trigger |
| | `run.googleapis.com` | Only if you later add Cloud Run services |
| | `iamcredentials.googleapis.com` | Workload Identity |

💰 **Tip:** APIs are free to enable — you pay only for the resources they create.

---

## Phase 1 — Networking

### P1.1 VPC + Subnet

| Terraform Resource | Setting | Value | Why This Value |
|---|---|---|---|
| `google_compute_network` | name | `saathi-vpc` | |
| | auto_create_subnetworks | `false` | We create explicit subnets |
| | routing_mode | `REGIONAL` | Single region — simpler |
| `google_compute_subnetwork` | name | `saathi-subnet` | |
| | network | `saathi-vpc` | |
| | ip_cidr_range | `10.0.0.0/20` | 4,096 IPs — enough for GKE pods + services + Cloud SQL |
| | region | `australia-southeast1` | |
| | private_ip_google_access | `true` | Pods reach GCP APIs without external IP |
| | secondary_ip_range { name="pods" } | `10.4.0.0/14` | 262,144 pod IPs — GKE requirement |
| | secondary_ip_range { name="services" } | `10.0.32.0/20` | 4,096 service IPs — GKE requirement |

💰 **Alternative:** If you're truly budget-constrained, use `10.0.0.0/24` for primary (256 IPs) and skip the secondary ranges — but then you can't use VPC-native GKE (you'd need routes-based, which is legacy). Don't do this.

### P1.2 Cloud Router + NAT

| Terraform Resource | Setting | Value | Notes |
|---|---|---|---|
| `google_compute_router` | name | `saathi-router` | |
| | network | `saathi-vpc` | |
| | region | `australia-southeast1` | |
| `google_compute_router_nat` | name | `saathi-nat` | |
| | router | `saathi-router` | |
| | nat_ip_allocate_option | `AUTO_ONLY` | Google-managed IPs — no static IP cost |
| | source_subnetwork_ip_ranges_to_nat | `ALL_SUBNETWORKS_ALL_IP_RANGES` | All pods can reach internet (Anthropic, Voyage) |

💰 **Cost:** Cloud NAT = ~$42/mo for the gateway + data processed. This is the single biggest non-database cost.
💰 **Alternative:** Use `AUTO_ONLY` (free IPs). If you go Cloud Run, you don't need NAT at all (direct egress). But with GKE private nodes, NAT is required.

### P1.3 Firewall Rules

| Terraform Resource | Setting | Value | Why |
|---|---|---|---|
| `google_compute_firewall` | name | `allow-gke-health-check` | |
| | network | `saathi-vpc` | |
| | direction | `INGRESS` | |
| | source_ranges | `[130.211.0.0/22, 35.191.0.0/16]` | GKE health check ranges |
| | allow { protocol="tcp", ports=["80","443"] } | | |
| `google_compute_firewall` | name | `allow-ssh-from-iap` | |
| | source_ranges | `[35.235.240.0/20]` | IAP tunnel range (for debugging only) |
| | allow { protocol="tcp", ports=["22"] } | | |
| `google_compute_firewall` | name | `allow-cloud-sql` | |
| | source_tags | `["gke-node"]` | |
| | allow { protocol="tcp", ports=["5432"] } | | Cloud SQL → GKE |
| | target_tags | `["cloud-sql"]` | | Only if using public IP (don't — use private) |

💰 **FREE** — firewall rules cost nothing.

---

## Phase 2 — GKE Cluster

### P2.1 Service Account (GKE nodes)

| Terraform Resource | Setting | Value |
|---|---|---|
| `google_service_account` | account_id | `sa-gke-node` |
| | display_name | `GKE Node SA` |
| `google_project_iam_member` | role | `roles/logging.logWriter` |
| | member | `sa-gke-node@...` |
| | role | `roles/monitoring.metricWriter` |
| | role | `roles/monitoring.viewer` |
| | role | `roles/storage.objectViewer` (for pulling images from Artifact Registry) |

### P2.2 GKE Cluster

| Terraform Resource | Setting | Value | Why This Value |
|---|---|---|---|
| `google_container_cluster` | name | `saathi-cluster` | |
| | location | `australia-southeast1` | Regional cluster (3 zones) |
| | **remove_default_node_pool** | `true` | We create managed node pools below |
| | initial_node_count | `1` | Placeholder — removed immediately |
| | **networking_mode** | `VPC_NATIVE` | Required for private nodes + secondary ranges |
| | network | `saathi-vpc` | |
| | subnetwork | `saathi-subnet` | |
| | **private_cluster_config { enable_private_nodes** } | `true` | 💰 Nodes have only private IPs — no public exposure |
| | private_cluster_config { enable_private_endpoint } | `false` | Keep control plane endpoint accessible for kubectl |
| | private_cluster_config { master_ipv4_cidr_block } | `172.16.0.0/28` | Control plane CIDR |
| | **ip_allocation_policy** | Use secondary ranges from P1.1 | |
| | release_channel { channel } | `REGULAR` | Stable releases, not RAPID |
| | workload_identity_config { workload_pool } | `saathi-prod.svc.id.goog` | Required for WI |
| | monitoring_service | `monitoring.googleapis.com/kubernetes` | |
| | logging_service | `logging.googleapis.com/kubernetes` | |
| | deletion_protection | `true` | Prevent accidental deletion |

💰 **GKE cluster management fee: $0/hour for one zonal, $0.10/hour for regional**  
💰 **Regional = $73/mo cluster fee.** Zonal = $0/mo but single-AZ (not recommended for prod).  

**Money-saving decision tree:**
```
Beta (<100 users) → Zonal (australia-southeast1-a only) → $0/mo cluster fee ✅
1K+ users → Regional → $73/mo for HA
```
💰 **Recommendation: START ZONAL.** You're beta with <500 users. Regional adds $73/mo for risk you don't have yet.

### P2.3 Node Pool

| Terraform Resource | Setting | Value | Why |
|---|---|---|---|
| `google_container_node_pool` | name | `saathi-pool` | |
| | cluster | `saathi-cluster` | |
| | location | `australia-southeast1-a` | Zonal (single AZ) |
| | **node_count** | `2` | Minimum for rolling updates |
| | **autoscaling { min=2, max=6 }** | | Scale with load |
| | management { auto_repair=true, auto_upgrade=true } | | Hands-off node maintenance |
| | node_config { **machine_type** } | `e2-standard-2` | 2 vCPU / 8 GB RAM |
| | node_config { **disk_size_gb** } | `50` | Balanced PD |
| | node_config { **disk_type** } | `pd-balanced` | 💰 SSD not needed for GKE nodes |
| | node_config { **preemptible** } | `false` | 💰 Could be true for dev — saves 80%, nodes die every 24h |
| | node_config { oauth_scopes } | `cloud-platform` scope | |
| | node_config { service_account } | `sa-gke-node@...` | |
| | node_config { **tags** } | `["gke-node"]` | For firewall rules |
| | node_config { labels } | `{env="prod"}` | |

💰 **Cost: 2 × e2-standard-2 × 730h × $0.067 = ~$98/mo for nodes**  
💰 **Alternative: `e2-medium` (1 vCPU / 4 GB) = $49/mo** — sufficient for beta if pods are light  
💰 **Alternative: `e2-small` (0.5 vCPU / 2 GB) = $24/mo** — risky, only for dev staging  
💰 **Preemptible nodes = $0.020/hr = ~$29/mo** — but node dies every 24h, pods restart. OK for stateless services (web, api), NOT OK for manaslu-agent (active SSE sessions die).

**Recommendation:**
```
Dev:      e2-small × 2 preemptible = $10/mo
Staging:  e2-medium × 2 = $49/mo
Prod:     e2-standard-2 × 2 = $98/mo (non-preemptible for manaslu SSE)
```

### P2.4 K8s Provider (after cluster exists)

```hcl
data "google_client_config" "default" {}

provider "kubernetes" {
  host  = "https://${google_container_cluster.saathi.endpoint}"
  token = data.google_client_config.default.access_token
  cluster_ca_certificate = base64decode(
    google_container_cluster.saathi.master_auth[0].cluster_ca_certificate
  )
}
```

---

## Phase 3 — Cloud SQL (PostgreSQL)

### P3.1 Instance

| Terraform Resource | Setting | Value | Why |
|---|---|---|---|
| `google_sql_database_instance` | name | `saathi-db` | |
| | region | `australia-southeast1` | |
| | database_version | `POSTGRES_16` | Required for pgvector extension |
| | **tier** | `db-custom-1-3840` | 1 vCPU / 3.75 GB RAM |
| | **disk_size** | `50` GB | Auto-grow enabled below |
| | disk_autoresize | `true` | |
| | disk_autoresize_limit | `200` GB | Cap expansion |
| | disk_type | `PD_SSD` | SSD for pgvector index performance |
| | **availability_type** | `ZONAL` | 💰 Regional ($110/mo) vs Zonal ($55/mo) |
| | deletion_protection | `true` | |
| | ip_configuration { ipv4_enabled } | `false` | **Only private IP** |
| | ip_configuration { private_network } | `saathi-vpc` | Access from GKE via VPC |
| | ip_configuration { enable_private_path } | `true` | |
| | backup_configuration { enabled=true } | `true` | |
| | backup_configuration { start_time } | `03:00` | 3 AM AEST |
| | backup_configuration { point_in_time_recovery } | `true` | PITR — can restore to any point |
| | backup_configuration { transaction_log_retention_days } | `7` | 7 days of PITR |
| | maintenance_window { day=7, hour=4 } | | Sunday 4 AM AEST |

💰 **Cost: $55/mo (zonal) vs $110/mo (regional/HA)**  
💰 **Savings: `db-f1-micro` (shared core, 0.6 GB RAM) = $9/mo** — works for dev, NOT for pgvector queries  
💰 **Savings: `db-g1-small` (shared core, 1.7 GB RAM) = $27/mo** — borderline for pgvector + 2 databases

**Money-saving strategy:**
```
Dev:      db-f1-micro, no backups, no PITR = ~$9/mo
Staging:  db-g1-small, daily backups = ~$30/mo
Prod:     db-custom-1-3840, daily backups + PITR = ~$55/mo
```

### P3.2 Databases + Users

| Terraform Resource | Setting | Value |
|---|---|---|
| `google_sql_database` | name | `saathi` |
| | instance | `saathi-db` |
| `google_sql_database` | name | `manaslu` |
| | instance | `saathi-db` |
| `google_sql_user` | name | `saathi-app` |
| | instance | `saathi-db` |
| | password | From Secret Manager (random_password) |
| `google_sql_user` | name | `manaslu-app` |
| | instance | `saathi-db` |
| | password | From Secret Manager |
| `random_password` | (2 separate resources) | 32 chars, special=true |
| `google_secret_manager_secret_version` | (store both passwords) | |

### P3.3 pgvector Extension

**Not Terraform-managed** — run as SQL migration in saathi repo:

```sql
CREATE EXTENSION IF NOT EXISTS vector;
CREATE EXTENSION IF NOT EXISTS pg_cron;
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
```

---

## Phase 4 — Storage (GCS)

### P4.1 Buckets

| Terraform Resource | Setting | Value | Why |
|---|---|---|---|
| `google_storage_bucket` | name | `saathi-user-documents` | |
| | location | `AUSTRALIA-SOUTHEAST1` | Data residency |
| | **storage_class** | `STANDARD` | 💰 Could be NEARLINE for cost savings |
| | uniform_bucket_level_access | `true` | Required for IAM-based access |
| | public_access_prevention | `enforced` | Never public |
| | versioning { enabled } | `true` | Recovery from accidental deletion |
| `google_storage_bucket` | name | `saathi-filled-artifacts` | |
| | (same settings as above) | | |
| `google_storage_bucket` | name | `saathi-tfstate` | Terraform state (create manually FIRST) |
| | versioning | `true` | Required for state locking |

### P4.2 Lifecycle Policies

| Terraform Resource | Setting | Value |
|---|---|---|
| `google_storage_bucket.lifecycle_rule` | condition { age } | `365` days |
| (user-documents) | action { type } | `Delete` |
| (filled-artifacts) | condition { age } | `365` days |
| | action { type } | `Delete` |

💰 **Tip:** Set to 90 days for staging, 365 for prod. NEARLINE storage class = 50% cheaper but retrieval costs apply.

---

## Phase 5 — IAM & Workload Identity

### P5.1 Service Accounts (per workload)

| Terraform Resource | Setting | Value |
|---|---|---|
| `google_service_account` | account_id | `sa-web` |
| | display_name | `Saathi Web SA` |
| `google_service_account` | account_id | `sa-api` |
| `google_service_account` | account_id | `sa-knowledge` |
| `google_service_account` | account_id | `manaslu-agent` |

### P5.2 IAM Bindings (least privilege)

| Terraform Resource | SA | Role | Why |
|---|---|---|---|
| `google_project_iam_member` | sa-api | `roles/cloudsql.client` | Connect to Cloud SQL |
| | sa-api | `roles/storage.objectUser` | Read/write GCS |
| | sa-api | `roles/secretmanager.secretAccessor` | Read API keys |
| | sa-api | `roles/run.invoker` | Call manaslu (Cloud Run) |
| | sa-knowledge | `roles/cloudsql.client` | Write to pgvector |
| | sa-knowledge | `roles/secretmanager.secretAccessor` | Read Voyage/Notion keys |
| | manaslu-agent | `roles/cloudsql.client` | Read/write manaslu DB |
| | manaslu-agent | `roles/storage.objectUser` | Read/write GCS |
| | manaslu-agent | `roles/secretmanager.secretAccessor` | Read Anthropic key |

### P5.3 Workload Identity (K8s SA → GCP SA)

| Terraform Resource | K8s SA | GCP SA | Namespace |
|---|---|---|---|
| `google_service_account_iam_binding` | sa-web | sa-web@... | saathi |
| | sa-api | sa-api@... | saathi |
| | sa-knowledge | sa-knowledge@... | saathi |
| | manaslu-agent | manaslu-agent@... | manaslu |

```hcl
resource "google_service_account_iam_binding" "wi-sa-api" {
  service_account_id = google_service_account.sa-api.name
  role               = "roles/iam.workloadIdentityUser"
  members = [
    "serviceAccount:saathi-prod.svc.id.goog[saathi/sa-api]",
  ]
}
```

**This is the cleanest auth pattern in GCP** — pods get GCP permissions without any key files. The K8s SA annotation does the rest:

```yaml
# In the K8s Deployment pod spec
serviceAccountName: sa-api
# Pod automatically gets sa-api@saathi-prod.iam.gserviceaccount.com permissions
```

---

## Phase 6 — Secret Manager

### P6.1 Secrets

| Terraform Resource | Setting | Value | Rotation |
|---|---|---|---|
| `google_secret_manager_secret` | secret_id | `anthropic-api-key` | 💰 Manual (no auto-rotation needed) |
| | replication { auto {} } | | |
| `google_secret_manager_secret` | secret_id | `voyage-api-key` | Manual |
| | secret_id | `notion-integration-token` | Manual |
| | secret_id | `db-password-saathi` | Rotate quarterly |
| | secret_id | `db-password-manaslu` | Rotate quarterly |
| | secret_id | `fcm-server-key` | Manual |

| `google_secret_manager_secret_version` | For each secret above | Value from `.tfvars` (NEVER in code) | |

💰 **Cost: $0.06/secret/month × 6 = $0.36/mo + $0.03/10K access ops** — negligible.

### P6.2 External Secrets Operator (K8s → Secret Manager)

| Resource | Setting | Value |
|---|---|---|
| Helm release `external-secrets` | chart | `external-secrets/external-secrets` |
| `ClusterSecretStore` | provider | `gcpsm` |
| | project | `saathi-prod` |
| `ExternalSecret` (per secret) | target | K8s Secret name |
| | secretKey | The key inside the K8s Secret |

This keeps K8s Secrets in sync with Secret Manager — no hardcoded values in Git.

---

## Phase 7 — DNS + TLS

### P7.1 Cloud DNS

| Terraform Resource | Setting | Value |
|---|---|---|
| `google_dns_managed_zone` | name | `saathi-zone` |
| | dns_name | `saathi.app.` (trailing dot) |
| | visibility | `public` |

| `google_dns_record_set` | name | `saathi.app.` |
| | type | `A` |
| | ttl | `300` |
| | rrdatas | `[GKE_LB_IP]` |

| `google_dns_record_set` | name | `api.saathi.app.` |
| | type | `A` |
| | rrdatas | `[GKE_LB_IP]` |

| `google_dns_record_set` | name | `www.saathi.app.` |
| | type | `CNAME` |
| | rrdatas | `["saathi.app."]` |

💰 **Cost: $1/mo for zone + $0.40/million queries** — negligible.

### P7.2 Managed Certificate (TLS)

| Terraform Resource | Setting | Value |
|---|---|---|
| `google_certificate_manager_dns_authorization` | name | `saathi-auth` |
| | domain | `saathi.app` |
| `google_certificate_manager_certificate` | name | `saathi-cert` |
| | managed { domains=["saathi.app", "api.saathi.app", "*.saathi.app"] } | |
| | managed { dns_authorizations } | Reference above |

🆓 **Google-managed certificates are FREE** — no Let's Encrypt or paid CA needed.

---

## Phase 8 — Artifact Registry + CI/CD

### P8.1 Registry

| Terraform Resource | Setting | Value |
|---|---|---|
| `google_artifact_registry_repository` | repository_id | `saathi-docker` |
| | format | `DOCKER` |
| | location | `australia-southeast1` |
| | cleanup_policy { action="DELETE", condition { tag_state="UNTAGGED", older="7d" } } | |

💰 **Cost: $0.10/GB/month × ~5GB = $0.50/mo** — negligible.

### P8.2 CI/CD Service Account

| Terraform Resource | Setting | Value |
|---|---|---|
| `google_service_account` | account_id | `sa-cicd` | For Cloud Build or GitHub Actions |
| `google_project_iam_member` | role | `roles/container.developer` | Push to GKE |
| | role | `roles/artifactregistry.writer` | Push images |
| | role | `roles/cloudsql.client` | Run migrations |
| `google_service_account_iam_binding` | WIF binding | GitHub repo → sa-cicd | |

```hcl
# Workload Identity Federation: GitHub Actions → GCP (no keys!)
resource "google_iam_workload_identity_pool" "github" {
  workload_identity_pool_id = "github-pool"
}

resource "google_iam_workload_identity_pool_provider" "github" {
  workload_identity_pool_id = google_iam_workload_identity_pool.github.id
  workload_identity_pool_provider_id = "github-provider"
  attribute_mapping = {
    "google.subject" = "assertion.sub"
    "attribute.repository" = "assertion.repository"
  }
  oidc {
    issuer_uri = "https://token.actions.githubusercontent.com"
  }
}
```

🆓 **WIF = free. No service account keys to rotate ever.**

---

## Phase 9 — Identity Platform (Auth)

| Terraform Resource | Setting | Value |
|---|---|---|
| `google_identity_platform_config` | autodelete_anonymous_users | `false` |
| | sign_in { email { enabled=true, password_required=true } } | |
| | sign_in { phone_number { enabled=false } } | No SMS at MVP |
| | sign_in { anonymous { enabled=false } } | |
| | authorized_domains | `["saathi.app"]` | |

**OAuth providers** (configure via Console, not Terraform — one-time setup):
- Google OAuth: Client ID + Secret → Secret Manager
- Apple OAuth (Phase 2 — deferred until iOS app)

| Setting | Value |
|---|---|
| `google_identity_platform_default_supported_idp_config` | Google OAuth |
| | client_id | From GCP Console |
| | client_secret | From Secret Manager |

🆓 **Identity Platform is FREE for <50K MAU** — don't overthink this.

### Firebase Setup (for FCM Push + Auth compatibility)

| Terraform Resource | Setting | Value |
|---|---|---|
| `google_firebase_project` | project | `saathi-prod` |
| `google_firebase_web_app` | display_name | `Saathi Web` |

Identity Platform = the GCP name for Firebase Auth. Same product, same pricing.

---

## Phase 10 — Monitoring + Alerting

### P10.1 Uptime Checks

| Terraform Resource | Setting | Value |
|---|---|---|
| `google_monitoring_uptime_check_config` | display_name | `saathi-api-health` |
| | http_check { path="/health", port=443, use_ssl=true } | |
| | monitored_resource { type="uptime_url", labels={host="api.saathi.app"} } | |
| | period | `60s` |
| | timeout | `10s` |

| `google_monitoring_uptime_check_config` | display_name | `saathi-web-health` |
| | (same) | host=`saathi.app` |

🆓 **Uptime checks are FREE (within limits)** — up to 1M executions/month.

### P10.2 Alerting Policies

| Terraform Resource | Setting | Value | Trigger |
|---|---|---|---|
| `google_monitoring_alert_policy` | name | `High API Error Rate` | 5xx rate > 5% for 5 min |
| | Notification: Email | | |
| `google_monitoring_alert_policy` | name | `High API Latency` | P95 > 500ms for 5 min |
| `google_monitoring_alert_policy` | name | `Pod Crash Loop` | K8s pod restarts > 3/hour |
| `google_monitoring_alert_policy` | name | `DB Storage > 80%` | Cloud SQL disk usage |
| `google_monitoring_alert_policy` | name | `DB Connections > 80%` | Connection pool saturation |

### P10.3 Notification Channel

| Terraform Resource | Setting | Value |
|---|---|---|
| `google_monitoring_notification_channel` | type | `email` |
| | labels { email_address } | Your email |

💰 **Email notifications are free. PagerDuty/Slack/webhook channels cost extra.**

---

## Phase 11 — K8s Resources (Deployed from each repo's CI, not Terraform)

These technically live in `saathi/k8s/` and `manaslu/k8s/`, but for completeness:

### K8s Namespaces

| Resource | Namespace | Purpose |
|---|---|---|
| `kubectl create namespace` | `saathi` | All saathi workloads |
| | `manaslu` | All manaslu workloads |
| | `monitoring` | Observability tools |

### K8s Deployments

| Resource | Namespace | Image | Replicas | CPU Req/Lim | Mem Req/Lim |
|---|---|---|---|---|---|
| Deployment `saathi-web` | saathi | `saathi-web:latest` | 2 / 6 | 256m / 1 | 512Mi / 1Gi |
| Deployment `saathi-api` | saathi | `saathi-api:latest` | 2 / 4 | 512m / 2 | 1Gi / 2Gi |
| Deployment `manaslu-agent` | manaslu | `manaslu-agent:latest` | 1 / 3 | 1 / 4 | 2Gi / 4Gi |
| CronJob `saathi-knowledge` | saathi | `saathi-knowledge:latest` | Job | 1 / 2 | 2Gi / 4Gi |
| Schedule: `0 21 * * *` (7am AEST from UTC+10) | | | | | |

### K8s Services

| Resource | Namespace | Type | Port |
|---|---|---|---|
| Service `saathi-web-svc` | saathi | ClusterIP | 3000 |
| Service `saathi-api-svc` | saathi | ClusterIP | 8000 |
| Service `manaslu-svc` | manaslu | ClusterIP | 8000 |

### K8s NetworkPolicy

| Resource | Namespace | Rule |
|---|---|---|
| NetworkPolicy `allow-saathi-to-manaslu` | manaslu | Ingress: from namespace=saathi, port=8000 |

### K8s ConfigMap

| Resource | Namespace | Key | Value |
|---|---|---|---|
| ConfigMap `saathi-config` | saathi | `API_BASE_URL` | `https://api.saathi.app` |
| | | `MANASLU_INTERNAL_URL` | `http://manaslu-svc.manaslu:8000` |
| | | `GCS_BUCKET` | `saathi-user-documents` |
| | | `DB_HOST` | Cloud SQL private IP |

### K8s Gateway (Ingress)

| Resource | Setting | Value |
|---|---|---|
| GatewayClass | name | `gke-l7-gxlb` | GKE Gateway controller |
| Gateway | name | `saathi-gateway` | |
| | listeners { name="http", port=80, protocol="HTTP" } | Redirect to HTTPS |
| | listeners { name="https", port=443, protocol="HTTPS", tls { certificateRefs } } | |
| HTTPRoute `web` | hostnames | `["saathi.app", "www.saathi.app"]` |
| | backendRefs | `saathi-web-svc:3000` |
| HTTPRoute `api` | hostnames | `["api.saathi.app"]` |
| | backendRefs | `saathi-api-svc:8000` |

---

## Phase 12 — Money-Saving Summary

### Cost Tier Matrix

| Tier | Monthly Cost | What You Get | When To Use |
|------|-------------|-------------|-------------|
| **Dev Minimum** | **$65/mo** | Zonal GKE (e2-small×2 preemptible), db-f1-micro, 10GB GCS, no PITR, no monitoring alerts | Local dev and CI testing only |
| **Staging** | **$160/mo** | Zonal GKE (e2-medium×2), db-g1-small + backups, 25GB GCS, basic monitoring | Pre-launch testing |
| **Beta Prod** | **$275/mo** | Zonal GKE (e2-standard-2×2), db-custom-1-3840 + PITR, 50GB GCS, full monitoring | Your current target |
| **Prod HA** | **$400/mo** | Regional GKE ($73), Regional Cloud SQL ($110), same compute | When you have paying users |

### Quick Wins (Save $60-100/mo immediately)

| What | Current | Change | Saves |
|------|---------|--------|-------|
| GKE cluster | Regional ($73/mo) | Zonal ($0/mo) | **-$73/mo** |
| Node type | e2-standard-2 ($98/mo) | e2-medium ($49/mo) | **-$49/mo** |
| Cloud SQL | db-custom-1-3840 ($55) | db-g1-small ($27) | **-$28/mo** |
| | | | **Total: -$150/mo** |

### What NOT To Skimp On

| Resource | Why | Minimum |
|----------|-----|---------|
| Cloud SQL backups | You will need them | Daily backups, 7-day retention |
| Node count | Rolling updates need 2 nodes minimum | min=2 |
| GCS versioning | Accidental deletion recovery | Enabled |
| Secret Manager | Never hardcode keys in Git | Always, no exceptions |
| Deletion protection | On cluster + DB | Always enabled |

---

## Phase 13 — Terraform Execution Order

```
1.  google_project                      (Create project)
2.  google_project_service              (Enable APIs — depends on project)
3.  google_compute_network              (VPC)
4.  google_compute_subnetwork           (Subnet + secondary ranges)
5.  google_compute_router               (Cloud Router)
6.  google_compute_router_nat           (Cloud NAT)
7.  google_compute_firewall             (Firewall rules)
8.  google_service_account (gke-node)   (GKE node SA)
9.  google_project_iam_member (node)    (Node permissions)
10. google_container_cluster            (GKE cluster)
11. google_container_node_pool          (Node pool)
12. google_sql_database_instance        (Cloud SQL)
13. google_sql_database                 (Databases)
14. random_password                     (DB passwords)
15. google_sql_user                     (DB users)
16. google_secret_manager_secret        (All secrets)
17. google_secret_manager_secret_version (Secret values)
18. google_storage_bucket               (GCS buckets)
19. google_service_account (workloads)   (4 SAs)
20. google_project_iam_member (workloads) (SA roles)
21. google_service_account_iam_binding   (Workload Identity)
22. google_artifact_registry_repository  (Docker registry)
23. google_dns_managed_zone              (DNS)
24. google_dns_record_set               (DNS records)
25. google_certificate_manager_*         (TLS certs)
26. google_monitoring_uptime_check_config (Uptime)
27. google_monitoring_alert_policy      (Alerts)
28. google_identity_platform_config     (Auth)
29. google_firebase_project             (Firebase)
30. kubectl_manifest / helm_release     (K8s resources — deployed from each repo's CI)

Terraform apply → 30 resources created → cluster ready for workloads
Then: CI/CD deploys saathi + manaslu images to GKE
```

---

*Resource map compiled: August 2026 | For: `karki-labs-infra` repo*  
*Pricing source: cloud.google.com/pricing (australia-southeast1 rates, August 2026)*
