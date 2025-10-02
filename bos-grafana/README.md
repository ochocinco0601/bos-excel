# BOS Sample Data

Mock data demonstrating Business Observability System (BOS) methodology for correlating technical signals with business outcomes.

## 🚀 Quick Start: SPL Deployment Method (Recommended)

**No file uploads needed!** Deploy BOS dashboards by copy-pasting SPL scripts directly into Splunk.

**See:** `SPLUNK-DEPLOYMENT.md` for complete step-by-step instructions.

**Quick workflow:**
1. Copy-paste 8 SPL scripts into Splunk (creates lookup tables automatically)
2. Update dashboard JSON datasource UIDs
3. Import 4 dashboard JSON files into Grafana
4. Done! (~15-20 minutes total)

**Benefits:**
- ✅ No CSV file uploads
- ✅ Air-gap friendly (plain text transfer)
- ✅ Repeatable data refresh
- ✅ Self-documenting

## Files

### SPL Data Generation Scripts (Recommended Method)
- `create-bos_services.spl` - Creates bos_services.csv lookup (20 services)
- `create-bos_signal_status.spl` - Creates bos_signal_status.csv lookup (22 signals)
- `create-bos_incidents.spl` - Creates bos_incidents.csv lookup (9 incidents)
- `create-bos_sli_definitions.spl` - Creates bos_sli_definitions.csv lookup (12 SLIs)
- `create-bos_slo_configurations.spl` - Creates bos_slo_configurations.csv lookup (9 SLOs)
- `create-bos_sli_metrics.spl` - Creates bos_sli_metrics.csv lookup (1000 data points)
- `create-bos_stakeholder_expectations.spl` - Creates bos_stakeholder_expectations.csv lookup (6 expectations)
- `create-bos_impact_indicators.spl` - Creates bos_impact_indicators.csv lookup (4 indicators)

### Dashboard JSON Files
- `l4-minimalist-splunk-v1.json` - Grafana L4 dashboard JSON (Grafana v11+ flat structure)
- `l3-minimalist-splunk-v1.json` - Grafana L3 dashboard JSON (Grafana v11+ flat structure)
- `l3-product-services-splunk-v1.json` - Grafana Services dashboard JSON (Grafana v11+ flat structure)
- `l3-service-detail-splunk-v1.json` - Grafana Service Detail dashboard JSON (Grafana v11+ flat structure)

### Dashboard Query Reference Files
- `l4-dashboard-splunk-query.spl` - Complete SPL query for L4 dashboard (Splunk datasource)
- `l3-dashboard-splunk-query.spl` - Complete SPL query for L3 dashboard (Splunk datasource)
- `services-dashboard-splunk-query.spl` - Complete SPL queries for Services dashboard (Splunk datasource)
- `service-detail-dashboard-splunk-query.spl` - Complete SPL queries for Service Detail dashboard (Splunk datasource)

### CSV Files (Optional - Use SPL Scripts Instead)
- `services.csv` - Service catalog with L4/L3 hierarchy
- `signal_status.csv` - Service health signals (Green/Amber/Red)
- `incidents.csv` - Sample incident data

### Documentation
- `SPLUNK-DEPLOYMENT.md` - **Complete step-by-step deployment guide using SPL scripts**
- `SPLUNK-SQL-TRANSLATION-LESSONS.md` - **Critical lessons learned translating SQL to SPL**
- `SPLUNK-TROUBLESHOOTING.md` - **Step-by-step diagnostic guide for "no results" issues**

## Data Source

Generic banking industry services synthesized from publicly available information. Educational/demonstration use only.

**⚠️ Limited Sample Data**: Only 3 services (SVC001, SVC002, SVC003) have signals, all in **Home Lending** L4 product line. Other product lines will show 0% coverage. See `SPLUNK-TROUBLESHOOTING.md` for testing guidance.

## Usage

### Recommended: SPL Script Deployment (Splunk)

**No file uploads needed!** See `SPLUNK-DEPLOYMENT.md` for complete instructions.

