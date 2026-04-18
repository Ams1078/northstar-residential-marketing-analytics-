---
title: "NorthStar Marketing Analytics Dashboard"
subtitle: "Release Notes — v1.0"
author: "NorthStar Residential Group"
date: "April 2026"
---

# About These Notes

This document describes every visual that ships in **v1.0** of the NorthStar Marketing Analytics Dashboard. It is written for end users who need to read the dashboard correctly — what each visual is for, how it's calculated, what it means, and where it can mislead.

Each visual entry follows the same structure:

- **Purpose** — what question it answers
- **Inputs** — fact tables and dimensions involved
- **DAX / Measure** — the measure(s) behind it (named, with formula or pattern)
- **How to read it** — what to look for first, second, third
- **Edge cases** — when the number can mislead and how to spot it

Numbers shown in screenshots are illustrative, taken from the v1.0 demo data covering April 2025 – April 2026 across 120 properties, 11 vendors, and 12 markets.

---

# What's in v1.0

| Page | Type | Audience | Refresh |
|---|---|---|---|
| Executive Summary | Primary | Executives, Directors | Daily |
| Operations Monitor | Primary | Ops Leadership, Regional Mgrs | Daily |
| Marketing Funnel | Primary | Marketing, Vendor Managers | Daily |
| Geography Page | Primary | Marketing, Regional Leadership | Daily |
| Operations Detail | Drill-through | Property/Asset Managers | Daily |
| Funnel Detail | Drill-through | Vendor Managers, Marketing Ops | Daily |
| Geography Detail | Drill-through | Regional Marketing, Property Mgmt | Daily |

**Data foundation:** Star schema with 4 dimensions (`dim_property`, `dim_vendor`, `dim_market`/`dim_region`, `dim_date` plus role-playing `dim_lease_date`) feeding 4 fact tables: `fact_property_ops_daily` (99K rows), `fact_leasing_daily` (99K rows), `fact_marketing_spend_daily` (1.09M rows), and `fact_prospect_journey` (multi-touch attribution).

**Three composite indices** drive every score in the dashboard. Read these first — every visual that shows a tier, badge, or ranked list is built on one of them.

## VPI — Vendor Performance Index

Evaluates each marketing vendor across the full leasing funnel. Conversion-heavy weighting:

| Stage | Weight | Components |
|---|---|---|
| Awareness | 15% | Impressions, CTR, CPC, Click-to-Visit (25% each) |
| Consideration | 20% | Visits, CPV, Visit-to-Lead (33/34/33) |
| Evaluation | 25% | Leads, CPL, Lead-to-Lease (33/33/34) |
| Conversion | 40% | New Leases, CP-Lease, Imp-to-Lease, Sell Rate (25% each) |

Every component is a **portfolio percentile** (0–100) — the vendor is ranked against all other vendors on that metric. Tier thresholds: **Elite ≥75, Strong 60–74, Moderate 48–59, Underperforming 35–47, Critical <35.**

## GPI — Geographic Performance Index

Two-pillar score for property/market/region marketing health:

- **Pillar 1 — Absolute (60%)**: portfolio percentile across CPL, Cost-per-Lease, Sell Rate, Vacancy Fill %, Vacant Ready %, SLA Compliance %
- **Pillar 2 — Trend (40%)**: period-over-period scaled deltas, capped at ±15%, on New Leases, Cost-per-Lease, Cost-per-Lead, Net Absorption, and Lease Velocity

Tiers: **Excellent · High Performing · Below Average · At Risk · Critical.** Pillar Balance flag: Absolute-Led / Momentum-Led / Balanced (within 5 pts).

## Ops Index Score

Property-level operational health blending occupancy, leasing velocity, absorption, vacancy fill, and pipeline coverage (scheduled move-ins ÷ lease expirations next 60 days). Tiers: **Optimized · Optimizing · At Risk · Critical.**

## The 30-Day Minimum Window Rule

**This is the most important rule to internalize.** Every base measure that feeds a percentile or score enforces a 30-day minimum scoring window. If the filter context resolves to fewer than 30 days, the window automatically expands backward 30 days from the latest date in filter:

```
_Base_<Metric> =
VAR DaysInFilter = [_DaysInFilter]
VAR MaxDate      = [_MaxDateInFilter]
RETURN
IF(
    DaysInFilter < 30,
    CALCULATE([Metric], DATESINPERIOD(dim_date[Date], MaxDate, -30, DAY)),
    [Metric]
)
```

**Why:** percentile rankings on thin data (a single week, a single day) are statistically unreliable and produce wild rank swings. The 30-day floor keeps scores stable.

