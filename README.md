# Inventory Availability & Profit Protection Dashboard

**Excel · Power BI · DAX · Star Schema · UK Retail**

---

## Overview

This project delivers a two-page interactive Power BI dashboard that gives retail operations managers a single, consolidated view of whether inventory availability is supporting — or undermining — sales and gross profit performance.

The dataset spans **150 active products** across **40 locations** (Stores, Warehouses, and Fulfilment Centres) in **6 UK regions** over a **two-year period (July 2024 – June 2026)**. The solution is built on a fully validated **five-table constellation star schema**, documented end-to-end with a structured data dictionary and a 29-point validation log.

---

## Business Problem

Retail managers had no single view linking stock availability to sales and profit outcomes. Stockouts were going undetected across regions while excess stock was simultaneously tying up capital at other locations — suggesting a **distribution and allocation problem**, not a company-wide supply failure.

---

## Business Questions Answered

1. What is the overall sales and gross profit performance across the estate?
2. Which departments generate the most net sales and contribute the most gross profit?
3. How does performance trend month-over-month across the full two-year period?
4. What percentage of inventory snapshots are flagged as stockouts, and where is the risk highest?
5. Which regions carry the greatest stockout exposure?
6. Which specific product–location combinations need immediate replenishment or stock rebalancing?

---

## Tools Used

| Tool | Purpose |
|------|---------|
| Microsoft Excel | Data dictionary, 29-point validation log, baseline KPI verification |
| Power BI Desktop | Star schema data model, interactive two-page dashboard |
| DAX | 11 KPI measures — aggregations, CALCULATE filters, DIVIDE for rates |
| Power Query | Data type enforcement, column selection, query hygiene before model load |

---

## Data & Methodology

### Dataset Structure

| Table | Type | Rows | Description |
|-------|------|------|-------------|
| `Fact_SalesLine` | Fact | 5,000 | One row per sales transaction line — quantity, net sales, gross profit |
| `Fact_InventorySnapshot` | Fact | 5,000 | Periodic stock snapshots — available quantity, inventory value, stockout and excess flags |
| `Dim_Product` | Dimension | 150 | Product catalogue — department, name, perishable and private label flags |
| `Dim_Location` | Dimension | 40 | Location master — region, city, location type, status |
| `Dim_Date` | Dimension | 730 | Continuous calendar from 01 Jul 2024 to 30 Jun 2026 |

**Total: 10,730 rows across 5 tables**

---

### Data Model

A clean **constellation star schema** — two fact tables sharing three dimension tables via single-direction (One-to-Many) relationships. No direct relationship exists between the two fact tables. `Dim_Date` is marked as a date table; `MonthName` is sorted by `MonthNumber` for correct chart ordering.

![Data Model](images/03_data_model.png)

---

### Data Validation — 29 Checks, All Passed

All checks were logged and verified in Excel before Power BI build began.

| Category | Scope | Result |
|----------|-------|--------|
| Row counts | All 5 tables | ✅ Sales 5,000 · Inventory 5,000 · Products 150 · Locations 40 · Dates 730 |
| Primary key uniqueness | All 5 PKs | ✅ Zero duplicates |
| Blank values in critical columns | Keys, amounts, flags | ✅ Zero blanks in blocking columns |
| Date continuity | `Dim_Date` | ✅ 730 consecutive dates, no gaps |
| Sales arithmetic | 4 checks × 5,000 rows | ✅ All rows within £0.01 tolerance |
| Foreign key resolution | 6 FK relationships | ✅ Every fact key resolves to a dimension |
| Flag domain validation | `StockoutFlag`, `ExcessStockFlag` | ✅ Only 0 or 1 present |
| Baseline reconciliation (Excel vs Power BI) | 3 core KPIs | ✅ Exact match |

---

### Baseline KPIs (Verified)

