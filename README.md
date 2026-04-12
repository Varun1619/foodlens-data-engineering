# FoodLens — Food Inspection Analytics

> Analyze, model, and visualize food inspection data from Chicago and Dallas to uncover public health insights and improve transparency.

---

## Team

| Name | Student ID | Email |
|---|---|---|
| Akshay Govind | 002509562 | govind.ak@northeastern.edu |
| Varun Singh | 002301584 | singh.varun3@northeastern.edu |
| Krupali Tejani | 002400000 | tejani.k@northeastern.edu |

---

## Project Overview

This project builds a complete data engineering and analytics pipeline on **Databricks** using the **Medallion Architecture** (Bronze → Silver → Gold) over food inspection datasets from two cities:

- **Chicago** — 308,161 inspections (2010–2026)
- **Dallas** — 78,984 inspections (2016–2024)

The pipeline covers data ingestion, profiling, cleansing, dimensional modeling, and BI dashboards.

---

## Repository Structure

```
foodlens-data-engineering/
├── 01_Bronze_Ingestion.ipynb       # Raw CSV ingestion into Bronze Delta tables
├── 02_DQX_Profiling.ipynb          # Data profiling using Databricks DQX
├── 03_Bronze_Silver.ipynb          # Data cleansing, validation, Silver layer
├── 04_Silver_to_Gold.ipynb         # Dimensional model load, Gold layer
└── README.md
```

---

## Architecture

```
Raw CSV Files
     │
     ▼
┌─────────────┐
│   BRONZE    │  Raw ingestion with metadata columns
│             │  chicago_inspections, dallas_inspections
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   SILVER    │  Cleansed, validated, DQX rules applied
│             │  chicago_inspections, dallas_inspections
│             │  chicago_violations, dallas_violations
└──────┬──────┘
       │
       ▼
┌─────────────┐
│    GOLD     │  Star schema dimensional model
│             │  7 dimensions + 2 fact tables
└─────────────┘
```

---

## Tools & Technologies

| Tool | Purpose |
|---|---|
| Databricks (Free Edition) | Data processing, notebooks, Delta Lake |
| Databricks DQX | Data quality profiling and validation |
| PySpark | Data transformations |
| Delta Lake | Storage format for all layers |
| Navicat | ER diagram and dimensional modeling |
| Power BI | BI dashboards and visualizations |
| Git / GitHub | Version control |

---

## Databricks Schema

| Layer | Schema |
|---|---|
| Bronze | `workspace.foodlens_bronze` |
| Silver | `workspace.foodlens_silver` |
| Gold | `workspace.foodlens_gold` |

---

## Notebooks

### 01 — Bronze Ingestion
- Reads Chicago and Dallas raw CSVs from Databricks Volumes
- Adds metadata columns: `_source_file`, `_load_timestamp`, `_source_city`
- Writes to Bronze Delta tables with column mapping enabled

### 02 — DQX Profiling
- Uses `DQProfiler` and `dbutils.data.summarize` for full profiling
- Null analysis, distribution analysis, schema comparison
- Documents data quality findings across both city datasets

### 03 — Bronze → Silver
- Parses Chicago violation strings (pipe-delimited) into structured fields
- Unpivots Dallas 25 violation slots into long format
- Derives Chicago violation scores from inspection results
- Generates SHA2 inspection_id for Dallas (no natural key)
- Applies all DQX validation rules (all as `error` — bad rows dropped)
- Trims Dallas violations to top 3 for inspections with score ≥ 90
- Deduplicates violations by content
- Writes 4 Silver tables + quarantine tables

### 04 — Silver → Gold
- Builds full star schema dimensional model
- Implements SCD Type 2 on `dim_establishment` using Delta Lake merge
- Loads `fact_inspection` and `fact_inspection_violation`
- Validates referential integrity (0 orphans across all FK relationships)

---

## Dimensional Model

### Dimensions
| Table | Description |
|---|---|
| `dim_date` | Calendar dimension (2000–2031) |
| `dim_location` | Address, city, state, zip, lat/long |
| `dim_inspection_type` | Inspection type names per city |
| `dim_result` | Inspection results / score buckets |
| `dim_risk_category` | Risk levels (Chicago only) |
| `dim_violation` | Unified violation codes and descriptions |
| `dim_establishment` | Business info with SCD Type 2 |

### Facts
| Table | Description |
|---|---|
| `fact_inspection` | One row per inspection (both cities) |
| `fact_inspection_violation` | Bridge table — inspections to violations |

### Gold Layer Row Counts
| Table | Rows |
|---|---|
| dim_date | 11,688 |
| dim_location | 36,947 |
| dim_inspection_type | 64 |
| dim_result | 10 |
| dim_risk_category | 3 |
| dim_violation | 116 |
| dim_establishment | 44,497 |
| fact_inspection | 294,164 |
| fact_inspection_violation | 1,332,652 |

---

## Silver Validation Rules

All rules applied using Databricks DQX — bad rows dropped to quarantine tables.

| Rule | City | Criticality |
|---|---|---|
| Restaurant Name not null | Both | error |
| Inspection Date not null | Both | error |
| Inspection Type not null | Both | error |
| Zip Code not null | Both | error |
| Zip Code valid format | Both | error |
| Chicago Results not null | Chicago | error |
| Dallas score ≤ 100 | Dallas | error |
| Dallas score ≥ 0 | Dallas | error |
| At least 1 violation per inspection | Both | error |
| Pass result cannot have Urgent/Critical violations | Both | error |
| Score ≥ 90 → max 3 violations (trimmed, not dropped) | Dallas | transform |

---

## Chicago Score Derivation

| Result | Derived Score |
|---|---|
| Pass | 90 |
| Pass w/ Conditions | 80 |
| Fail | 70 |
| No Entry | 0 |
| All other types | NULL |

---

## How to Run

1. Clone this repository into your Databricks workspace via Git integration
2. Attach a cluster (Databricks Runtime 13.x or higher recommended)
3. Run notebooks in order: `01` → `02` → `03` → `04`
4. All schemas are created automatically on first run
5. Widget defaults are pre-set — no manual configuration needed

---

## Deliverables

- [x] Data profiling with DQX documentation
- [x] Dimensional model (ER diagram via Navicat)
- [x] Source-to-Target mapping document
- [x] Bronze → Silver → Gold pipeline notebooks
- [x] SCD Type 2 on dim_establishment
- [x] BI Dashboard (Power BI)
- [x] Git repository with all code

---

*Northeastern University — Data Engineering Final Project — Spring 2026*