**Consequence to know:** if you select "This Week" in a date slicer, the percentile-based scores you see were calculated against the **last 30 days**, not the week — but the volume KPIs (Spend, Leases, Impressions) reflect the week as selected. This is intentional but can surprise users who expect everything to follow the slicer.

\newpage

# Page 1 — Executive Summary

![](screen-1.jpg)

**Purpose:** Top-of-house view for executives. Answers "How is the portfolio performing this period, and where is the biggest signal?"

**Default period:** Last 12 Months. Period toggle along the top: This Month · Last Month · This Quarter · Last Quarter · This Year · Last Year · Last 12 Months.

## 1.1 Portfolio Health Composite

**Purpose:** Single 0–100 score for the entire portfolio's marketing + operations health, with a star rating and prior-period delta arrow.

**Inputs:** Three pillar measures derived from `fact_property_ops_daily` and `fact_leasing_daily`:

- `Portfolio_Health_Pillar1_Occupancy` — occupancy + vacancy + absorption components
- `Portfolio_Health_Pillar2_Leasing` — leasing efficiency (cost + velocity)
- `Portfolio_Health_Pillar3_Demand` — demand conversion (top-of-funnel → lease)

**DAX:** `Portfolio_Pillar_Value` switches the displayed pillar by the slicer; the composite is the average of the three pillar scores. Bar color is driven by `Portfolio_Pillar_Bar_Color` which thresholds at 85 / 70 / 55.

**How to read it:**

1. Headline number (80.7) is the composite. Star rating is a visual shortcut for the same number.
2. Three rows below break the composite into its pillars. **The lowest pillar is the bottleneck** — in this screenshot, Demand Conversion at 63 is dragging Leasing Efficiency (91) and Occupancy Health (85) down.
3. Green arrow + percentage is the change vs. the prior equal-length period.

**Edge case:** Star rating thresholds (75 / 60 / 48 / 35) and tier thresholds in the rest of the dashboard are slightly different — don't expect a 4-star rating to align perfectly with "Strong" tier elsewhere.

## 1.2 Region Portfolio Health

**Purpose:** Side-by-side region comparison of the same composite.

**Inputs:** `Portfolio_Pillar_Value` calculated per `dim_region[RegionName]`.

**How to read it:** Bars are sorted by score descending. The numerical badge at the right edge is the integer score. Color gradient is gold-on-dark — darker bars are weaker scores.

**Edge case:** Only 4 regions exist (CE, EA, SO, WE). If a region's data is incomplete (e.g., a market has no spend), the score will still calculate but on a smaller component count. Hover for the tooltip to confirm.

## 1.3 KPI Card Cluster (top-right)

**Purpose:** Six volume + efficiency KPIs with prior-period arrows.

| Card | Measure | Direction Indicator |
|---|---|---|
| Marketing Spend | `Marketing Spend` (sum of `fact_marketing_spend_daily.Spend`) | ▼ = lower spend (could be good or bad — read with leases) |
| New Leases | `New Leases` (from `fact_leasing_daily.NewLeases`) | ▲ = more leases |
| Cost per Lease | `Cost per Lease (Actual)` = Spend ÷ New Leases | ▼ = cheaper per lease |
| Lead-to-Lease % | `Lead-to-Lease Conversion %` | ▲ = better conversion |
| Occupancy Rate % | `Occupancy Rate %` (latest snapshot, not period avg) | ▲ = better |
| Sell Rate | `Sell Rate` = New Leases ÷ Available Units | ▲ = better |

**How to read it:** **Always read Spend and New Leases together** — a falling spend with rising leases is great, but a falling spend with falling leases is just a slowdown.

**Edge case:** Occupancy Rate is a *snapshot* metric (latest available date in filter), not a period average. The prior-period arrow compares snapshot-to-snapshot.

## 1.4 Vendor Spend vs Outcome (Quadrant)

**Purpose:** Identify which vendors/channels are scaling efficiently vs. overspending vs. under-invested.

**Inputs:** Scatter of `Marketing Spend` (Y) vs `New Leases (Funnel)` (X), sliced by Channel or Vendor (dropdown top-right).

**How to read it (quadrants):**

- **Scale & Invest** (top-right): high spend, high leases — keep funding
- **Efficient** (bottom-right): low spend, high leases — increase budget here
- **Overspending** (top-left): high spend, low leases — cut or fix
- **Under-Invested** (bottom-left): low spend, low leases — investigate before scaling

**Quadrant cutoffs** are the median spend and median leases across the items in scope (not fixed thresholds), so the chart is **always relative to the current selection**.

**Edge case:** With only 2–3 channels visible, "median" is defined by very few points and one mover can flip a quadrant. Switch to Vendor view for a denser plot before drawing conclusions.