| KPI | Value |
|-----|-------|
| Net Sales | £69,045.88 |
| Gross Profit | £33,917.38 |
| Gross Margin % | 49.1% |
| Inventory Value | £705,107.90 |
| Stockout Rate | 8.0% (400 / 5,000 snapshots) |
| Excess Stock Rate | 12.0% (600 / 5,000 snapshots) |

---

### DAX Measures

```dax
-- Sales
Net Sales      = SUM(Fact_SalesLine[NetSalesAmount])
Gross Profit   = SUM(Fact_SalesLine[GrossProfitAmount])
Gross Margin % = DIVIDE([Gross Profit], [Net Sales])
Units Sold     = SUM(Fact_SalesLine[Quantity])
Transactions   = DISTINCTCOUNT(Fact_SalesLine[TransactionID])

-- Inventory
Inventory Value        = SUM(Fact_InventorySnapshot[InventoryValue])
Inventory Snapshots    = COUNTROWS(Fact_InventorySnapshot)
Stockout Snapshots     = CALCULATE([Inventory Snapshots], Fact_InventorySnapshot[StockoutFlag] = 1)
Stockout Rate          = DIVIDE([Stockout Snapshots], [Inventory Snapshots])
Excess Stock Snapshots = CALCULATE([Inventory Snapshots], Fact_InventorySnapshot[ExcessStockFlag] = 1)
Excess Stock Rate      = DIVIDE([Excess Stock Snapshots], [Inventory Snapshots])
```

---

## Power BI Dashboard

### Page 1 — Overview

Four interactive slicers: **Date Range · Department · Region · Location Type**  
Page-level filters applied: Active products and Active locations only.

![Dashboard Overview](images/01_dashboard_overview.png)

| KPI | Full-Range Value |
|-----|-----------------|
| Net Sales | £69.05K |
| Gross Profit | £33.92K |
| Gross Margin % | 49.1% |
| Inventory Value | £705.11K |
| Stockout Rate | 8.0% |
| Excess Stock Rate | 12.0% |
| Units Sold | ~11K |
| Inventory Snapshots | 5K |

---

### Page 2 — Analysis

![Analysis Page](images/02_analysis_page.png)

| Visual | Type | Business Question |
|--------|------|-------------------|
| Net Sales & Gross Profit by Department | Clustered bar chart | Which department drives the most value? |
| Net Sales & Gross Profit by Month | Clustered column with year overlay | How does performance trend over time? |
| Stockout Rate by Region | Horizontal bar chart | Which regions are most exposed? |
| Product–Location Action List | Filtered interactive table | Which product–location pairs need action now? |

---

## Key Findings

**1. 49.1% Gross Margin — Commercially Healthy**  
Net Sales of £69.0K and Gross Profit of £33.9K over two years deliver a near-50% gross margin, indicating a well-structured pricing model and a strong product mix at portfolio level.

**2. Fashion Delivers the Strongest Profit Contribution**  
Fashion generates approximately £13.9K gross profit from £25.0K in net sales — the highest profit efficiency of all five departments and the highest priority to protect from avoidable stockouts. Grocery leads on sales volume (~£27K) but at lower margins.

**3. A Distribution Problem, Not a Supply Problem**  
8% of inventory snapshots show zero available stock (400 stockout rows) while 12% show excess stock (600 rows). These two conditions co-exist across the estate simultaneously, pointing to a stock allocation and distribution failure rather than a company-wide supply shortage.

**4. Electronics Carries the Highest Department Stockout Risk (~9.1%)**  
As a high-value, higher-margin category, undetected stockouts in Electronics represent the greatest financial exposure across all five departments. Replenishment settings and lead times should be reviewed here first.

**5. Yorkshire and North West Lead Regional Stockout Exposure (~8.6% each)**  
Regional stockout rates vary measurably across all six UK regions. Yorkshire and the North West are consistently the most exposed, confirming that the risk is geographically concentrated and suited to a targeted, location-first intervention.

