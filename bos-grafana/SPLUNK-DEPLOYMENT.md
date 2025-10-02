# BOS Dashboard Deployment Guide for Splunk

## Overview

This guide provides step-by-step instructions for deploying BOS dashboards in Splunk environment using **SPL data generation scripts** (no file uploads required).

## Deployment Method: SPL Scripts (Recommended)

Instead of manually uploading CSV files, you'll **copy-paste SPL scripts** that create lookup tables directly in Splunk.

**Benefits:**
- ✅ No file upload hassle
- ✅ Air-gap friendly (plain text)
- ✅ Repeatable (rerun to refresh data)
- ✅ Self-documenting

---

## Prerequisites

- [ ] Splunk Enterprise access with Search & Reporting permissions
- [ ] Permission to create lookup tables
- [ ] Grafana instance with:
  - Grafana v11.1.3+ installed
  - HTML Graphics panel plugin (`gapit-htmlgraphics-panel`)
  - Splunk datasource plugin v5.6.1+

---

## Step 1: Create Lookup Tables (8 SPL Scripts)

### Instructions:

For each SPL script below:
1. **Copy the entire script** from GitHub
2. **Open Splunk** → Search & Reporting
3. **Paste the script** into search bar
4. **Click "Search"** to run
5. **Verify** the message showing lookup table created

### Required Scripts (Run in this order):

1. **`create-bos_services.spl`** (20 services)
   - Creates: `bos_services.csv` lookup table
   - Contains: Service catalog with L4/L3 hierarchy

2. **`create-bos_signal_status.spl`** (22 signals)
   - Creates: `bos_signal_status.csv` lookup table
   - Contains: Health signals (Green/Amber/Red status)

3. **`create-bos_incidents.spl`** (9 incidents)
   - Creates: `bos_incidents.csv` lookup table
   - Contains: Sample incident data

4. **`create-bos_sli_definitions.spl`** (12 SLIs)
   - Creates: `bos_sli_definitions.csv` lookup table
   - Contains: Service Level Indicator definitions

5. **`create-bos_slo_configurations.spl`** (9 SLOs)
   - Creates: `bos_slo_configurations.csv` lookup table
   - Contains: Service Level Objective targets

6. **`create-bos_sli_metrics.spl`** (1000 data points)
   - Creates: `bos_sli_metrics.csv` lookup table
   - Contains: Time-series SLI performance data
   - ⚠️ **Largest file** (159 KB) - may take 10-15 seconds to run

7. **`create-bos_stakeholder_expectations.spl`** (6 expectations)
   - Creates: `bos_stakeholder_expectations.csv` lookup table
   - Contains: Stakeholder expectations

8. **`create-bos_impact_indicators.spl`** (4 indicators)
   - Creates: `bos_impact_indicators.csv` lookup table
   - Contains: Business impact measurement definitions

---

## Step 2: Verify Lookup Tables

After running all 8 scripts, verify each lookup table exists:

```spl
| inputlookup bos_services.csv | head 5
```

Expected: 5 rows showing service data

**Run verification for all 8 tables:**

```spl
| inputlookup bos_services.csv | stats count
| inputlookup bos_signal_status.csv | stats count
| inputlookup bos_incidents.csv | stats count
| inputlookup bos_sli_definitions.csv | stats count
| inputlookup bos_slo_configurations.csv | stats count
| inputlookup bos_sli_metrics.csv | stats count
| inputlookup bos_stakeholder_expectations.csv | stats count
| inputlookup bos_impact_indicators.csv | stats count
```

**Expected counts:**
- bos_services: 20
- bos_signal_status: 22
- bos_incidents: 9
- bos_sli_definitions: 12
- bos_slo_configurations: 9
- bos_sli_metrics: 1000
- bos_stakeholder_expectations: 6
- bos_impact_indicators: 4

---

## Step 3: Configure Grafana Datasource UID

Before importing dashboards, you **must update the datasource UID** to match your Splunk datasource.

### Find Your Splunk Datasource UID:

1. In Grafana: **Settings → Data sources**
2. Click your Splunk datasource
3. **Copy UID from URL**: `/datasources/edit/{YOUR_UID}`
   - Example format: `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`

### Update Dashboard JSON Files:

The dashboard JSON files contain an embedded datasource UID. You must replace it with YOUR actual Splunk datasource UID.

**Search for this pattern in each JSON file:**
```json
"datasource": {
  "type": "grafana-splunk-datasource",
  "uid": "EXISTING_UID_HERE"
}
```

**Replace the UID value** with your actual datasource UID from the step above.

⚠️ **Important:** Each dashboard JSON has multiple datasource blocks. Use find-and-replace to update ALL occurrences.

**Dashboard files to update:**
- `l4-minimalist-splunk-v1.json`
- `l3-minimalist-splunk-v1.json`
- `l3-product-services-splunk-v1.json`
- `l3-service-detail-splunk-v1.json`

---

## Step 4: Import Dashboards

Import dashboards in this order (top-down navigation):

### 1. L4 Product Lines Dashboard
- **File:** `l4-minimalist-splunk-v1.json`
- **Import:** Dashboards → Import → Upload JSON file
- **UID:** `exec-l4-minimalist-splunk-v1`