## 1.5 Market Spend vs Outcome (Quadrant)

**Purpose:** Same logic as 1.4, but at the geographic level (Region or Market dropdown).

**How to read it:** Identical four-quadrant logic. EA in the top-right means it has the highest spend AND highest leases — that's the workhorse region. CE in the bottom-left means low spend AND low leases — investigate whether that's deliberate (smaller portfolio) or a missed opportunity.

## 1.6 Region Performance Table

**Purpose:** Tabular view of the same six KPIs broken out by region.

**How to read it:** Conditional formatting on CPL, L2L, Occupancy, Sell Rate columns — green = good, red = needs attention. Thresholds are portfolio percentile based (`CF_Region_*` measures).

## 1.7 Cost Per Lease & Lead to Lease Conversion (Channel Bars)

**Purpose:** Two horizontal bar charts ranking channels.

- **Cost Per Lease** — sorted ascending (cheapest first). ILS at $193 is the cheapest channel; Email at $800 is the most expensive on a per-lease basis.
- **Lead to Lease Conversion** — sorted descending (best conversion first). ILS at 13.1% converts best.

**Edge case:** Email shows high CPL *and* low conversion — looks bad. But Email also has low total spend, so it's not the biggest dollar problem. Always cross-reference with the Vendor quadrant in 1.4.

## 1.8 Signal Callout Cards (left rail)

Six narrative cards that name the top signal in each category. Driven by their own measures:

- **Most Improved Area** — highest period-over-period delta among GPI components
- **Weakest Pillar** — lowest of the three Portfolio Health pillars
- **Market Signal** — top + bottom GPI markets by name
- **Sharpest Decline Area** — largest negative delta vs prior period
- **Coverage Risk Signal** — coverage ratio (scheduled move-ins ÷ expirations next 60D); ≥1.0 is healthy
- **Velocity Outlook** — period-over-period change in lease velocity

**How to read it:** These are the executive's "what to look at first" cards. They're text strings generated by DAX, so they're always specific (e.g. "Pacific Coast 76.3") — not generic.

\newpage

# Page 2 — Operations Monitor

![](screen-2.jpg)

**Purpose:** Property-health monitor for operations leadership. Answers "Which properties are at risk, why, and how concentrated is the risk?"

## 2.1 Ops Health Tile (header)

**Purpose:** Single 0–100 score for portfolio-wide ops health, color-coded badge.

**DAX:** Average of `Operations Index Score` across all properties in scope. Green checkmark when ≥85; warning icon below.

## 2.2 Five-KPI Header Strip

| Card | Measure | What "good" looks like |
|---|---|---|
| Occupancy % | Latest snapshot of `Occupancy Rate %` | ▲ trending up; ≥90% is strong |
| Lease Velocity | `Lease Velocity` — leases ÷ units in period | ▲ trending up |
| Absorption Days | `Vacant Unit Absorption Days (Est)` | ▼ trending down (faster fill) |
| Vacancy Fill | `Vacancy Fill % (30D)` | ▲ trending up |
| Operations SLA | `% Properties Within SLA` | ▲ trending up |

Each card shows a 3-month average for context and the prior-period arrow.

**Edge case:** Absorption Days is an *estimate* derived from vacant ready units and recent move-in pace — not a direct measurement. Treat it as a directional indicator, not an absolute.

## 2.3 Highest Risk / Slowest / Weakest Callouts (left rail)

Six narrative signal cards, each pointing to the *single* worst case in its category:

- **Highest Risk Area** — region with lowest GPI + count of Critical/At Risk properties
- **SLA Breach Concentration** — region with highest % of properties out of SLA
- **Slowest Absorption** — region with highest avg absorption days
- **Weakest Pipeline Coverage** — region with lowest scheduled move-ins ÷ expirations ratio
- **Move-Out Pressure** — same coverage ratio framed as a count
- **Lease Velocity Gap** — region underperforming its own 3-month average

**How to read it:** These cards are deliberately repetitive across categories so you can spot a region appearing in multiple cards — that's a concentration signal. In the screenshot, "Central" appears in 4 of 6 cards.

## 2.4 Leasing Pressure & Portfolio Exposure (Scatter)

**Purpose:** Vacant Ready % (X) vs Absorption Days (Y) per market — exposes properties simultaneously high on supply and slow on absorption.

**Quadrants:**

- **Healthy** (bottom-left): low vacancy, fast absorption — best
- **Demand** (bottom-right): high vacancy, fast absorption — supply pressure but moving
- **Velocity Critical** (top-left): low vacancy, slow absorption — small problem but slow
- **Critical** (top-right): high vacancy, slow absorption — biggest problem

