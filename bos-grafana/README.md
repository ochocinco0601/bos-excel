# BOS Sample Data

Mock data demonstrating Business Observability System (BOS) methodology for correlating technical signals with business outcomes.

## Files

- `services.csv` - Service catalog with L4/L3 hierarchy
- `signal_status.csv` - Service health signals (Green/Amber/Red)
- `incidents.csv` - Sample incident data

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

```
1. Upload CSVs to Splunk: Settings → Lookups → Lookup table files → Add new
2. Create lookup definitions for each CSV file
3. Use in Grafana with Splunk datasource

Example SPL:
| inputlookup services.csv
| join service_id [| inputlookup signal_status.csv]
| stats count by l4_product_line, status
```

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
