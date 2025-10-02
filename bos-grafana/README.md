# BOS Sample Data

Mock data demonstrating Business Observability System (BOS) methodology for correlating technical signals with business outcomes.

## Files

- `services.csv` - Service catalog with L4/L3 hierarchy
- `signal_status.csv` - Service health signals (Green/Amber/Red)
- `incidents.csv` - Sample incident data
- `l4-dashboard-splunk-query.spl` - Complete SPL query for L4 dashboard (Splunk datasource)
- `SPLUNK-SQL-TRANSLATION-LESSONS.md` - **Critical lessons learned translating SQL to SPL**

## Data Source

Generic banking industry services synthesized from publicly available information. Educational/demonstration use only.

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
