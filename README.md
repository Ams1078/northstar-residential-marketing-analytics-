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

This is the end-to-end architecture that powers the platform:

![System Flow](./images/NorthStar_Pipeline_Schema_Flow.svg)

This platform is designed as a full analytics pipeline, not just a reporting layer.

Data flows from synthetic source generation through bronze ingestion, silver standardization, identity resolution, and attribution modeling before being published to gold fact tables that power the Power BI semantic model.

Each stage includes validation, data quality checks, and audit logging to ensure that marketing activity can be consistently traced from spend through funnel performance, prospect journeys, leasing outcomes, and operational results.

The pipeline is supported by monitoring and logging infrastructure that tracks run status, captures failures, enforces watermark-based processing, and triggers alerts when issues occur.

---

## Technical Deep Dive

This project includes full pipeline documentation covering data ingestion, validation, identity resolution, attribution modeling, and audit layer design.

[View CRM Pipeline Process Documentation](./images/CRM_Pipeline_Process_Documentation.docx)

---

## Engagement Overview

Northstar Residential Group is a fictional, enterprise-scale multifamily real estate company used for this portfolio case study. The organization operates a large, geographically distributed property portfolio and relies heavily on digital marketing to drive leasing demand.

This engagement simulates the design and delivery of a dedicated marketing analytics platform built to replace legacy SSRS-based reporting, which provided accurate data but lacked visibility, flexibility, and the ability to connect performance across the full marketing and leasing lifecycle.

At the time of the engagement, marketing stakeholders faced several limitations:

- Performance data was fragmented across platforms (ad networks, CRM, property systems)  
- Reporting was static and difficult to interpret at an executive level  
- Attribution between marketing activity and leasing outcomes was unclear  
- Data quality issues (duplicate records, inconsistent identifiers, incomplete CRM data) created additional uncertainty  

The objective of this project was to design a unified analytics system that could:

- Align marketing spend, funnel performance, and leasing outcomes  
- Provide consistent attribution across channels and touchpoints  
- Surface performance at the portfolio, market, and property level  
- Improve visibility into both marketing efficiency and operational impact  

---

## Role & Ownership

I led this engagement end-to-end, acting as both the marketing analytics lead and system designer.

This included:

- Defining the marketing KPI framework and measurement strategy  
- Translating legacy reporting into a modern analytical model  
- Designing the data model and attribution logic  
- Building the Power BI reporting layer and executive dashboard  
- Developing supporting pipeline logic, validation rules, and audit tracking  
- Generating insights and strategic recommendations  

This role required bridging domain knowledge in digital marketing, CRM data, and performance measurement with technical implementation across data modeling, pipeline design, and reporting.

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
