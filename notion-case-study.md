# Project: Azure Synapse → Fabric Migration Tracker

**Type:** Side project / interview preparation
**Status:** Complete · v2.1
**Built:** June 2025
**Links:** [GitHub Repo] · [Live Demo]

---

## The problem I was trying to solve

When Microsoft deprecated Azure Synapse Analytics and began pushing customers toward Microsoft Fabric, the core challenge wasn't technical — it was operational. How do you:

1. Know which customers are still actively using the deprecated service?
2. Prioritise outreach when you have thousands of tenants?
3. Communicate in a way that actually gets a response?
4. Track whether migration is happening before the EOL deadline hits?

I came across the Azure Data PM role at Microsoft, which describes exactly this problem. The JD mentions telemetry-driven outreach, deprecation playbook development, and communication management at scale. I decided to build something that shows how I'd actually approach it.

---

## What I built

A migration tracking tool with five sections:

**Overview** — KPI cards showing migration completion, at-risk tenant count, days to EOL. A timeline showing which deprecation phase we're in. A region-level breakdown showing where migration is lagging.

**Telemetry** — A table of tenants still making daily Synapse API calls, sorted by volume. A trend chart comparing Synapse usage (declining) vs Fabric adoption (growing). This is the core signal — active API usage after a deprecation announcement means the customer hasn't started migrating.

**Customer table** — Full tenant list with migration %, contact status, and recommended next action. Filterable and searchable. Built to replicate what you'd actually want in front of you during a daily outreach sprint.

**SQL + KQL queries** — The four queries that would power the real data behind this tool. Two SQL queries (T-SQL) for customer segmentation. Two Kusto queries (KQL) for Azure Monitor telemetry. These are real and designed to work in an actual Azure environment.

**Outreach queue** — Draft customer emails that reference actual usage data from telemetry — not generic notices, but messages that say "your workspace (t_8821a) is running 2,841 pipeline operations per day, you have 16 days left." That specificity is what drives response rates.

**Playbook** — A six-step process for handling migrations at scale: telemetry pull → segmentation → first contact → follow-up cadence → validation → post-migration support. Written to be repeatable without PM involvement at every step.

---

## The SQL and KQL queries

I wrote these from scratch while learning the Azure data schema. The SQL queries run against a relational model where `api_usage`, `tenants`, and `migration_status` are joined. The KQL queries target `AzureDiagnostics` in Azure Monitor — which is the actual log table for Synapse workspace activity.

The most useful query is the urgency segmentation one. It takes every unmigrated tenant, calculates days to EOL, and assigns a tier: CRITICAL (under 14 days), HIGH (under 30), MEDIUM (under 60), LOW (everything else). That output becomes the outreach priority list.

---

## What I learned

**On the PM problem:**
The hard part of a deprecation program isn't the technical migration — it's the customers who don't respond. The tool is designed around that: everything flows from "who is still active and hasn't replied?" rather than "who has migrated?"

**On KQL:**
KQL (Kusto Query Language) is Azure's query language for logs and telemetry. It's different from SQL — more like a pipeline of transformations. I'd used it for basic monitoring at Accenture but hadn't written complex aggregations before. The `summarize...by` pattern and `bin()` for time bucketing are the key patterns for this kind of telemetry work.

**On Synapse vs Fabric:**
Synapse Analytics was Microsoft's answer to AWS Redshift and Google BigQuery — a unified analytics platform. Fabric is the next generation, built on a lake-centric architecture with tighter Power BI integration. The migration isn't just a version upgrade — customers are moving to a different data model, which is why it requires PM-driven outreach rather than an automated upgrade.

---

## Connection to the role

The JD describes four core responsibilities:

| JD requirement | How this project addresses it |
|---|---|
| Telemetry-driven outreach | KQL queries identify active users; outreach queue is sorted by API call volume |
| Process & communication management | Playbook defines a repeatable 6-step process; outreach drafts are templated from telemetry |
| Customer support on deprecation | Outreach tab shows personalised, data-driven communications |
| Documentation & playbook enhancement | Playbook tab is the full written process with ownership, tools, and SLAs |

---

## What I'd do next

If this were a real internal tool, I'd add:

- Integration with the Azure Retirement Team's CRM to auto-sync contact status
- A SLA compliance tracker (response within 48h, migration call within 7 days)
- Automated playbook triggers: if a CRITICAL tenant hasn't responded in 3 days, auto-escalate
- Power BI embed for the leadership KPI view (the JD specifically mentions Power BI)

---

## Reflection

I went into this project thinking it was mostly about building something that looks good for an interview. It ended up being genuinely useful for understanding what the job actually involves. The deprecation program problem is an operations + communication + data problem all at once, and the PM role is essentially the connective tissue between all three. That's the part I find most interesting.
