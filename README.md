# Northstar Residential Group — Marketing Analytics Platform

A full-scale, end-to-end marketing analytics system I designed to simulate how a real multifamily real estate organization measures performance. Connecting marketing spend and attribution to leasing outcomes and occupancy through a continuously updated, deterministic data pipeline.

This is not a dashboard demo, it’s a fully modeled analytics ecosystem with aligned funnel, attribution, and operational data designed to mirror real-world decision environments, supported by automated monitoring that tracks ETL pipeline health, logs failures, and triggers alerts.

---

## Executive Dashboard

![Executive Summary](./Executive%20Summary.png)

*A portfolio-level view of marketing performance, leasing efficiency, and operational health across 120 properties, 12 markets, and 4 regions, built using real-world marketing analytics frameworks and domain and real world experience to reflect how performance is measured in practice.

---

## The Problem This Solves

Marketing performance data is inherently fragmented across systems.

- Spend lives in ad platforms (Google Ads, Meta, Programmatic Display Ads, Paid Search, ILS partners, etc..)
- Leads and interactions live in CRM systems
- Leasing outcomes live in property management systems
- Operational metrics (occupancy, absorption, vacancy) live elsewhere entirely

These systems are rarely aligned — and the data within them is often inconsistent.

- Duplicate lead records and inconsistent identifiers across systems  
- Incomplete or delayed CRM updates  
- Mismatched attribution logic between platforms  
- Data quality issues that distort funnel and conversion metrics  

As a result, organizations struggle to answer fundamental questions:

- Which marketing channels are actually driving leases?
- Where is spend efficient versus wasted?
- How does marketing performance vary by market and property?
- How do marketing efforts translate into operational outcomes like occupancy?

Legacy reporting approaches — often static, tabular, and disconnected — make this even harder by requiring manual reconciliation, masking data quality issues, and limiting visibility across the full funnel.

This project was built to solve that problem by creating a unified, end-to-end analytics system where:

**Spend → Funnel → Prospect Journey → Leasing → Operations**

are fully aligned, standardized, and measurable within a single model.

---

## System Flow

The platform is built as a connected analytics system rather than a standalone report.

```text
Synthetic Source Generation
        ↓
Bronze Ingest
        ↓
Silver Standardization + Validation
        ↓
Identity Resolution + Attribution Logic
        ↓
Gold Fact Tables
        ↓
Power BI Semantic Model + Executive Reporting
        ↓
Pipeline Monitoring, Logging, and Alerts


---

## Scope of Work
**In scope**
- Marketing-focused discovery and requirements gathering  
- KPI framework definition and documentation  
- Data model design using simulated enterprise data  
- Power BI dashboard wireframing and development  
- Insight development and strategic recommendations  
- Full project documentation (scenario, requirements, data dictionary, wireframes)

**Out of scope**
- Enterprise BI dashboards outside of Marketing  
- Financial, accounting, or operational reporting  
- Production data pipelines or live system integrations  

---

## Project Artifacts
This repository contains documentation and artifacts covering the full lifecycle of the engagement, including:
- Scenario and discovery documentation
- Business requirements and scope definition
- Marketing KPI frameworks and definitions
- Data model design and data dictionary
- Dashboard wireframes and UX decisions
- Final insights and recommendations

Artifacts are organized chronologically by project phase to reflect a real analytics delivery process.

---

## Data & Privacy Notice
Northstar Residential Group is a fictional company created for portfolio purposes. All data used in this project is simulated. Table structures, KPIs, and reporting logic are modeled after real enterprise marketing analytics environments, but no proprietary or client data is included.

Power BI files and full datasets are intentionally excluded from this repository.

---

## Repository Use
This repository is intended for review as a **case study**, not as a runnable or production codebase. Each document can be reviewed independently to understand the project approach, decision-making process, and outcomes.

---

## Status
This case study represents a completed end-to-end analytics engagement and is maintained for portfolio and interview discussion purposes.
