---
title: Why Your Data Pipeline Is a Liability — And How to Fix It
slug: why-your-data-pipeline-is-a-liability-and-how-to-fix-it
date: '2026-05-11T06:33:54.050Z'
updatedAt: '2026-05-11T06:33:54.050Z'
updatedBy: Murali Doss
updatedByPhoto: >-
  https://lh3.googleusercontent.com/a/ACg8ocL6fpP_XJjqotikeym-CLUAS6LxY4eEzK8x1N8OCJ1jk_63G1UHYw=s96-c
description: By Murali Doss | DevSecOps & DataOps Consultant, Rtym AI Works
tags:
  - data
  - pipeline
  - business
  - gold
  - quality
  - three
  - layer
  - rollback
cover: >-
  https://res.cloudinary.com/djexzbqvx/image/upload/v1778481158/%22website-blog-images%22/Firefly_gpt-image_Three_horizontal_glowing_bands_in_bronze_silver_and_gold__colors_against_a_dark_navy_928144_aozagm.png
canonical: >-
  https://www.rtymaiworks.com/blog/why-your-data-pipeline-is-a-liability-and-how-to-fix-it
seoTitle: Why Your Data Pipeline Is a Liability — And How to Fix It
seoDescription: By Murali Doss | DevSecOps & DataOps Consultant, Rtym AI Works
seoKeywords:
  - data
  - pipeline
  - business
  - gold
  - quality
  - three
  - layer
  - rollback
  - version
  - gate
---

# Why Your Data Pipeline Is a Liability — And How to Fix It

**By Murali Doss | DevSecOps & DataOps Consultant, Rtym AI Works**

---

