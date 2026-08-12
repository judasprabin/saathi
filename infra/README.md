# Saathi — Infrastructure Documentation

**Location:** `saathi/infra/` | **IaC Repo:** `karki-labs-infra`  
**Cloud:** GCP | **Region:** australia-southeast1 (Sydney)

---

## Files

| Doc | Lines | What It Contains |
|-----|-------|-----------------|
| [terraform-resource-map.md](./terraform-resource-map.md) | 690 | **Complete GKE resource map** — every Terraform resource, settings, costs, and money-saving alternatives. Follow top-to-bottom to create all 30 GCP resources. |
| [infrastructure-comparison.md](./infrastructure-comparison.md) | 316 | **GKE vs Cloud Run** side-by-side comparison — cost tables, feature matrix, scalability thresholds, recommendation |
| [infrastructure.md](./infrastructure.md) | 307 | Original GKE-only infrastructure plan (superseded by the two above — kept for reference) |
| [tasks-infra.md](./tasks-infra.md) | 108 | Prioritized Terraform/IaC implementation tasks (30+ tasks across 4 phases) |

---

## Quick Reference

### Service Inventory

| Service | Repo | Runtime | Public? |
|---------|------|---------|---------|
| saathi-web | `judasprabin/saathi` | Next.js PWA | Yes |
| saathi-api | `judasprabin/saathi` | FastAPI | Yes |
| manaslu-agent | `judasprabin/manaslu` | FastAPI (gap-resolution engine) | IAM only |
| saathi-knowledge | `judasprabin/saathi` | Python batch (Cloud Run Job / K8s CronJob) | IAM only |

### Key GCP Resources

| Resource | Spec | Monthly Cost |
|----------|------|-------------|
| GKE Cluster | Zonal, australia-southeast1-a | $0/mo (zonal tier) |
| Node Pool | 2× e2-standard-2, auto-scaling to 6 | ~$98/mo |
| Cloud SQL | db-custom-1-3840, 50GB SSD, PITR | ~$55/mo |
| Cloud NAT | 1 gateway, AUTO_ONLY IPs | ~$42/mo |
| GCS | 3 buckets, ~10GB total | ~$2/mo |
| Serverless VPC Connector | 1 connector (for Cloud SQL private IP) | ~$35/mo |
| All other (DNS, Artifact Registry, Secrets, Monitoring) | — | ~$12/mo |
| **Total** | | **~$275/mo** |

### Cost Tiers

| Tier | Monthly | Use Case |
|------|---------|----------|
| Dev Minimum | $65/mo | Preemptible nodes, db-f1-micro, no backups |
| Staging | $160/mo | e2-medium nodes, db-g1-small, daily backups |
| **Beta Prod** | **$275/mo** | **e2-standard-2 nodes, db-custom-1-3840, PITR** |
| Prod HA | $400/mo | Regional GKE + Regional Cloud SQL |

### Terraform Execution (quick order)

```
1. Project + APIs → 2. VPC + Subnet → 3. NAT + Firewall → 4. GKE Cluster →
5. Node Pool → 6. Cloud SQL → 7. GCS → 8. IAM + WI → 9. Secrets →
10. DNS + TLS → 11. Artifact Registry → 12. Auth → 13. Monitoring
```

Full dependency graph with all 30 resources: [terraform-resource-map.md §13](./terraform-resource-map.md#phase-13--terraform-execution-order)