**How to read it:** Bubble color reflects GPI tier. A red bubble in the top-right is the most urgent intervention.

## 2.5 Operational Risk Distribution Matrix

**Purpose:** Per-region count of properties by risk tier (Critical, At Risk, Optimizing, Optimized).

**DAX:** `Property Count Matrix` — counts properties per region × per status, where status is derived from `Operations Index Score` thresholds (≥85 Optimized, 65–84 Optimizing, etc.).

**How to read it:** Read each row across — a region with most weight in "Optimized" and "Optimizing" is healthy. A region with weight in "Critical" needs intervention. The screenshot shows CE with 11 Critical properties (most concentrated risk).

**Edge case:** Counts are absolute, not percentages. CE having 11 critical and EA having 2 critical doesn't mean CE is 5× worse — it means CE has 5× more critical properties out of its 30 total. Use the Operational Region Heatmap (page 4) for a normalized view.

## 2.6 Region × Status Percentage Heatmap

**Purpose:** Per-region split of properties across Healthy / Demand / Velocity / Critical buckets, as percentages.

**How to read it:** Reads the same way as 2.5 but normalized by region. CE at 67% Critical is concerning; EA at 73% Healthy is healthy. The color gradient (green→red) is the visual shortcut.

## 2.7 Operations Property Drill List

**Purpose:** Sortable property table — Region, Market, Property, SLA badge, Score, Risk Level. Click-through to **Operations Detail** page (drill-through).

**How to read it:** Default sort is by score descending. The SLA column shows a green checkmark when within SLA, red when not. The Risk Level column maps to the same tiers as the matrix above.

\newpage

# Page 3 — Marketing Funnel

![](screen-3.jpg)

**Purpose:** Vendor performance scoreboard with funnel conversion, contribution gaps, and VPI rankings. Answers "Which vendors are paying off, where in the funnel are we losing prospects, and where is spend misallocated?"

## 3.1 Funnel Health Tile (header)

**Purpose:** Single 0–70-something composite for funnel health.

**DAX:** Average funnel-stage score weighted by stage. The yellow warning icon at 70 indicates the score is in the "Performing" tier but not "Top Performer" — a single click on the tile shows the breakdown.

## 3.2 Five-Stage Funnel Header

Five cards: **Impressions · Clicks · Leads (Funnel) · Visits (Funnel) · New Leases (Funnel)**

Each card shows total volume in period, the 3-month average, and the prior-period delta arrow. The "(Funnel)" suffix distinguishes funnel measures (`fact_marketing_funnel_daily`) from operational measures (`fact_leasing_daily`).

**Edge case:** Leads (Funnel) and Leads (Actual) can differ. Funnel leads count *touchpoints* tagged as leads in the marketing funnel; Actual leads count CRM lead records. Reconciliation between the two is part of the ETL pipeline (`pipeline_flags` table); some divergence is normal.

## 3.3 Best/Worst Performance Callouts (left rail)

Six narrative cards:

- **Best Performer Cost Per Lease** — vendor with lowest CPLease
- **Best Performer Lead to Lease** — vendor with highest L2L %
- **Biggest Spend Waste** — vendor with high spend AND low VPI
- **Best Efficiency** — vendor with best L2L given spend
- **Top Movements** — vendor with biggest period-over-period VPI change
- **Lowest Results** — vendor with worst funnel gap

**How to read it:** These are the marketing leader's "where to focus this week" cards.

## 3.4 Contribution vs Results (Donut)

**Purpose:** Vendor's share of impressions (or leads/spend, by toggle).

**How to read it:** Each segment is one vendor; outer ring shows % of total volume. Pair this with 3.6 — a vendor with a big slice here AND a positive Contribution Gap is healthy; big slice here AND negative gap is overspending.

## 3.5 Funnel Performance Visual

**Purpose:** Stage-by-stage waterfall — Impressions → Clicks → Leads → Visits → New Leases — with conversion % at each step and spend bars overlaid.

**How to read it:**

- The **width of each bar** is volume
- The **percentage label** is the conversion rate from the prior stage
- The **right-side gold bars** are the cost-per-step ($1.56 per click, $9.87 per lead, etc.)

The biggest drop-off in the screenshot is Clicks → Leads at 25.8% — that's the stage to fix first.

## 3.6 Contribution Gap Bar Chart

**Purpose:** Disparity between a vendor's share of spend and share of conversion.

**DAX:** `(Vendor's % of Conversions) − (Vendor's % of Spend)`. Positive = converting more than its spend share (efficient); negative = overspending relative to results.

**How to read it:** Vendors are sorted descending by gap.

