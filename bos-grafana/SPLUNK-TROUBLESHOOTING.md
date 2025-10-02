# Splunk Query Troubleshooting Guide

## Data Availability

The sample BOS data has **limited signals**:
- **Only 3 services have signals**: SVC001, SVC002, SVC003
- **All are in Home Lending L4 product line**
- **L3 products**: Home Originations (2 services), Home Servicing (1 service)

**This means:**
- L4 dashboard: Only "Home Lending" row will have health data
- L3 dashboard: Only works when filtering by `l4_product_line="Home Lending"`
- Other product lines: Will show 0% coverage, null health

## L3 Dashboard - No Results Troubleshooting

### Step 1: Verify Lookup Tables Exist

```spl
| inputlookup bos_services.csv | head 5
```

**Expected:** 5 rows with service data

**If no results:**
- Tables not uploaded to Splunk
- Wrong table names (should be `bos_services.csv`, `bos_signal_status.csv`, `bos_incidents.csv`)

### Step 2: Test Simple L3 Query (No Variable)

```spl
| inputlookup bos_services.csv
| where l4_product_line="Home Lending"
| stats count by l3_product
```

**Expected Output:**
```
l3_product            count
Home Originations     2
Home Servicing        1
(other L3 products)   (counts for other services without signals)
```

**If no results:**
- Field name case-sensitive mismatch
- CSV headers don't match query field names
- Check: `| inputlookup bos_services.csv | head 1` to see actual field names

### Step 3: Test Signal Lookup Join

```spl
| inputlookup bos_signal_status.csv
| lookup bos_services.csv service_id OUTPUT l3_product l4_product_line
| where l4_product_line="Home Lending"
| stats count by service_id l3_product
```

**Expected Output:**
```
service_id  l3_product          count
SVC001      Home Originations   9
SVC002      Home Originations   8
SVC003      Home Servicing      5
```

**If no results:**
- Lookup join failed (service_id field mismatch)
- Signal_status.csv not loaded
- Check: `| inputlookup bos_signal_status.csv | head 5`

### Step 4: Variable Substitution Test

If using dashboard variable `$l4_product_line$`:

```spl
| inputlookup bos_services.csv
| where l4_product_line="$l4_product_line$"
| stats count
```

**Set variable to:** `Home Lending` (exact case match)

**Expected:** Non-zero count

**If zero results:**
- Variable not set
- Variable value doesn't match exactly (case-sensitive)
- Try hardcoded value instead: `l4_product_line="Home Lending"`

## Common Issues

### Issue 1: Wrong Lookup Table Names

**Query expects:** `bos_services.csv`, `bos_signal_status.csv`, `bos_incidents.csv`

**If you named them differently**, update the query:
```spl
# Change this:
| inputlookup bos_services.csv

# To this (your actual name):
| inputlookup services.csv
```

### Issue 2: Field Name Case Sensitivity

Splunk fields are **case-sensitive**. The query uses:
- `l4_product_line` (lowercase L, underscore)
- `l3_product` (lowercase L, underscore)
- `l3_description`
- `service_id`
- `status`

Run this to verify actual field names:
```spl
| inputlookup bos_services.csv
| head 1
| transpose
```

### Issue 3: No Data for Selected L4 Product Line

Only **Home Lending** has signals in the sample data.

**Test with hardcoded value:**
```spl
| inputlookup bos_signal_status.csv
| lookup bos_services.csv service_id OUTPUT l3_product l4_product_line
| where l4_product_line="Home Lending"  # Hardcode for testing
| stats count
```

### Issue 4: CSV Files Not Properly Uploaded

Verify CSV files loaded correctly:

```spl
| inputlookup bos_services.csv | stats count
```

**Expected:** 20 rows

```spl
| inputlookup bos_signal_status.csv | stats count
```

**Expected:** 22 rows

```spl
| inputlookup bos_incidents.csv | stats count
```

**Expected:** 10+ rows

## Minimal Test Query for L3

This is the **simplest possible L3 query** to verify data:

```spl
| inputlookup bos_services.csv
| where l4_product_line="Home Lending"
| stats
    dc(service_id) as service_count
    by l3_product
| sort l3_product
```

**Expected Output:**
```
l3_product              service_count
Home Equity             (count)
Home Originations       2
Home Servicing          1
```

If this works, then progressively add complexity until you find what breaks.

## Dashboard Variable Configuration

In Grafana, the L3 dashboard needs a variable:

**Variable Setup:**
- Name: `l4_product_line`
- Type: Query
- Data source: Your Splunk datasource
- Query: `| inputlookup bos_services.csv | stats count by l4_product_line | fields l4_product_line | sort l4_product_line`
- Refresh: On Dashboard Load

**Test the variable query separately in Splunk:**
```spl
| inputlookup bos_services.csv
| stats count by l4_product_line
| fields l4_product_line
| sort l4_product_line
```

**Expected:** 4 rows (Auto Lending, Credit Cards, Home Lending, Personal Loans)

## Full L3 Query with Diagnostic Comments

Here's the full query with comments showing expected results at each stage:

```spl
# Step 1: Get all signals - Expected: 22 rows
| inputlookup bos_signal_status.csv

# Step 2: Join service metadata - Expected: 22 rows with l3_product, l4_product_line
| lookup bos_services.csv service_id OUTPUT l3_product l3_description l4_product_line

# Step 3: Filter by L4 (Home Lending) - Expected: 22 rows (SVC001=9, SVC002=8, SVC003=5)
| where l4_product_line="Home Lending"

# Step 4: Calculate health scores per signal
| eval health_score=case(
    status=="Green", 100,
    status=="Amber", 50,
    status=="Red", 0,
    1=1, null())

# Step 5: Aggregate by service - Expected: 3 rows (SVC001, SVC002, SVC003)
| stats
    avg(health_score) as service_health
    count as signal_count
    by l3_product, l3_description, service_id

# Continue with rest of query...
```

Add `| table *` after each step to inspect intermediate results.

## Still Not Working?

If you've verified all the above and still get no results:

1. **Export a test query result** from Splunk Search to verify data loads
2. **Check Grafana datasource configuration** - does it connect to Splunk?
3. **Test with L4 query first** - simpler query with no variables
4. **Share the exact error message** or what you see in Grafana query inspector

## Expected Final Results for L3 (Home Lending)

When working correctly with `l4_product_line="Home Lending"`:

```
l3_product          business_purpose                    service_count  coverage  health   health_status
Home Equity         (description)                       (count)        0.0       null     Unknown
Home Originations   Automate loan application...        2              100.0     (calc)   (status)
Home Servicing      (description)                       1              100.0     (calc)   (status)
```

Only Home Originations and Home Servicing will have non-null health values because they have signals.