### 2. L3 Products Dashboard
- **File:** `l3-minimalist-splunk-v1.json`
- **Import:** Dashboards → Import → Upload JSON file
- **UID:** `exec-l3-minimalist-splunk-v1`

### 3. L3 Product Services Dashboard
- **File:** `l3-product-services-splunk-v1.json`
- **Import:** Dashboards → Import → Upload JSON file
- **UID:** `exec-l3-product-services-splunk-v1`

### 4. Service Detail Dashboard
- **File:** `l3-service-detail-splunk-v1.json`
- **Import:** Dashboards → Import → Upload JSON file
- **UID:** `exec-l3-service-detail-splunk-v1`

---

## Step 5: Test Navigation Flow

Complete navigation flow: **L4 → L3 → Services → Service Detail**

### Test Path:

1. **Start:** Open L4 Product Lines dashboard
2. **Click:** "Home Lending" (should show 79.1% health, Red status)
3. **Navigate:** Opens L3 Products with 3 products
4. **Click:** "Home Originations"
5. **Navigate:** Opens Services with 3 services
6. **Click:** "Treasury Order Funding Service"
7. **Navigate:** Opens Service Detail with:
   - Service info, health metrics
   - Stakeholder expectations
   - Operational signals
   - Business impact indicators
   - SLI table
   - Active incidents

### Verify Breadcrumbs:

Each dashboard should show breadcrumb navigation:
- **L3:** L4 Product Lines › {L4 Name}
- **Services:** L4 Product Lines › {L4 Name} › {L3 Name}
- **Service Detail:** L4 Product Lines › {L4 Name} › {L3 Name} › {Service Name}

---

## Troubleshooting

### Issue: "No results found"

**Cause:** Lookup table not created or empty

**Fix:**
1. Verify lookup exists: `| inputlookup bos_services.csv | head 1`
2. If empty, rerun the SPL creation script
3. Check for SPL syntax errors in search.log

### Issue: Dashboard shows "No data available"

**Cause:** Dashboard variable not configured correctly

**Fix:**
1. Check dashboard variables (top right dropdown)
2. Verify `l4_product_line` variable loads options
3. Test variable query directly:
   ```spl
   | inputlookup bos_services.csv
   | stats count by l4_product_line
   | fields l4_product_line
   | sort l4_product_line
   ```

### Issue: Fields missing or misaligned

**Cause:** Column name mismatch in SPL script

**Fix:**
1. Check SPL script field names match dashboard queries
2. Common mismatches:
   - `sli_name` vs `sliName`
   - `signal_id` vs `signalId`
3. Verify with: `| inputlookup bos_services.csv | head 1 | transpose`

### Issue: Dashboard import fails

**Cause:** Datasource UID not updated or JSON syntax error

**Fix:**
1. Verify you updated ALL occurrences of datasource UID in JSON
2. Check JSON syntax (trailing commas, brackets)
3. Try importing via "Paste JSON" instead of file upload
4. Verify datasource plugin is installed and enabled

### Issue: SPL script shows "Error in 'outputlookup' command"

**Cause:** Insufficient permissions to create lookup tables

**Fix:**
1. Verify you have write permissions in Splunk
2. Check if lookup directory is writable
3. Contact Splunk admin for lookup creation permissions

---

## Data Refresh Workflow

To update data in the future:

1. **Get updated SPL scripts** from GitHub
2. **Rerun each script** in Splunk (replaces existing lookup table)
3. **Dashboards auto-refresh** (no reimport needed)

---

## Sample Data Details

**⚠️ Limited Sample Data:**
- Only 3 services have signals (SVC001, SVC002, SVC003)
- All in **Home Lending** L4 product line
- Other product lines show 0% coverage (expected for demo data)

**For testing:**
- Use "Home Lending" → "Home Originations" path
- This has complete data: services, signals, SLIs, incidents, expectations, impact

---

## Support

**Common Issues:** See `SPLUNK-TROUBLESHOOTING.md`

**SPL Translation:** See `SPLUNK-SQL-TRANSLATION-LESSONS.md` for SQL→SPL conversion patterns

**Questions:** Reference BOS documentation in repository README.md

---

## Quick Reference

**8 SPL Scripts (copy-paste order):**
1. ✅ `create-bos_services.spl`
2. ✅ `create-bos_signal_status.spl`
3. ✅ `create-bos_incidents.spl`
4. ✅ `create-bos_sli_definitions.spl`
5. ✅ `create-bos_slo_configurations.spl`
6. ✅ `create-bos_sli_metrics.spl` ⚠️ (159 KB - largest)
7. ✅ `create-bos_stakeholder_expectations.spl`
8. ✅ `create-bos_impact_indicators.spl`

**4 Dashboard JSONs (import order):**
1. ✅ `l4-minimalist-splunk-v1.json`
2. ✅ `l3-minimalist-splunk-v1.json`
3. ✅ `l3-product-services-splunk-v1.json`
4. ✅ `l3-service-detail-splunk-v1.json`

**Total deployment time:** ~15-20 minutes
