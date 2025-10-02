# BOS Sample Data

Mock data demonstrating Business Observability System (BOS) methodology for correlating technical signals with business outcomes.

## Files

- `services.csv` - Service catalog with L4/L3 hierarchy
- `signal_status.csv` - Service health signals (Green/Amber/Red)
- `incidents.csv` - Sample incident data
- `l4-dashboard-splunk-query.spl` - Complete SPL query for L4 dashboard (Splunk datasource)
- `l4-minimalist-splunk-v1.json` - Grafana L4 dashboard JSON (Grafana v11+ flat structure)
- `l3-dashboard-splunk-query.spl` - Complete SPL query for L3 dashboard (Splunk datasource)
- `l3-minimalist-splunk-v1.json` - Grafana L3 dashboard JSON (Grafana v11+ flat structure)
- `SPLUNK-SQL-TRANSLATION-LESSONS.md` - **Critical lessons learned translating SQL to SPL**
- `SPLUNK-TROUBLESHOOTING.md` - **Step-by-step diagnostic guide for "no results" issues**

## Data Source

Generic banking industry services synthesized from publicly available information. Educational/demonstration use only.

**⚠️ Limited Sample Data**: Only 3 services (SVC001, SVC002, SVC003) have signals, all in **Home Lending** L4 product line. Other product lines will show 0% coverage. See `SPLUNK-TROUBLESHOOTING.md` for testing guidance.

## Usage

Import into Grafana datasources (SQLite, Splunk lookups, etc.) for BOS dashboard development.

### SQLite Example

```bash
# Convert CSVs to SQLite database
python3 csv_to_sqlite.py

# Use with Grafana SQLite datasource
```

### Splunk Lookup Tables

**Setup:**
1. Upload CSVs to Splunk: Settings → Lookups → Lookup table files → Add new
   - Upload as: `bos_services.csv`, `bos_signal_status.csv`, `bos_incidents.csv`
   - Or rename lookups after upload
2. Create lookup definitions for each CSV file
3. Test: `| inputlookup bos_services.csv | head 5`

**L4 Dashboard Query:**

Use the complete SPL query in `l4-dashboard-splunk-query.spl` for the L4 Product Lines dashboard.

**Key points:**
- Query expects lookup tables prefixed with `bos_` (e.g., `bos_services.csv`)
- Modify lines 13, 16, 38 if using different names
- Test in Splunk Search before using in Grafana
- Returns 4 rows: Auto Lending, Credit Cards, Home Lending, Personal Loans

**L3 Dashboard Query:**

Use the complete SPL query in `l3-dashboard-splunk-query.spl` for the L3 Products dashboard.

**Key points:**
- Query expects lookup tables prefixed with `bos_` (e.g., `bos_services.csv`)
- Filters by L4 Product Line using dashboard variable `$l4_product_line$`
- Includes business_purpose field (l3_description from services.csv)
- Test in Splunk Search before using in Grafana (replace variable with actual value)
- Returns variable rows depending on L4 Product Line selected

**Grafana Integration:**
1. Replace SQL query in dashboard JSON with SPL query
2. Change datasource type to `grafana-splunk-datasource`
3. Update datasource UID to your Splunk datasource
4. Use Splunk v5.6.1 format: `query` field with `rawQuery: true` (NOT `queryText` or `format`)

**⚠️ Important for SQL → SPL Translation:**

If translating additional SQL queries to SPL, **read `SPLUNK-SQL-TRANSLATION-LESSONS.md` first**.

Key insights:
- Splunk `join` is 1-to-1 (not 1-to-many like SQL LEFT JOIN) - use `lookup` instead
- Multi-level aggregations require multiple `stats` commands
- Always handle null values explicitly (coalesce, fillnull)
- Multi-value fields need mvindex() extraction

These lessons prevent hours of debugging!

## Grafana Dashboard Deployment (Grafana v11+)

**Dashboard JSON**: `l4-minimalist-splunk-v1.json` (Grafana v11+ flat structure)

**Prerequisites**:
- Grafana Enterprise/OSS v11.1.3+ (flat JSON format required)
- HTML Graphics panel plugin installed (`gapit-htmlgraphics-panel`)
- Splunk datasource plugin v5.6.1+ installed
- Splunk datasource configured with UID: `d4c682f8-fd9b-46b2-9b89-e15410aa52dc`
- Lookup tables uploaded (bos_services.csv, bos_signal_status.csv, bos_incidents.csv)

**Deployment Steps**:

1. **Download dashboard JSON**
   - Download `l4-minimalist-splunk-v1.json` from this directory

2. **Import to Grafana**
   - Go to Dashboards → Import
   - Upload JSON file or paste content
   - Dashboard imports fully configured with datasource and queries
   - **That's it!** Dashboard is ready to use.

**Zero-Step Import**:
- ✅ Datasource pre-configured (UID: d4c682f8-fd9b-46b2-9b89-e15410aa52dc)
- ✅ SPL queries embedded in dashboard JSON (auto-import)
- ✅ Variables pre-configured (if applicable)
- ✅ No manual configuration required

**Grafana v11 + Splunk v5.6.1 Requirements**:
- ✅ Flat JSON structure (no "dashboard" wrapper key)
- ✅ Embedded datasource UID in targets
- ✅ Splunk v5.6.1 query format: `query` field with `rawQuery: true`
- ✅ No `queryText`/`rawQueryText` or `format` fields (deprecated)