- **Green positive bars** at top — these vendors are over-delivering. Apartments.com at +15.4% means it converts 15.4 percentage points more than its spend share would predict.
- **Red negative bars** at bottom — these vendors are over-paid relative to results. Bing Ads at -4.8% is the largest underperformer.

This is **the most actionable single visual** for budget reallocation conversations.

## 3.7 VPI Vendor Ranking Table

**Purpose:** Definitive ranked list of vendors by VPI score.

| Column | Source |
|---|---|
| Vendor | `dim_vendor[VendorName]` |
| Rank | `VPI Portfolio Rank` |
| Score | `Vendor Performance Index` |
| VPI Consistency | `VPI Consistency` (range across 4 stage scores) |
| Status | `VPI_Status_Icon` (Top Performer / Performing / Warning / At Risk) |

**How to read it:**

1. Score is the headline.
2. **Consistency tells you whether to trust the score.** "Very Consistent" means the vendor is balanced across all 4 funnel stages. "Very Inconsistent" means the vendor is great at 1–2 stages and terrible at others — its top-line VPI may hide a serious weakness.
3. Status icon is a quick visual tier marker.

In the screenshot, Google Ads ranks #4 with 61.2 (Performing) — but is "Very Inconsistent." That's a yellow flag: the score is propped up by one or two stages.

**Edge case:** Rank ties get the same number (competition rank). Two vendors tied at 65.0 will both show #3, and the next vendor will show #5 — not #4.

\newpage

# Page 4 — Geography Page

![](screen-4.jpg)

**Purpose:** GPI-driven view of marketing property health across regions, markets, and properties. Answers "Where geographically is marketing performing, where is it failing, and where should we intervene?"

## 4.1 GPI Tier Icon (header)

**Purpose:** Tier badge for the current selection (portfolio, region, market, or property depending on filter context).

**DAX:** `GPI Tier` based on `GPI Score`. Tiers: Excellent (≥80), High Performing (65–79), Below Average (50–64), At Risk (35–49), Critical (<35).

## 4.2 Five-KPI Header Strip

| Card | Measure |
|---|---|
| Cost per Lease | `Cost per Lease (Actual)` |
| Sell Rate | `Sell Rate` |
| Avg Month Spend | `Marketing Spend` ÷ months in period |
| % Marketing SLA | `% Properties Within SLA` |
| Lead-to-Lease | `Lead-to-Lease Conversion %` |

## 4.3 Geographic Signal Callouts (left rail)

Six cards naming the top signal in each category:

- **Highest Risk Area** — region with lowest GPI + worst SLA
- **Best ROI Area** — region with highest GPI + lowest CPL
- **Intervention Concentration Area** — region with most properties needing intervention
- **Lead to Lease Gap** — region where L2L lags portfolio average most
- **Occupancy Recovery Signal** — region trending up most in occupancy
- **Scale Opportunity** — region with capacity to absorb more spend efficiently

## 4.4 Vendor vs Operations Health Matrix (Scatter)

**Purpose:** Lead-to-Lease % (X) vs Cost-per-Lease (Y), per property — surfaces marketing efficiency vs. yield combinations.

**Quadrants:**

- **High Performing** (low CPL, high L2L) — best
- **Efficient** (low CPL, low L2L) — cheap but not converting
- **Underinvested** (high CPL, high L2L) — converting but expensive
- **Rethink** (high CPL, low L2L) — worst, intervention needed

Bubble size = property scale; color = GPI tier.

## 4.5 Geo Score Map

**Purpose:** Choropleth-style U.S. map with property pins colored by GPI tier (green = Excellent, red = Critical, gray = no data).

**How to read it:** Cluster of green pins = healthy region; cluster of red = problem region. Click a pin or region to drill the rest of the page; click through to **Geography Detail** for a single property deep dive.

## 4.6 Region × Tier Distribution Matrix

**Purpose:** Per-region count of properties by Marketing Tier (Efficient, High Performing, Rethink, Underinvested).

**How to read it:** Same logic as the Ops matrix on page 2 — read each row across; concentrations of "Rethink" or "Underinvested" are the targets.

## 4.7 Geo Metric Scores by Geography (Right Table)

**Purpose:** Sortable table — Market, SLA Risk, Score, Tier — for the geographic level selected (Region or Market dropdown).

**How to read it:** Default sort is alphabetical. Sort by Score descending to find the worst markets first. The SLA Risk column has its own threshold ("Healthy" vs "Operational Risk") independent of GPI tier — a market can be Healthy on SLA but still Below Average on GPI.

\newpage

# Page 5 — Operations Detail (Drill-Through)

![](screen-5.jpg)

**Purpose:** Single-property deep dive. Reached via drill-through from page 2 (Operations Monitor). Answers "Why is this specific property scoring what it scores, and how is it trending?"

