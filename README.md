# Azure Synapse → Microsoft Fabric Migration Tracker

A product management tool for tracking, prioritising, and managing customer migrations away from deprecated Azure Synapse Analytics workspaces.

Built to demonstrate end-to-end PM thinking on a real Microsoft deprecation scenario — from telemetry-based identification to structured outreach to playbook execution.

**Live demo:** [your-github-pages-url-here]

---

## Why I built this

Synapse → Fabric migration problem is essentially the same challenge at 100x scale. I built this to think through the problem properly — not just as a conceptual exercise but as something operationally useful.
---

## What the tool does

**Overview dashboard**
Migration completion by tier and region. Timeline of the deprecation phases. At-a-glance KPIs: total tenants, migrated count, at-risk count, days to EOL.

**Telemetry view**
Shows which tenants are still making daily Synapse API calls. Sorted by usage volume. The trend chart shows Synapse usage declining as Fabric adoption grows — confirming migration is happening but not fast enough.

**Customer table**
Full tenant list with migration %, contact status, next action. Searchable and filterable. Built to replicate what a PM would actually need in front of them during an outreach sprint.

**SQL + KQL queries**
Four real queries I wrote to power this data model:
- SQL: Find tenants still on deprecated API version (last 30 days)
- SQL: Segment tenants by migration urgency (CRITICAL / HIGH / MEDIUM / LOW)
- KQL: Pull active Synapse workspaces from Azure Monitor
- KQL: Compare Synapse vs Fabric API call volumes per tenant

**Outreach queue**
Draft emails generated from telemetry data — not generic, but referencing each tenant's actual API call count, pipeline runs, and days to EOL. Includes send tracking and edit functionality.

**Deprecation playbook**
Six-step structured process: telemetry pull → urgency segmentation → first outreach → follow-up cadence → migration validation → post-migration support. Designed to scale without requiring manual effort per customer.

---

## Queries

### SQL — Identify tenants on deprecated API
```sql
SELECT
    t.tenant_id,
    t.org_name,
    t.subscription_tier,
    t.region,
    COUNT(a.call_id)           AS api_calls_30d,
    MAX(a.timestamp)           AS last_seen,
    m.migration_pct
FROM   api_usage a
JOIN   tenants t  ON a.tenant_id = t.tenant_id
LEFT JOIN migration_status m ON m.tenant_id = t.tenant_id
WHERE  a.api_version   = 'synapse-rest-v2'
  AND  a.timestamp    >= DATEADD(day, -30, GETDATE())
  AND  m.migration_pct < 100
GROUP BY
    t.tenant_id, t.org_name,
    t.subscription_tier, t.region,
    m.migration_pct
ORDER BY api_calls_30d DESC;
```

### KQL — Active Synapse workspaces via Azure Monitor
```kusto
AzureDiagnostics
| where TimeGenerated >= ago(30d)
| where ResourceType == "MICROSOFT.SYNAPSE/WORKSPACES"
| where OperationName has_any ("Pipeline", "SparkPool", "SQLPool")
| summarize
    TotalOperations   = count(),
    LastSeen          = max(TimeGenerated),
    UniqueOperations  = dcount(OperationName)
    by SubscriptionId, ResourceGroup
| where TotalOperations > 0
| order by TotalOperations desc
```

---

## Stack

- HTML / CSS / vanilla JS — no framework, intentionally lightweight
- Chart.js — trend visualisation
- Azure SQL (T-SQL) — customer segmentation queries
- Kusto (KQL) — Azure Monitor telemetry queries
- GitHub Pages — hosting

---

## Data note

All tenant data in the demo is synthetic. Org names, tenant IDs, API call volumes, and migration percentages are generated for demonstration purposes. Query logic and schema design reflect real Azure Monitor / Azure SQL patterns.

---

## Folder structure

```
/
├── index.html          # Main app
├── README.md           # This file
└── docs/
    └── playbook.md     # Deprecation playbook (standalone)
```

---