**6. £705K Inventory Value Warrants a Turnover Conversation**  
Total inventory value across the estate is £705K against £69K of period net sales. Without agreed turnover targets it is premature to call this inefficient, but the ratio — combined with a 12% excess stock rate — indicates capital concentration risk that merits stakeholder review.

**7. 60 Negative-Profit Lines Identified**  
Sixty individual sales lines carry a negative gross profit (combined impact approximately –£28). The financial exposure is negligible, but the lines should be reviewed for discounting policy or data-entry causes before they become a pattern.

---

## Business Recommendations

| Priority | Recommendation |
|----------|---------------|
| 🔴 High | Build a **weekly stockout exception list** of active products above the 8.0% portfolio average, ranked by Net Sales and Gross Profit, for immediate replenishment action. |
| 🔴 High | **Review Electronics replenishment settings and supplier lead times first** — highest stockout rate in the highest-margin category equals the highest revenue risk. |
| 🟠 Medium | **Start regional location reviews with Yorkshire and the North West**, then drill to individual store-product combinations using the Product–Location Action List. |
| 🟠 Medium | **Identify inter-location transfer opportunities** — where the same SKU is stocked out at one location and in excess at another, stock transfers are the fastest and lowest-cost resolution. |
| 🟡 Low | Investigate the **60 negative-profit sales lines** for discounting policy or data-entry root causes before the next reporting cycle. |
| 🟡 Low | Work with stakeholders to **agree acceptable thresholds** for stockout and excess rates, enabling the dashboard to move from descriptive monitoring to target-driven performance management. |

---

## Project Structure

```
inventory-profit-protection/
│
├── README.md
│
├── data/
│   └── 01_Source_Omnichannel_Inventory.xlsx       ← Source data
│
├── excel/
│   └── Inventory_Profit_Protection_Working.xlsx
│       ├── 00_README                              ← Project scope and metadata
│       ├── 01_DATA_DICTIONARY                     ← All 5 tables fully documented
│       └── 02_DATA_VALIDATION                     ← 29 validation checks (all pass)
│
├── power-bi/
│   └── Inventory_Profit_Protection.pbix           ← Full Power BI report
│
├── images/
│   ├── 01_dashboard_overview.png                  ← Page 1: KPI cards and slicers
│   ├── 02_analysis_page.png                       ← Page 2: Charts and action table
│   └── 03_data_model.png                          ← Star schema model view
│
└── documentation/
    └── Inventory_Profit_Protection_Guide.pdf      ← Step-by-step execution guide
```

---

## Skills Demonstrated

| Category | Detail |
|----------|--------|
| Data Modelling | Constellation star schema · dual-fact architecture · single-direction relationships · date table configuration |
| DAX | `SUM` · `CALCULATE` · `COUNTROWS` · `DISTINCTCOUNT` · `DIVIDE` · measure formatting and organisation in a dedicated Measures Table |
| Power Query | Data type enforcement · column trimming · query management before model load |
| Data Quality | 29-point structured validation log · row count verification · FK resolution · arithmetic tolerance testing · Excel-to-Power BI baseline reconciliation |
| Excel | Structured documentation (README · Data Dictionary · Validation Log) · COUNTIF and SUM-based validation formulas |
| Dashboard Design | KPI card layout · interactive slicer design · filter synchronisation across pages · chart type selection matched to each business question |
| Analytical Thinking | Translating stockout and excess stock metrics into prioritised, business-ready recommendations |

---

## Notes

- **Date Range:** 01 July 2024 – 30 June 2026 (730 continuous days)
- **Scope:** Active products and active locations only (applied as page-level filters on both report pages)
- **Out of Scope:** Customer data, supplier records, promotions, forecasts, returns, purchase orders
- **Stockout Rate:** A snapshot-based measure (share of inventory snapshot rows where `StockoutFlag = 1`), not an estimate of lost sales revenue
- **Dataset:** Synthetic dataset created for portfolio demonstration purposes

---

*Muhammad Danish · Data Analyst · Birmingham, UK*