**Header:** Property name (Greystar Oxnard), Region | Market | property number, and Ops Index Tier badge (Excellent in the screenshot).

## 5.1 Five-KPI Strip with Region Benchmarks

Each KPI card shows three things:

1. **Current value** for the property in the selected period
2. **vs Region Avg** — property minus region average, in absolute or percentage points
3. **Prior Period** — property's own period-over-period change

| Card | Measure |
|---|---|
| Occupancy Rate % | `Occupancy Rate %` |
| Vacant Ready Units | `Vacant Ready Units (Est)` |
| Absorption Days (Est) | `Vacant Unit Absorption Days (Est)` |
| Net Absorption | Move-Ins − Move-Outs in period |
| Vacancy Fill % (30D) | `Vacancy Fill % (30D)` |

**How to read it:** Two arrows per card — *prior period* (the property's trend) and *vs region* (the benchmark gap). Both green = strong on both; both red = serious problem; mixed = need to read carefully.

**DAX pattern (region avg comparison):**

```
<Metric> - vs Region Avg = 
VAR PropVal   = [<Metric>]
VAR RegionAvg = CALCULATE([<Metric>], 
                          ALLEXCEPT(dim_property, dim_property[RegionKey]))
RETURN PropVal - RegionAvg
```

**Edge case:** Region averages are unweighted property averages, not weighted by units. A 10-unit boutique property and a 400-unit complex contribute equally to the average.

## 5.2 Index Score / Rank / SLA Cards (left rail)

- **Ops Index Score** — property's score with star rating
- **Index Rank** — `#43 of 120` ranking format (`Ops_Index_Property_Rank_Display`)
- **Within SLA** — Yes/No badge based on coverage ratio + score thresholds
- **Strongest Component / Weakest Component** — component name driving up / pulling down the score

**How to read it:** Strongest/Weakest is the most actionable pair on this page. If the property scores 90 but its weakest component is "Occupancy Rate," that's where to focus the next ops review.

## 5.3 Leasing Readiness vs Fill Rate (Combo Chart)

**Purpose:** 12-month combo chart with Vacancy Fill % (30D) line and Vacant Ready % area.

**How to read it:** Falling Vacant Ready paired with rising Vacancy Fill = the property is processing units faster than it's accumulating vacancies. The opposite (rising Vacant Ready, falling Vacancy Fill) is a backlog warning.

## 5.4 Unit Flow & Net Absorption (Bar + Line)

**Purpose:** Move-Ins (yellow bars) and Move-Outs (orange bars) with Net Absorption (line) over 12 months.

**How to read it:** Bars above zero are inflow, below are outflow; the line tracks the net. Sustained negative net absorption is a red flag regardless of headline occupancy.

## 5.5 Trend Table

**Purpose:** Month-by-month detail — Index Score, Occupancy %, Vacant Units, Move-Ins (60D), Lease Expirations, Lease Velocity.

**How to read it:** Default sort is descending by month. Look for trend reversals (e.g., Occupancy was rising then started falling) before isolated bad months.

\newpage

# Page 6 — Funnel Detail (Drill-Through)

![](screen-6.jpg)

**Purpose:** Single-vendor deep dive. Reached via drill-through from page 3 (Marketing Funnel). Answers "Why is this specific vendor scoring what it scores, and where in the funnel is it strong or weak?"

**Header:** Vendor — Channel — rank format (Apartments.com | ILS | #2), with VPI Tier Icon (`[Elite]` in the screenshot).

## 6.1 Five-Stage KPI Strip with Vendor-Avg Benchmarks

Same five funnel stages as page 3, but each card now shows:

1. **Current value** for this vendor
2. **vs Vendor Avg** — vendor minus cross-vendor average
3. **Prior Period** — vendor's own period-over-period change

The Vendor Avg comparison is the analog of the Region Avg comparison on page 5 — same pattern, different dimension.

## 6.2 Index Score / Portfolio Rank Cards (left rail)

- **Index Score** — VPI score with star rating
- **Portfolio Rank** — `#2 of 11` ranking format (`VPI Portfolio Rank`)
- **Strongest Area** — name of the strongest funnel stage component
- **Weakest Area** — name of the weakest funnel stage component
- **Stage Scores Summary** — compact `A:## | C:## | E:## | V:##` ribbon

**The Stage Scores Summary is the single best diagnostic on this page.** The screenshot shows `A:100 | C:92 | E:92 | V:100` — Apartments.com is essentially perfect across all four stages. A vendor showing `A:95 | C:88 | E:45 | V:30` would have a strong front-end but a fatal weakness at lead conversion.

**DAX:**
```
VPI Stage Scores Summary =
"A:" & FORMAT([Awareness_Score], "0") & 
" | C:" & FORMAT([Consideration_Score], "0") & 
" | E:" & FORMAT([Evaluation_Score], "0") & 
" | V:" & FORMAT([Conversion_Score], "0")
```

## 6.3 Investment vs Efficiency Over Time (Combo Chart)

**Purpose:** 12-month combo — Marketing Spend bars + Cost-per-Lease line.

**How to read it:** Rising spend with falling CPL is the ideal pattern (you're scaling efficiently). Rising spend with rising CPL means diminishing returns — the vendor is saturated. Falling spend with rising CPL means you're losing scale efficiency.

## 6.4 Funnel Performance Trend (Combo Chart)

**Purpose:** 12-month combo — Leads (Funnel) and Visits (Funnel) over time.

**How to read it:** The two lines should trend roughly together. Divergence signals a conversion-rate change at the visit-to-lead step — worth investigating.

## 6.5 Trend Table

**Purpose:** Month-by-month — Index Score, Spend, Touches, New Leases, L2L, Cost per Lead, Visits, CPV, CTR %, etc.

**How to read it:** Use this to validate the headline VPI score against the underlying inputs. If the score moves but no individual metric changed dramatically, the cause is usually a percentile shift — the vendor's competitors moved.

\newpage

# Page 7 — Geography Detail (Drill-Through)

![](screen-7.jpg)

**Purpose:** Single-property GPI deep dive. Reached via drill-through from page 4 (Geography Page). Answers "Why is this specific property scoring what it scores on marketing, and how is it trending?"

**Header:** Property name (Broadstone Allentown), Region | Market | property number, with Marketing Tier badge (`[Excellent]`).

## 7.1 Five-KPI Strip with Region Benchmarks

Same pattern as page 5 (vs Region Avg + Prior Period), but with marketing-flavored KPIs:

| Card | Measure |
|---|---|
| Marketing Spend | `Marketing Spend` |
| New Leases (Actual) | `New Leases` |
| Cost per Lease | `Cost per Lease (Actual)` |
| Sell Rate | `Sell Rate` |
| Lead-to-Lease % | `Lead-to-Lease Conversion %` |

## 7.2 Index Score / GPI Rank Cards (left rail)

- **Index Score** — GPI score with star rating
- **GPI Rank** — `#5 of 30` (rank within region)
- **Top Measurement** — name of the strongest GPI component
- **Bottom Measurement** — name of the weakest GPI component
- **GPI Summary** — formatted string e.g. `GPI: 89 | Excellent | Abs: 95.0 / Trend: 80.3 | vs Portfolio: +0 pts`

**The GPI Summary string is the single most informative element on this page.** It shows: headline score, tier, both pillar scores, and the gap to portfolio in one line.

**DAX:**
```
GPI Summary =
"GPI: " & FORMAT([GPI Score], "0") &
" | " & [GPI Tier] &
" | Abs: " & FORMAT([GPI Pillar 1 (Absolute)], "0.0") &
" / Trend: " & FORMAT([GPI Pillar 2 (Trend)], "0.0") &
" | vs Portfolio: " & 
    IF([GPI vs Portfolio] >= 0, "+", "") & 
    FORMAT([GPI vs Portfolio], "0") & " pts"
```

**How to read it:** If Abs and Trend are within 5 points of each other, the property is "Balanced" — score is reliable. If one pillar is much higher than the other (e.g., Abs 90 / Trend 40), the property is being held up by either current performance or recent improvement, not both.

## 7.3 Lead % Lease Volume Trend (Combo)

**Purpose:** 12-month combo — New Leases (Actual) bars, Visits (Actual) line, Leads line.

**How to read it:** Watch for the three lines to move together. Diverging lines indicate a conversion-rate shift somewhere in the funnel.

## 7.4 Vacancy & Sell Trend (Multi-Line)

**Purpose:** 12-month chart — Vacancy Fill (property) vs Vacancy Region Avg (dashed), Sell Rate (property) vs Sell Region Avg (dashed).

**How to read it:** Solid lines should sit above the dashed lines for the property to outperform its region. Where solid drops below dashed, the property is lagging.

## 7.5 Trend Table

**Purpose:** Month-by-month — Index Score, Spend, New Leases, L2L, Cost per Lease, Vacancy Fill, Sell Rate.

\newpage

# Cross-Cutting Behaviors

## Date Slicer Interaction

The date slicer at the top of pages 2, 3, and 4 controls all visuals on that page. **Two important behaviors:**

1. **Score measures (VPI, GPI, Ops Index)** apply the 30-day minimum window rule. Selecting "Last 7 Days" still scores against 30 days.
2. **Volume measures (Spend, Leases, Impressions)** follow the slicer exactly. They reflect what's selected.

This means a 7-day selection will show: real 7-day volumes, but 30-day-windowed scores. This is intentional and prevents misleading rankings on thin data.

## Drill-Through Filter Carry

When you drill from a portfolio page into a detail page (e.g., Geography → Geography Detail), only the Property filter carries forward. Date Range, Region, Market are reset to "All". You can re-apply them on the detail page if needed.

## Period-over-Period Calculations

The "Prior Period" arrow on every KPI card uses an equal-length, immediately-preceding window. If the current period is "This Quarter" (Jan–Mar), the prior period is the prior quarter (Oct–Dec). If the current period is a custom 47-day range, the prior is the immediately-preceding 47 days.

**Edge case:** When the prior window resolves to a date before your data starts (e.g., comparing the first month of available data), the delta will be blank rather than misleadingly large.

## Tie Handling in Ranks

Rank measures use **competition ranking**: tied scores receive the same rank, and the next score skips ranks accordingly. Two vendors tied at #3 = both show #3, next is #5 (not #4).

Some legacy rank measures use DENSE rank (no skipping); these are tagged `_Fixed` in the measure name. Where both exist, the non-Fixed version is the production measure.

## Coverage Ratio (Operations Risk)

Coverage ratio = `ScheduledMoveIns_Next60D ÷ LeaseExpirations_Next60D`.

- **≥1.10** — Healthy, surplus pipeline
- **0.95–1.09** — OK, balanced
- **<0.95** — Risk, expirations exceed scheduled move-ins

This drives the "Coverage Risk" callout on page 1, the "Within SLA" badge on page 5, and the "Pipeline Coverage" callout on page 2.

\newpage

# Known Limitations in v1.0

- **Funnel vs Actual reconciliation:** Some divergence between Funnel-stage measures (from `fact_marketing_funnel_daily` and `fact_prospect_journey`) and Actual measures (from `fact_leasing_daily`) is normal and reflects timing + multi-touch attribution differences. A reconciliation report is on the v1.1 roadmap.

- **Region averages unweighted:** Property-vs-region comparisons treat all properties equally regardless of unit count. A unit-weighted version is on the v1.1 roadmap.

- **Channel rollup uses spend-weighted VPI:** When viewing a Channel rather than a single Vendor, the displayed VPI is a spend-weighted average of vendor VPIs in that channel. This means a single high-spending vendor can dominate the channel score.

- **30-day window minimum applies to scores only:** Volume KPIs follow the date slicer literally, which can confuse users selecting tight date ranges. A future version will surface a "Scoring Window: <date> – <date>" subtitle when the minimum kicks in.

- **No mobile layout in v1.0:** Dashboard is built for desktop / tablet (1920×1080). Mobile view is planned for v1.2.

# Glossary

| Term | Definition |
|---|---|
| **Attributed Lease** | A new lease where the prospect's journey can be traced to one or more vendor touches |
| **Coverage Ratio** | Scheduled move-ins ÷ lease expirations over the next 60 days |
| **CPL** | Cost per Lead = Marketing Spend ÷ Leads |
| **Cost per Lease** | Marketing Spend ÷ New Leases |
| **Dense Rank** | Ranking method where ties get the same rank and the next rank is consecutive (1, 2, 2, 3) |
| **Competition Rank** | Ranking method where ties get the same rank but next rank skips (1, 2, 2, 4) |
| **Funnel Stage** | One of: Awareness, Consideration, Evaluation, Conversion |
| **GPI** | Geographic Performance Index (0–100) |
| **L2L** | Lead-to-Lease conversion percentage |
| **Pillar Balance** | Whether a GPI score is led by Absolute pillar, Trend pillar, or Balanced (within 5 pts) |
| **Ops Index** | Property-level operational health score (0–100) |
| **SLA** | Service Level Agreement — operational threshold a property must meet |
| **Sell Rate** | New Leases ÷ Available Units |
| **Vacancy Fill % (30D)** | Percentage of vacant units filled in the trailing 30-day window |
| **Vacant Ready %** | Vacant units ready to lease ÷ total vacant units |
| **VPI** | Vendor Performance Index (0–100) |

# Support

For data discrepancies, raise a ticket in the Analytics queue with the page name, visual name, screenshot, and your filter selections. For DAX-level questions, see the full measure documentation in `VPI_Documentation_v2.docx` and the `MAA_All_Measures_Current.tsv` measure inventory in the project repository.

---

*NorthStar Residential Group · Marketing Analytics · v1.0 · April 2026*