![data pipeline medallion pvaehd](https://res.cloudinary.com/djexzbqvx/image/upload/v1778480278/%22website-blog-images%22/data-pipeline-medallion_pvaehd.png)

---

There is a pattern I see repeatedly when I walk into a new data engagement.

A team of capable engineers. A Databricks or Synapse environment that someone spent months setting up. Dashboards that the business relies on every morning. And underneath all of it — a deployment process that consists of one engineer SSHing into a notebook, making changes directly in production, and hoping nothing breaks.

No version control. No quality gate. No automated deployment. No rollback plan.

When I ask how they recover if a bad transformation corrupts the data, the answer is usually some variation of: "We restore from a backup if we have one, or we manually rebuild it. Last time it took three days."

This is not an edge case. This is the default state of most data pipelines I encounter, including those at large, well-funded organisations.

This post is about why that default state is a serious business risk — and what a properly engineered alternative looks like.

---

## The Problem Nobody Talks About in the Sales Deck

When organisations evaluate Databricks or Azure Synapse or Snowflake, the conversation is almost always about capability. How many terabytes can it process? How fast are the queries? How good are the dashboards?

The conversation that rarely happens is about operational maturity. Who owns the notebooks? How are changes reviewed? What happens when the SAP export arrives with a different column structure than last night? What happens when a data engineer introduces a bug in a transformation at 11 PM and 120 business users open their dashboards at 9 AM?

These are not edge cases. They are the normal operational challenges of any organisation running a live data platform. And most organisations have no systematic answer to any of them.

The result is predictable. Data incidents that take days to diagnose. Dashboard numbers that the business has quietly stopped trusting. Engineering teams spending their weekends on manual recovery. And a growing gap between what the data platform was promised to deliver and what it actually delivers.

---

## The Core Idea: Treat Data Transformations Like Application Code

The engineering discipline that solved these exact problems for application deployment is DevOps. Every change is version controlled. Every deployment goes through automated gates. Nothing reaches production without review. Recovery from a bad deployment takes minutes, not days.

DataOps applies the same discipline to data pipelines. The insight is straightforward:

A notebook that transforms your SAP data is code. It should be version controlled, reviewed, tested, and deployed through an automated pipeline — exactly like application code.

When you accept that premise, everything else follows. Quality gates before deployment. Automated rollback when something goes wrong. Full audit trail of every change. Monitoring that catches problems before business users do.

---

## What We Built — A Reference Implementation

To demonstrate what this looks like in practice, we built a complete DataOps pipeline on Databricks, modelled around the operational data of a heavy equipment dealer managing equipment sales, parts procurement, and field service operations across multiple branches and regions in India.

The architecture follows the Medallion pattern — three layers of data with increasing quality and business readiness — wired together with a GitHub Actions CI/CD pipeline, automated quality gates, and a Delta Lake rollback mechanism.

Here is the complete flow:

---

### The Three Quality Gates

Before any data reaches the Bronze layer, three mandatory gates run in sequence. If any gate fails, the pipeline stops completely. Yesterday's clean data remains untouched in production.

**Gate 1 — Schema Validation**

The first thing that can go wrong with a data pipeline is structural. The SAP export that arrives tonight might have a column renamed, a column dropped, or three new columns added by someone in the ERP team who didn't think to notify the data engineering team.

The schema gate compares every incoming file against a defined contract — the expected column names and data types. If the structure has changed, the pipeline stops immediately with a specific message explaining exactly what changed.

```
SCHEMA VALIDATION GATE
✅ equipment_sales     — 13 columns present, types valid
✅ parts_orders        — 11 columns present, types valid
✅ service_workorders  — 14 columns present, types valid
✅ customers           — 9 columns present, types valid

ALL SCHEMA CHECKS PASSED
```

**Gate 2 — Data Quality Validation**

The second gate checks content. The structure might be correct but the data itself might be wrong — null values in required fields, negative amounts in a revenue column, unknown values in a field that should only contain a defined set of options.

We run 62 checks across the four datasets. Every check has a specific business reason:

- Equipment sales must not have null customer IDs — you cannot process a sale with no customer
- Sale amounts must be positive — a zero or negative revenue record will corrupt your regional revenue totals
- Region must be one of the five known regions — an unknown region means a mapping error in SAP that will create a phantom entry in your dashboard

```
VALIDATION SUMMARY
equipment_sales       ✅ PASSED   18/18 checks
parts_orders          ✅ PASSED   14/14 checks
service_workorders    ✅ PASSED   17/17 checks
customers             ✅ PASSED   13/13 checks

Total checks : 62
Passed       : 62
Failed       : 0

ALL QUALITY CHECKS PASSED — Pipeline may proceed
```

If a check fails, the output is specific:

```
❌ Column 'sale_amount_inr' values between 1 and 100,000,000
   → 31 values out of range

❌ Column 'region' only contains valid values
   → Invalid values: {'MARS'}

QUALITY CHECKS FAILED — Pipeline BLOCKED
```

The data team knows exactly what failed, exactly how many records are affected, and exactly what to investigate in the source system. No guessing. No three-day investigation.

**Gate 3 — Code Review via Pull Request**

The third gate is human. Every notebook change goes through a GitHub pull request before it can be merged to main. A second engineer reviews it. Only after approval does the pipeline deploy it to Databricks.

This is the gate most data teams skip entirely. It is also the gate that catches the most expensive mistakes — logic errors that pass automated checks but produce subtly wrong numbers that nobody notices for weeks.

---

### The Medallion Architecture

Once data passes all three gates, it flows through three transformation layers.

**Bronze — The Audit Layer**

The Bronze layer ingests raw data exactly as it arrived from the source system. Nothing is modified. Nothing is cleaned. Every record gets three audit columns appended:

- `_ingested_at` — the exact timestamp when this record entered the platform
- `_source_file` — which file this record came from
- `_record_hash` — a hash fingerprint of the entire record

The record hash is what enables change detection. If the same record arrives tomorrow with a different value in any field, the hash changes. You know exactly which records changed, when they changed, and what they changed from.

Bronze is never modified after ingestion. It is your audit trail and your recovery point. If anything goes wrong downstream in Silver or Gold, you always have the original raw data to reprocess from.

**Silver — The Trusted Layer**

The Silver layer applies business rules. Records that fail validation are not deleted — they are moved to a quarantine table with a reason. This distinction matters more than it might seem.

When you silently delete bad records, the problem is invisible. Nobody knows the source system has a data quality issue. It continues producing bad data indefinitely. When you quarantine bad records with a reason, the problem is visible. The data team can trace every rejected record back to its cause and escalate to the source system team.

Silver also adds calculated fields that the business needs. For the heavy equipment dealer use case, we added:

- `sla_breach` — a boolean flag on every service workorder indicating whether the SLA was met
- `overdue_hours` — exactly how many hours past the SLA commitment the job resolved
- `customer_tier` — Enterprise, Mid-Market, or SME based on fleet size
- `sale_quarter` — derived from the sale date for easy time-series analysis

These fields are computed once in Silver and stored. Every query against Silver benefits from them without recalculating.

**Gold — The Business Layer**

The Gold layer contains pre-computed aggregations that directly answer the business questions leadership asks every day. Five tables, five conversations:

| Gold Table | Business Question It Answers |
|---|---|
| `gold_sales_performance` | Which region and segment is generating the most revenue this quarter? |
| `gold_sla_compliance` | Which branches are meeting their SLA commitments? Which fault types are causing the most breaches? |
| `gold_parts_analysis` | Which parts account for the most spend? Where are emergency orders concentrated? |
| `gold_customer_portfolio` | Which customers are at risk of churning? Who are our strategic accounts? |
| `gold_executive_kpis` | What are the headline numbers — revenue, SLA compliance, at-risk customers — right now? |

Gold is rebuilt completely on every pipeline run. It is always consistent with Silver. Nobody inserts or updates Gold directly. Ever.

---

### The Rollback Mechanism

Delta Lake stores a version history for every table. Every write creates a new version. Version 0 is the first write. Version 4 is the fifth write. You can query any version at any time.

We built a rollback script that uses Delta Lake's `RESTORE TABLE` command to roll back any layer — Bronze, Silver, Gold, or all three — to any previous version:

```bash
# Roll back Gold layer by 1 version — undo the last pipeline run
python scripts/rollback.py --layer gold --steps 1 --token YOUR_TOKEN

# List all versions of Gold tables
python scripts/rollback.py --list gold --token YOUR_TOKEN
```

The list output shows the complete version history:

```
Table: gold_sales_performance
  v4  2026-05-08  CREATE OR REPLACE TABLE AS SELECT
  v3  2026-05-08  CREATE OR REPLACE TABLE AS SELECT
  v2  2026-04-27  CREATE OR REPLACE TABLE AS SELECT
  v1  2026-04-27  CREATE OR REPLACE TABLE AS SELECT
  v0  2026-04-25  CREATE OR REPLACE TABLE AS SELECT
```

A full Gold layer rollback completes in under 2 minutes. The dashboard reflects the previous clean state on the next refresh. No data loss. No weekend calls. No three-day investigation.

---

### The Executive Dashboard

The Gold layer feeds a Databricks dashboard with five panels that auto-refresh every morning at 6 AM. By the time leadership opens their screens at 9 AM, the dashboard shows last night's processed, validated, quality-checked data.

Revenue by Region Dashboard![Panel1 RevenueByRegionAndSegment xdcj9x](https://res.cloudinary.com/djexzbqvx/image/upload/v1778480108/%22website-blog-images%22/Panel1-RevenueByRegionAndSegment_xdcj9x.png)

SLA Compliance table ![Panel2 SLAComplianceByBranch teorgo](https://res.cloudinary.com/djexzbqvx/image/upload/v1778480109/%22website-blog-images%22/Panel2-SLAComplianceByBranch_teorgo.png)

Executive KPI summary ![Panel5 ExecutiveKPISummary qjpdh1](https://res.cloudinary.com/djexzbqvx/image/upload/v1778480108/%22website-blog-images%22/Panel5-ExecutiveKPISummary_qjpdh1.png)

The dashboard is not the most technically interesting part of this architecture. But it is the most important part from a business perspective. It is the answer to the question every CTO and CDO eventually asks: "So what do our business users actually see?"

The answer is: clean, validated, automatically updated numbers with a clear data lineage from source to screen.

---

## The Numbers That Made the Decision Clear

For the heavy equipment dealer use case, the business case for this architecture was direct.

The existing Azure Synapse environment cost approximately ₹18–20 lakhs per year. The Gold layer on Delta Lake serves the same purpose. The migration eliminates that cost.

The manual weekly report that required a data engineer spending approximately 8 hours every Friday compiling numbers from multiple sources into a spreadsheet — that process disappears. The Gold tables produce those numbers automatically every night.

The last major data incident — where a bad SAP export corrupted the production dashboard and required three days of manual recovery — would have been a 2-minute rollback with this architecture.

These are not hypothetical benefits. They are the direct consequence of applying engineering discipline to a data pipeline.

---

## What This Looks Like as a Consulting Engagement

DataOps modernisation engagements follow a consistent pattern regardless of industry or data platform:

**Phase 1 — Maturity Assessment (1–2 weeks)**
Audit the current state. Where is data modified directly in production? Where are quality checks missing? Where is the rollback plan? Deliver a gap analysis and architecture recommendation.

**Phase 2 — Foundation (3–4 weeks)**
Implement version control for notebooks. Set up the CI/CD pipeline. Define and automate the quality gates. Establish Bronze ingestion with audit columns.

**Phase 3 — Medallion Implementation (4–6 weeks)**
Build Silver transformation with business rules. Build Gold aggregations aligned to specific business questions. Connect to existing BI tools.

**Phase 4 — Handover and Enablement (1–2 weeks)**
Train the data engineering team on the new patterns. Document the runbooks. Hand over the rollback procedures. Establish monitoring and alerting.

The full implementation — from assessment to live production pipeline — typically runs 10–14 weeks for a mid-size data environment.

---

## Industries Where This Architecture Applies Directly

The patterns in this implementation are not specific to heavy equipment. They apply to any organisation where:

- Data flows from an ERP or operational system into an analytics platform
- Business users depend on dashboards for operational decisions
- Data engineers are making changes under time pressure without adequate safeguards
- The cost of a data incident — whether in recovery time or bad decisions made on wrong numbers — exceeds the cost of building the pipeline correctly

The industries where we see this most often: manufacturing, mining, construction, telecom, financial services, logistics, and healthcare.

---

## The Reference Implementation

The complete code for this pipeline is available on GitHub. It includes the data generator, all three notebook layers, the schema and quality validators, the rollback script, the GitHub Actions workflow, and the dashboard SQL queries.

**GitHub:** [github.com/MurLeeDas/dataops-medallion-pipeline](https://github.com/MurLeeDas/dataops-medallion-pipeline)

If your organisation is running a data platform and any of the patterns described here feel familiar — notebooks modified in production, dashboards that break without warning, recovery processes that take days — we would be glad to do a no-obligation assessment.

---

## About Rtym AI Works

Rtym AI Works is a technology consulting firm specialising in DevSecOps, DataOps, and AI-enabled automation for telecom, fintech, manufacturing, and industrial organisations. Our engagements span CI/CD modernisation, data platform architecture, and cloud infrastructure automation.

We have delivered engagements for clients in Bangladesh and in the Netherlands.

**Contact us:** https://www.rtymaiworks.com/contact

**LinkedIn:** [linkedin.com/in/dossops](https://www.linkedin.com/in/dossops/)

**GitHub:** [github.com/MurLeeDas](https://github.com/MurLeeDas)

---

*Published by Murali Doss, Rtym AI Works. May 2026.*