1. Copy-paste 8 SPL scripts into Splunk Search & Reporting
2. Each script creates a lookup table automatically
3. Import 4 dashboard JSON files into Grafana
4. Done!

### Alternative: SQLite Example

```bash
# Convert CSVs to SQLite database
python3 csv_to_sqlite.py

# Use with Grafana SQLite datasource
```

### Alternative: Manual CSV Upload (Splunk)

**⚠️ Not recommended** - Use SPL scripts instead (see above)

If you must upload CSVs manually:
1. Upload CSVs to Splunk: Settings → Lookups → Lookup table files → Add new
   - Upload as: `bos_services.csv`, `bos_signal_status.csv`, `bos_incidents.csv`, etc.
   - ⚠️ **Important:** CSV files contain `#` comment lines that Splunk does NOT ignore
   - You must remove all lines starting with `#` before uploading
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
- Splunk datasource configured in your Grafana instance
- Lookup tables uploaded (bos_services.csv, bos_signal_status.csv, bos_incidents.csv)

**Deployment Steps**:

1. **Configure datasource UID**
   - Before importing, update the datasource UID in the JSON file to match your Splunk datasource
   - Find your datasource UID: Settings → Data sources → [Your Splunk datasource] → Copy UID from URL
   - Replace all instances of the placeholder UID in the JSON file with your actual UID

2. **Import to Grafana**
   - Go to Dashboards → Import
   - Upload modified JSON file or paste content
   - Dashboard imports with queries and variables pre-configured
   - Select your Splunk datasource if prompted

**What's Pre-configured**:
- ✅ SPL queries embedded in dashboard JSON
- ✅ Variables pre-configured (if applicable)
- ✅ Grafana v11+ flat structure format
- ⚠️ **Datasource UID must be updated** before import to match your environment

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
- Splunk datasource configured in your Grafana instance
- Lookup tables uploaded (bos_services.csv, bos_signal_status.csv, bos_incidents.csv)

**Deployment Steps**:

1. **Configure datasource UID**
   - Before importing, update the datasource UID in the JSON file to match your Splunk datasource
   - Find your datasource UID: Settings → Data sources → [Your Splunk datasource] → Copy UID from URL
   - Replace all instances of the placeholder UID in the JSON file with your actual UID

2. **Import to Grafana**
   - Go to Dashboards → Import
   - Upload modified JSON file or paste content
   - Dashboard imports with queries and variables pre-configured
   - Select your Splunk datasource if prompted

**What's Pre-configured**:
- ✅ SPL queries embedded in dashboard JSON
- ✅ Variable `l4_product_line` pre-configured with SPL query
- ✅ Grafana v11+ flat structure format
- ⚠️ **Datasource UID must be updated** before import to match your environment

**Expected Results (Home Lending example)**:
- 3 rows: Mortgage Origination, Home Equity, Refinancing
- Each row shows: business_purpose, coverage, health status, incidents

### Services Dashboard Deployment (Grafana v11+)

**Dashboard JSON**: `l3-product-services-splunk-v1.json` (Grafana v11+ flat structure)

**Prerequisites**:
- Grafana Enterprise/OSS v11.1.3+ (flat JSON format required)
- HTML Graphics panel plugin installed (`gapit-htmlgraphics-panel`)
- Splunk datasource plugin v5.6.1+ installed
- Splunk datasource configured in your Grafana instance
- Lookup tables uploaded (bos_services.csv, bos_signal_status.csv, bos_incidents.csv)

**Deployment Steps**:

1. **Configure datasource UID**
   - Before importing, update the datasource UID in the JSON file to match your Splunk datasource
   - Find your datasource UID: Settings → Data sources → [Your Splunk datasource] → Copy UID from URL
   - Replace all instances of the placeholder UID in the JSON file with your actual UID

2. **Import to Grafana**
   - Go to Dashboards → Import
   - Upload modified JSON file or paste content
   - Dashboard imports with queries and variables pre-configured
   - Select your Splunk datasource if prompted

