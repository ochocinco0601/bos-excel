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
1. Replace SQL `rawQueryText` in dashboard JSON with SPL query
2. Change datasource type to `grafana-splunk-datasource`
3. Update datasource UID to your Splunk datasource

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
- Splunk datasource configured
- Lookup tables uploaded (bos_services.csv, bos_signal_status.csv, bos_incidents.csv)

**Deployment Steps**:

1. **Download dashboard JSON**
   - Download `l4-minimalist-splunk-v1.json` from this directory

2. **Import to Grafana**
   - Go to Dashboards → Import
   - Upload JSON file or paste content
   - Dashboard will import with null datasource

3. **Configure panel datasource**
   - Edit the imported dashboard
   - Click panel title → Edit
   - Set datasource to your Splunk datasource

4. **Add SPL query (manual step required)**
   - HTML Graphics panels don't auto-import queries
   - Open `l4-dashboard-splunk-query.spl` in a text editor
   - Copy lines 14-61 (the main query)
   - Paste into the panel query editor in Grafana
   - Set query format to "Table"

5. **Test and save**
   - Click "Run query" to test
   - Verify 4 rows appear with correct data
   - Save dashboard

**Grafana v11 Requirements**:
- ✅ Flat JSON structure (no "dashboard" wrapper key)
- ✅ Manual query paste for HTML Graphics panels
- ✅ Datasource set to null for import compatibility

**Expected Results**:
- 4 rows: Auto Lending, Credit Cards, Home Lending, Personal Loans
- Home Lending: health=79.1%, status=Red
- Other 3: services_with_signals=0, coverage=0.0, health=null, status=Unknown

### L3 Dashboard Deployment (Grafana v11+)

**Dashboard JSON**: `l3-minimalist-splunk-v1.json` (Grafana v11+ flat structure)

**Prerequisites**:
- Grafana Enterprise/OSS v11.1.3+ (flat JSON format required)
- HTML Graphics panel plugin installed (`gapit-htmlgraphics-panel`)
- Splunk datasource configured
- Lookup tables uploaded (bos_services.csv, bos_signal_status.csv, bos_incidents.csv)

**Deployment Steps**:

1. **Download dashboard JSON**
   - Download `l3-minimalist-splunk-v1.json` from this directory

2. **Import to Grafana**
   - Go to Dashboards → Import
   - Upload JSON file or paste content
   - Dashboard will import with null datasource

3. **Configure panel datasource**
   - Edit the imported dashboard
   - Click panel title → Edit
   - Set datasource to your Splunk datasource

4. **Add SPL query (manual step required)**
   - HTML Graphics panels don't auto-import queries
   - Open `l3-dashboard-splunk-query.spl` in a text editor
   - Copy the main query (lines 14-73)
   - Paste into the panel query editor in Grafana
   - Set query format to "Table"

5. **Configure dashboard variable**
   - Go to Dashboard Settings → Variables
   - Add new variable:
     - Name: `l4_product_line`
     - Type: Query
     - Data source: Your Splunk datasource
     - Query: `| inputlookup bos_services.csv | stats count by l4_product_line | fields l4_product_line | sort l4_product_line`
     - Refresh: On Dashboard Load

6. **Test and save**
   - Select an L4 Product Line from the dropdown (e.g., "Home Lending")
   - Click "Run query" to test
   - Verify rows appear with correct data
   - Save dashboard

**Expected Results (Home Lending example)**:
- 3 rows: Mortgage Origination, Home Equity, Refinancing
- Each row shows: business_purpose, coverage, health status, incidents

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
