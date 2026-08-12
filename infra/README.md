# Saathi — Infrastructure Documentation

**Location:** `saathi/infra/` | **IaC Repo:** `karki-labs-infra`  
**Cloud:** GCP | **Region:** australia-southeast1 (Sydney)

---

## Decision: Cloud Run, not GKE

`infrastructure-comparison.md` did the full GKE-vs-Cloud-Run analysis and
**recommends Cloud Run** — matching manaslu's own doc 09 (which already
rejected GKE) and saving ~$145/mo at beta scale. This is the settled compute
platform; everything below reflects it. `terraform-resource-map.md`
(GKE-oriented) and `infrastructure.md` (original GKE-only plan) are both
archived under `archive/` — kept only in case a future migration to GKE is
ever triggered (see the comparison doc's §"Migration Path" for the actual
triggers: >10K sustained MAU, NetworkPolicy needs, or a multi-cloud
requirement — none apply yet).

## Files

| Doc | Lines | What It Contains |
|-----|-------|-----------------|
| [infrastructure-comparison.md](./infrastructure-comparison.md) | 316 | **GKE vs Cloud Run** side-by-side comparison — cost tables, feature matrix, scalability thresholds, and the Cloud Run resource inventory (§3) + deployment matrix (§5) to build from |
| [tasks-infra.md](./tasks-infra.md) | Cloud Run-targeted | Prioritized Terraform/IaC implementation tasks |
| [manaslu-form-filling-investigation.md](./manaslu-form-filling-investigation.md) | 668 | Extraction approach comparison — hybrid/tiered recommendation, now merged into manaslu docs 02/03 |
| [manaslu-plan-audit.md](./manaslu-plan-audit.md) | 320 | Build-plan gap audit — mostly already incorporated into manaslu doc 11 |
| [manaslu-production-audit.md](./manaslu-production-audit.md) | 400 | Production-readiness critique — mostly already incorporated into manaslu doc 11 |
| [archive/terraform-resource-map.md](./archive/terraform-resource-map.md) | 690 | Superseded GKE resource map — reference only if migrating off Cloud Run later |
| [archive/infrastructure.md](./archive/infrastructure.md) | 307 | Superseded original GKE-only plan |

---

## Quick Reference

### Service Inventory

| Service | Repo | Runtime | Public? |
|---------|------|---------|---------|
| saathi-web | `judasprabin/saathi` | Next.js PWA (Cloud Run) | Yes |
| saathi-api | `judasprabin/saathi` | FastAPI (Cloud Run) | Yes |
| manaslu-agent | `judasprabin/manaslu` | FastAPI gap-resolution engine (Cloud Run) | IAM only |
| saathi-knowledge | `judasprabin/saathi` | Python batch (Cloud Run Job, via Cloud Scheduler) | IAM only |

### Key GCP Resources (Cloud Run — see infrastructure-comparison.md §3 for full detail)

| Resource | Spec | Monthly Cost |
|----------|------|-------------|
| Cloud Run services (web, api, manaslu-agent) | Scale-to-zero except manaslu-agent (min 1 in prod, for SSE latency) | ~$39/mo |
| Cloud Run Job (saathi-knowledge) | 1 CPU/2Gi, ~2h/day, triggered by Cloud Scheduler | ~$6/mo |
| Cloud SQL | db-custom-1-3840, 50GB SSD, PITR | ~$55/mo |
| Serverless VPC Connector | 1 connector (Cloud Run → Cloud SQL private IP) — replaces Cloud NAT, which Cloud Run doesn't need | ~$35/mo |
| GCS | 2 buckets, ~10GB total | ~$1.50/mo |
| All other (DNS, Artifact Registry, Secrets, Monitoring, Build) | — | ~$12/mo |
| **Total** | | **~$148/mo** |

### Cost Tiers

| Tier | Monthly | Use Case |
|------|---------|----------|
| Dev | ~$40/mo | All services min=0, Cloud SQL smallest tier |
| **Beta Prod** | **~$148/mo** | manaslu-agent min=1 (SSE latency), others min=0 |
| At scale (10K+ MAU) | Revisit — see comparison doc's Cloud Run→GKE migration triggers | |

### Terraform Execution (quick order)

```
1. Project + APIs → 2. VPC + Subnet → 3. Serverless VPC Connector →
4. Cloud SQL → 5. GCS → 6. IAM (per-service SAs) → 7. Secrets →
8. Cloud Run services + Job → 9. Cloud Scheduler → 10. DNS + TLS →
11. Artifact Registry → 12. Auth (Identity Platform) → 13. Monitoring
```

Full task breakdown: [tasks-infra.md](./tasks-infra.md).