**What's Pre-configured**:
- ✅ SPL Query A (service list) embedded in dashboard JSON
- ✅ SPL Query B (summary stats) embedded in dashboard JSON
- ✅ Variable `l4_product_line` pre-configured with SPL query
- ✅ Variable `l3_product` pre-configured with SPL query (depends on l4_product_line)
- ✅ Grafana v11+ flat structure format
- ⚠️ **Datasource UID must be updated** before import to match your environment

**Expected Results (Home Lending / Mortgage Origination example)**:
- Shows services within the selected L3 product
- Each row: service name, business purpose, health %, open incidents
- Summary stats cards at top

**Navigation Flow (Complete)**:
- L4 → L3 drill-down: ✅ Working (clickable L4 product lines)
- L3 → Services drill-down: ✅ Working (clickable L3 products)
- Services → Service Detail drill-down: ✅ Working (clickable service names)
- Breadcrumb navigation: ✅ Working (Service Detail → Services → L3 → L4)

### Service Detail Dashboard Deployment (Grafana v11+)

**Dashboard JSON**: `l3-service-detail-splunk-v1.json` (Grafana v11+ flat structure)

**Prerequisites**:
- Grafana Enterprise/OSS v11.1.3+ (flat JSON format required)
- HTML Graphics panel plugin installed (`gapit-htmlgraphics-panel`)
- Splunk datasource plugin v5.6.1+ installed
- Splunk datasource configured in your Grafana instance
- Lookup tables uploaded (bos_services.csv, bos_signal_status.csv, bos_incidents.csv, bos_sli_definitions.csv, bos_slo_configurations.csv, bos_sli_metrics.csv, bos_stakeholder_expectations.csv, bos_impact_indicators.csv)

**Deployment Steps**:

1. **Configure datasource UID**
   - Before importing, update the datasource UID in the JSON file to match your Splunk datasource
   - Find your datasource UID: Settings → Data sources → [Your Splunk datasource] → Copy UID from URL
   - Replace all instances of the placeholder UID in the JSON file with your actual UID

2. **Import to Grafana**
   - Go to Dashboards → Import
   - Upload modified JSON file or paste content
   - Dashboard imports with queries and variables pre-configured
   - Select your Splunk datasource if prompted

**What's Pre-configured**:
- ✅ SPL Query A (service info + health) embedded in dashboard JSON
- ✅ SPL Query B (operational signals) embedded in dashboard JSON
- ✅ SPL Query C (open incidents) embedded in dashboard JSON
- ✅ SPL Query D (SLI metrics) embedded in dashboard JSON
- ✅ SPL Query E (stakeholder expectations) embedded in dashboard JSON
- ✅ SPL Query F (business impact indicators) embedded in dashboard JSON
- ✅ Variable `l4_product_line` pre-configured with SPL query
- ✅ Variable `l3_product` pre-configured with SPL query (depends on l4_product_line)
- ✅ Variable `service_id` pre-configured with SPL query (depends on l4_product_line and l3_product)
- ✅ Grafana v11+ flat structure format
- ⚠️ **Datasource UID must be updated** before import to match your environment

**Dashboard Sections**:
1. **Service Header**: Service name, business purpose, breadcrumb navigation
2. **Health Metrics**: Current health score (signal-based), open incidents count
3. **Stakeholder Expectations**: Who cares about this service and what they expect
4. **Operational Signals**: Technical signals (Business, Process, System) with current status
5. **Business Impact**: Estimated impact on customer, financial, legal/risk, operational dimensions
6. **SLI Table**: Service Level Indicators with current vs target values
7. **Active Incidents**: Open incidents with severity and status

**Expected Results (Service SVC001 example)**:
- Service info: Display name, business purpose, health percentage
- Signals table: Multiple rows showing Business/Process/System signals
- SLI table: Performance metrics with current values and targets
- Business impact cards: Customer, Financial, Legal/Risk, Operational impacts
- Incidents table: Any open incidents for this service

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
