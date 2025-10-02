# Splunk SPL vs SQL Translation Lessons Learned

**Purpose**: Document critical differences between Splunk SPL and SQL discovered during L4 dashboard query development.

**Date**: 2025-10-02

---

## Critical Insight #1: JOIN vs LOOKUP (1-to-1 vs 1-to-many)

### The Problem

**Splunk's `join` command is fundamentally different from SQL's LEFT JOIN:**

- **SQL LEFT JOIN**: 1-to-many relationship (one service → many signals)
- **Splunk join**: **1-to-1 by default** (one service → one signal only)

### Real Example from L4 Dashboard

**SQL Query (correct):**
```sql
FROM services s
LEFT JOIN signal_status ss ON s.service_id = ss.service_id
```
Result: SVC001 with 9 signals returns **9 rows**

**Splunk join (wrong):**
```spl
| inputlookup bos_services.csv
| join type=left service_id [| inputlookup bos_signal_status.csv]
```
Result: SVC001 with 9 signals returns **1 row** (only first match)

### The Solution: Use LOOKUP Instead

**Correct SPL approach:**
```spl
| inputlookup bos_signal_status.csv              # Start with "many" side
| lookup bos_services.csv service_id OUTPUT l4_product_line  # Add fields from "one" side
```
Result: SVC001 with 9 signals returns **9 rows** (all signals preserved)

### Key Principle

**When translating SQL to SPL:**
- ❌ **Don't use**: `join` for 1-to-many relationships
- ✅ **Do use**: `lookup` or start with the "many" side table

---

## Critical Insight #2: Two-Level Averaging

### The Problem

SQL can do nested aggregations in a single query. SPL requires multiple `stats` commands.

### SQL Approach
```sql
SELECT
    l4_product_line,
    AVG((SELECT AVG(CASE status
        WHEN 'Green' THEN 100
        WHEN 'Amber' THEN 50
        ELSE 0 END)
        FROM signal_status WHERE service_id = s.service_id)) as health
FROM services s
GROUP BY l4_product_line
```

**Logic:**
1. Inner query: Average signals per service → 72.2%, 75.0%, 90.0%
2. Outer query: Average service averages → 79.1%

### SPL Approach

**Requires 3 stats commands:**

```spl
# First stats: Per-service average
| stats avg(health_score) as service_health by l4_product_line, service_id

# Handle services with no signals (append + merge)
| append [| inputlookup bos_services.csv | fields service_id l4_product_line]
| stats values(service_health) by l4_product_line, service_id
| eval service_health=mvindex(service_health,0)

# Second stats: Product line average
| stats avg(service_health) as health by l4_product_line
```

### Key Principle

**When translating nested aggregations:**
- Break into sequential `stats` commands
- Use `append` to include rows with no matches
- Use `values()` + `mvindex()` to handle multi-value fields

---

## Critical Insight #3: NULL Handling Differences

### The Problem

SQL and Splunk handle NULL values differently in certain contexts.

### Coverage Calculation Example

**SQL:**
```sql
COUNT(DISTINCT CASE WHEN EXISTS(...) THEN service_id END) / COUNT(DISTINCT service_id)
```
- Returns: 3/6 = 50.0% for Home Lending
- Returns: 0/6 = 0.0% for Auto Lending

**SPL (wrong approach):**
```spl
| stats sum(eval(signal_count > 0)) as services_with_signals by l4_product_line
```
- If `signal_count` is null, `signal_count > 0` is null, sum ignores it
- But we need it counted as 0!

**SPL (correct approach):**
```spl
| eval signal_count=coalesce(mvindex(signal_count,0),0)  # Convert null → 0 FIRST
| stats sum(eval(signal_count > 0)) as services_with_signals by l4_product_line
```

### Key Principle

**Always explicitly handle nulls:**
- Use `coalesce(field, 0)` to convert null → 0
- Use `fillnull value=0` for display fields
- Don't rely on implicit null handling in eval expressions

---

## Critical Insight #4: Multi-Value Field Extraction

### The Problem

After using `stats values()`, fields become multi-value even if only one value exists.

### Example from L4 Query

```spl
| stats values(service_health) as service_health by service_id
```

**Result**: `service_health` is now a multi-value field (even if only one value)

**Problem**: Can't use multi-value field in `avg()` calculation

**Solution**: Extract single value with `mvindex()`

```spl
| eval service_health=mvindex(service_health,0)  # Get first (only) value
```

### Key Principle

**After using `stats values()`:**
- Always extract single values with `mvindex(field,0)`
- Use `coalesce(mvindex(field,0), default)` if null is possible

---

## Critical Insight #5: Display vs Calculation NULL

### The Problem

Some fields should be NULL (no calculation possible), others should be 0 (zero count).

### L4 Dashboard Example

**Product line with no signals:**

