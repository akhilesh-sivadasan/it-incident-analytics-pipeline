# IT Support Desk: Incident Analytics & ETL Pipeline

> End-to-end analytics pipeline identifying SLA bottlenecks in IT helpdesk operations — from raw data ingestion through automated dashboard refresh.

---

## ⚡ TL;DR

- **The finding that matters:** Database-category incidents take ~7 days to resolve end-to-end, but only 5.15 of those days are spent with the Database Administration team. The missing ~2 days are tickets bouncing through wrong queues before reaching DBA — **~26% of Database MTTR is recoverable through routing fixes alone, with zero change to team capacity.**
- **What was built:** An automated Excel Power Query ETL pipeline feeding a Pivot Data Model and interactive dashboard with priority-level slicers.
- **Business impact:** 2–4 analyst-hours saved per weekly refresh · cadence shift from monthly to on-demand · ~1.8 days of Database MTTR recoverable if routing recommendation is actioned.

---

## 📊 Dashboard Preview

![Dashboard](dashboard_view.png)

*Interactive Excel dashboard: MTTR by Category, Ticket Volume by Priority, and Average Resolution Time by Assignment Group — filterable by priority level via slicers.*

---

## 🔍 Key Business Insights

### 1. Hidden Routing Inefficiency *(Primary Bottleneck)*
Database incidents average ~7 days end-to-end, but DBA only accounts for 5.15 of those days. The ~2-day delta represents tickets being mis-routed through Tier-1, Desktop Support, or Network Operations before reaching the correct team. **Fixing front-line triage logic could recover ~26% of total Database MTTR with zero change to DBA capacity.**

### 2. Team-Level Capacity Gap
Even after correct routing, Database Administration averages 5.15 days vs. 2–3 days for peer infrastructure teams. This residual gap points to a secondary capacity or complexity issue worth investigating separately from the routing fix above.

### 3. Priority Inversion on Critical Tickets
When filtered to Critical-priority tickets (8% of volume), Database Administration and Cloud Infrastructure remain the slowest responders. Critical tickets should be *fastest*, not slowest — this indicates a need for dedicated high-priority escalation protocols on these two teams.

---

## 🚀 Business Impact

| Metric | Impact |
|---|---|
| Manual cleanup effort | **2–4 analyst-hours saved per weekly refresh** |
| Refresh cadence | **Monthly → on-demand** (pipeline re-runs on file reload) |
| Recoverable MTTR (if routing action is taken) | **~1.8 days per Database ticket** — roughly one working week per quarter of recovered DBA time |

By automating ingestion and cleanup via Power Query, the IT management team no longer manually cleans weekly helpdesk exports. The pipeline ingests new data, refreshes the data model, and updates the dashboard on reload — freeing analyst time for interpretation rather than preparation.

---

## 🛠️ Tech Stack & Methodology

### Data Engineering (ETL)
Automated pipeline built in **Excel Power Query** (M language):
- Text transformations to standardize inconsistent category naming (`HW` → `hardware`, `DB` → `database`, etc.)
- Custom duration logic calculating resolution time in days, filtering negative-duration records as logic errors
- Null-handling for active / on-hold tickets — excluded from MTTR calculations but retained in volume metrics

### Data Modeling & Visualization
- **Excel Pivot Data Model** aggregating MTTR across Category × Assignment Group × Priority
- Interactive dashboard with **Slicers** for real-time priority-level filtering
- Chart types: column (MTTR by Category), pie (Volume by Priority), horizontal bar (MTTR by Assignment Group)

### Design Choice: Naming Conventions
Data-layer values are normalized to **lowercase snake_case** for SQL-compatible downstream use; the presentation layer applies **title-case** formatting for readability. This keeps the model portable if the pipeline is later migrated to a SQL warehouse.

### Dataset
1,000 synthetic incident records modeled on realistic ITSM distributions, with intentionally embedded data quality issues — inconsistent categoricals, negative-duration timestamps, and nulls for active tickets — designed to **stress-test the ETL pipeline** and simulate the messiness of raw helpdesk exports.

---

## 🧹 Data Quality Handling

| Issue | Frequency | Pipeline Handling |
|---|---|---|
| Inconsistent category naming (`HW`, `hardware`, `Hard Ware`, `DB`, `Net`, etc.) | ~3.9% of rows | Normalized via `Text.Lower` + lookup standardization |
| `resolved_at` earlier than `opened_at` (logic errors) | ~2% of rows | Filtered from duration calculations; flagged for triage review |
| Null `resolved_at` (On Hold / active tickets) | ~15% of rows | Excluded from MTTR metrics; retained in volume metrics |