**Expected Results**:
- 4 rows: Auto Lending, Credit Cards, Home Lending, Personal Loans
- Home Lending: health=79.1%, status=Red
- Other 3: services_with_signals=0, coverage=0.0, health=null, status=Unknown

### L3 Dashboard Deployment (Grafana v11+)

**Dashboard JSON**: `l3-minimalist-splunk-v1.json` (Grafana v11+ flat structure)

**Prerequisites**:
- Grafana Enterprise/OSS v11.1.3+ (flat JSON format required)
- HTML Graphics panel plugin installed (`gapit-htmlgraphics-panel`)
- Splunk datasource plugin v5.6.1+ installed
- Splunk datasource configured with UID: `d4c682f8-fd9b-46b2-9b89-e15410aa52dc`
- Lookup tables uploaded (bos_services.csv, bos_signal_status.csv, bos_incidents.csv)

**Deployment Steps**:

1. **Download dashboard JSON**
   - Download `l3-minimalist-splunk-v1.json` from this directory

2. **Import to Grafana**
   - Go to Dashboards → Import
   - Upload JSON file or paste content
   - Dashboard imports fully configured with datasource, queries, and variables
   - **That's it!** Dashboard is ready to use.

**Zero-Step Import**:
- ✅ Datasource pre-configured (UID: d4c682f8-fd9b-46b2-9b89-e15410aa52dc)
- ✅ SPL queries embedded in dashboard JSON (auto-import)
- ✅ Variable `l4_product_line` pre-configured with SPL query
- ✅ No manual configuration required

**Expected Results (Home Lending example)**:
- 3 rows: Mortgage Origination, Home Equity, Refinancing
- Each row shows: business_purpose, coverage, health status, incidents

### Services Dashboard Deployment (Grafana v11+)

**Dashboard JSON**: `l3-product-services-splunk-v1.json` (Grafana v11+ flat structure)

**Prerequisites**:
- Grafana Enterprise/OSS v11.1.3+ (flat JSON format required)
- HTML Graphics panel plugin installed (`gapit-htmlgraphics-panel`)
- Splunk datasource plugin v5.6.1+ installed
- Splunk datasource configured with UID: `d4c682f8-fd9b-46b2-9b89-e15410aa52dc`
- Lookup tables uploaded (bos_services.csv, bos_signal_status.csv, bos_incidents.csv)

**Deployment Steps**:

1. **Download dashboard JSON**
   - Download `l3-product-services-splunk-v1.json` from this directory

2. **Import to Grafana**
   - Go to Dashboards → Import
   - Upload JSON file or paste content
   - Dashboard imports fully configured with datasource, 2 queries, and 2 variables
   - **That's it!** Dashboard is ready to use.

**Zero-Step Import**:
- ✅ Datasource pre-configured (UID: d4c682f8-fd9b-46b2-9b89-e15410aa52dc)
- ✅ SPL Query A (service list) embedded in dashboard JSON
- ✅ SPL Query B (summary stats) embedded in dashboard JSON
- ✅ Variable `l4_product_line` pre-configured with SPL query
- ✅ Variable `l3_product` pre-configured with SPL query (depends on l4_product_line)
- ✅ No manual configuration required

**Expected Results (Home Lending / Mortgage Origination example)**:
- Shows services within the selected L3 product
- Each row: service name, business purpose, health %, open incidents
- Summary stats cards at top

**Navigation Flow (Complete)**:
- L4 → L3 drill-down: ✅ Working (clickable L4 product lines)
- L3 → Services drill-down: ✅ Working (clickable L3 products)
- Breadcrumb navigation: ✅ Working (Services → L3 → L4)

**Note:** Additional dashboard variants (Service Detail, Signal Detail) can be created for complete drill-down navigation.

**⚠️ Troubleshooting "No Results"?**

See `SPLUNK-TROUBLESHOOTING.md` for:
- Step-by-step diagnostic queries
- Common field name mismatches
- Variable configuration verification
- Minimal test queries to isolate issues

## Schema

### services.csv

| Column | Description |
|--------|-------------|
| service_id | Unique identifier (e.g., SVC001) |
| service_name | Technical service name |
| display_name | Human-readable name |
| business_purpose | What the service does |
| l4_product_line | Top-level business line |
| l3_product | Sub-product category |
| product_owner | Responsible owner email |

### signal_status.csv

| Column | Description |
|--------|-------------|
| signal_id | Unique signal identifier (e.g., SIG001) |
| signal_name | Human-readable signal name |
| signal_type | Category (Business, Process, System) |
| status | Current health (Green, Amber, Red) |
| service_id | Links to services.csv |

### incidents.csv

| Column | Description |
|--------|-------------|
| incident_id | Unique identifier (e.g., INC001) |
| severity | Severity level (Sev1, Sev2, Sev3, Sev4) |
| status | Current status (Open, Acknowledged, Resolved) |
| service_id | Affected service |
| l4_product_line | Business line affected |

## Data Model

The BOS data model connects:
- **Services** (what we provide)
- **Signals** (how we measure health)
- **Incidents** (what goes wrong)

This enables correlation of technical metrics with business impact.

## Related Resources

- BOS Methodology: Business observability as measurement of business outcome delivery
- Dashboard Patterns: L4 (Product Lines) → L3 (Products) → Services → Service Detail → Signal Detail
- Full data model: 11 CSV tables including SLI definitions, SLO configurations, and impact assessments

## License

Mock data for educational purposes. No warranty or guarantee of accuracy.