| Field | Value | Reason |
|-------|-------|--------|
| services_with_signals | **0** | Zero services have signals (count) |
| coverage | **0.0** | Zero percent coverage (calculated metric) |
| health | **null** | No health calculation possible (no data) |
| health_status | **"Unknown"** | Cannot determine status (no data) |
| incident_count | **0** | Zero incidents (count) |

### SPL Implementation

```spl
# health stays null naturally (avg of nulls = null)
| eval health=if(isnotnull(health), round(health, 1), null())

# health_status explicitly set to "Unknown" for null
| eval health_status=case(
    overall_min_status==3, "Green",
    overall_min_status==2, "Amber",
    overall_min_status==1, "Red",
    1=1, "Unknown")

# Counts explicitly set to 0 for display
| fillnull value=0 incident_count services_with_signals coverage
```

### Key Principle

**Distinguish between:**
- **Counts**: Always show 0 (use `fillnull value=0`)
- **Calculations**: Can be null if no data (use `if(isnotnull(...))`)
- **Status fields**: Provide explicit default (use `case ... 1=1, "Unknown"`)

---

## Complete Working Example: L4 Dashboard Query

**File**: `l4-dashboard-splunk-query.spl`

**Key sections:**

1. **Lines 14-30**: Start with signals, use lookup (not join), calculate per-signal values
2. **Lines 31-41**: Append all services, merge with values(), extract with mvindex()
3. **Lines 42-47**: Calculate product line aggregates
4. **Lines 48-54**: Round, handle nulls, calculate status
5. **Lines 55-59**: Join incidents, fill display nulls
6. **Lines 60-61**: Format output

**Verified Results:**
- Home Lending: health=79.1%, status=Red ✅
- Other 3 product lines: services_with_signals=0, coverage=0.0, health=null, status=Unknown ✅

---

## Checklist for Future SQL → SPL Translations

When converting SQL queries to SPL:

- [ ] **Identify 1-to-many relationships** → Use `lookup` not `join`
- [ ] **Start with "many" side table** → Preserves all rows
- [ ] **Map nested aggregations** → Break into multiple `stats` commands
- [ ] **Handle services with no signals** → Use `append` + `stats values()`
- [ ] **Extract multi-value fields** → Use `mvindex(field,0)`
- [ ] **Convert nulls for calculations** → Use `coalesce(field,0)`
- [ ] **Set display defaults** → Use `fillnull value=0` for counts
- [ ] **Test incrementally** → Verify each stats command separately
- [ ] **Compare results** → Run both SQL and SPL, verify exact match

---

## Testing Strategy

### Diagnostic Query Pattern

Always create a simplified test query first:

```spl
# Test per-service calculations before product line aggregation
| inputlookup bos_signal_status.csv
| lookup bos_services.csv service_id OUTPUT l4_product_line
| where l4_product_line="Home Lending"
| eval health_score=case(status=="Green", 100, status=="Amber", 50, status=="Red", 0, 1=1, null())
| stats avg(health_score) as service_health count as signal_count by service_id
| sort service_id
```

**Expected**: SVC001=72.2% with 9 signals, SVC002=75.0% with 8 signals, etc.

**If wrong**: Fix this before attempting full query

### Incremental Development

1. Get signals with lookups working
2. Add per-service calculations
3. Add services with no signals (append)
4. Add product line aggregation
5. Add display formatting
6. Add incident joins

**Test after each step!**

---

## Common Pitfalls

### Pitfall 1: Using dc(status) Instead of count

```spl
| stats dc(status) as signal_count  # WRONG - counts distinct statuses (max 3)
| stats count as signal_count        # CORRECT - counts all signals
```

### Pitfall 2: Not Handling Multi-Value Fields

```spl
| stats values(service_health) by service_id
| stats avg(service_health)  # WRONG - can't avg multi-value field

| stats values(service_health) by service_id
| eval service_health=mvindex(service_health,0)  # Extract single value first
| stats avg(service_health)  # CORRECT
```

### Pitfall 3: Forgetting to Fill Display Nulls

```spl
| table service_count services_with_signals  # Shows blank for 0 values

| fillnull value=0 services_with_signals     # Shows 0 for null values
| table service_count services_with_signals  # CORRECT
```

---

## Reference Links

- **Main query**: `l4-dashboard-splunk-query.spl` (lines 14-61)
- **Diagnostic query**: `l4-dashboard-splunk-query.spl` (lines 96-110)
- **CSV files**: `services.csv`, `signal_status.csv`, `incidents.csv`
- **SQL comparison**: See bos-artifacts repo `/grafana/grafana-panel-tools/` dashboard generators

---

## Session History

**2025-10-02**: Discovered join vs lookup issue
- Initial query showed wrong results (service_health=50 for SVC001, expected 72.2)
- Root cause: join was only returning 1 signal per service (1-to-1)
- Solution: Replaced join with lookup to get all signals (1-to-many)
- Verified diagnostic query matches SQL results
- Fixed main L4 query using same approach
- Added fillnull for display fields (services_with_signals, coverage)

**Result**: SPL query now produces identical results to SQL query ✅
