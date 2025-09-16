# BOS Excel Current Architecture - v3.4

## Field Mapping by Persona (CORRECTED)

### Product Owner Fields (22 total)
**Services Table (10 fields)**
- service_id, serviceName, displayName, businessPurpose, serviceType
- tierLevel, businessUnit, performanceQuestion, tags, productOwner

**SLI_Definitions Table (4 fields)**
- sliName, sliDisplayName, goodEventsCriteria_PO, totalEventsCriteria_PO

**SLO_Configurations Table (3 fields)**  
- sloTarget, sloTargetRationale, timeWindow

**Impact_Assessments Table (5 fields)**
- impactCategory, stakeholderType, stakeholderCount, failureScenario, businessConsequence, financialImpact, regulatoryImpact

### Developer Fields (15 total)
**SLI_Definitions Table (11 fields)**
- sliType, goodEventsCriteria_Dev, totalEventsCriteria_Dev, thresholdQuery_Dev
- thresholdOperator, thresholdValue, queryImplementation, dataSource
- dataSourceDetails, technicalOwner, implementationNotes

**Impact_Assessments Table (4 fields)**
- customerImpactQuery, financialImpactQuery, legalRiskQuery, operationalImpactQuery

### Operations Fields (18 total)
**Operational_Metadata Table (12 fields)**
- alertNotificationTargets, dashboardUrl, runbookUrl, alertingConfigured
- lastValidated, version, created, modified, modifiedBy, status, reviewDate, notes

**SLO_Configurations Table (6 fields)**
- timeWindowType, budgetingMethod, timeSliceTarget, timeSliceWindow
- alertingThreshold, pageThreshold

## Data Relationships
```
Services (1) ←→ (1) SLI_Definitions
Services (1) ←→ (1) SLO_Configurations  
Services (1) ←→ (1) Operational_Metadata
Services (1) ←→ (1-4) Impact_Assessments
```

## Excel Implementation Patterns

### Service Selection Pattern
```
User sees: "Treasury Order Funding Service" (display name)
System uses: "SVC001" (service_id)
Helper formula: =INDEX(Services!A:A,MATCH(B3,Services!C:C,0))
```

### Dynamic Lookup Pattern
```
=IFERROR(INDEX(TableName!Column:Column,MATCH(service_id,TableName!A:A,0)), "Not Defined")
```

### Color Coding Standards
- **Product Owner fields**: Light green (#E8F5E8)
- **Developer fields**: Light blue (#E1F4FD)  
- **Operations fields**: Light gray (#F2F2F2)
- **Section headers**: Themed colors (green=#70AD47, blue=#5B9BD5, red=#C5504B)

### Stats Box Configuration
```
CURRENT: Dynamic formula showing actual performance
TARGET: Lookup from SLO_Configurations table
STATUS: Conditional logic (OK/WARNING/CRITICAL with icons)
TREND: Static placeholder for future enhancement
```

## Professional Styling Standards
- **Title**: 18pt, white on navy gradient
- **Section headers**: 12pt bold white on colored background, 25px height
- **Field labels**: 10pt bold on light colored background
- **Values**: 10-12pt normal with text wrapping enabled
- **Column widths**: A=18, B=35, C-I=15-25 (responsive to content)

## Sample Data Structure (SVC001)
```csv
# Services
SVC001,treasury-order-funding-service,Treasury Order Funding Service,"Execute wire transfers to complete home purchase transactions",customer-facing,1,Home Lending,"What percentage of wire transfers complete successfully on scheduled closing date?",mortgage,sarah.chen@company.com

# SLI_Definitions  
SVC001,funding-success-rate,Wire Transfer Success Rate,ratioMetric,"Wire transfer completed successfully enabling scheduled closing","status='FUNDED' AND closing_date=scheduled_date AND amount>0","All wire transfer attempts for scheduled closings","request_type='CLOSING_FUNDING' AND scheduled_date IS NOT NULL",,,,SELECT COUNT(*) WHERE status='FUNDED',sql,"{""database"":""loans"",""table"":""wire_transfers""}",platform-engineering-team,"Requires 5-minute aggregation window"

# SLO_Configurations
SVC001,99.5,"Industry standard for critical financial transactions with manageable failure volume for escalation team",7d,rolling,Occurrences,,,99.3,99.0

# Impact_Assessments
SVC001,customer_experience,homebuyers,850 daily,"Wire transfer fails preventing scheduled closing","Homebuyers cannot complete purchase and may lose rate lock",,,COUNT(DISTINCT customer_id) WHERE status='FAILED',,,
SVC001,financial,company,,"Wire transfer failures prevent closings","Lost transaction revenue and potential customer churn",$380000 average per delayed closing,,SUM(wire_amount) WHERE status!='FUNDED',,

# Operational_Metadata
SVC001,email:homelending-ops@company.com;pagerduty:PD-HL-001,https://grafana.company.com/d/funding-slo,https://wiki.company.com/runbooks/funding,true,2024-01-15,1.0,2024-01-01T00:00:00Z,2024-01-15T10:30:00Z,sarah.chen@company.com,active,2024-07-01,"Initial BOS implementation for treasury service"
```

## Extension Points for Future Development

### Adding Entry Form Dynamic Population
1. Modify entry forms to use helper cells like Dashboard
2. Add service selection dropdowns to each form
3. Populate fields with INDEX/MATCH lookups
4. Add "Save Changes" functionality

### Multi-Impact Assessment Support
1. Modify Dashboard to show multiple impact rows
2. Add impact category filtering
3. Enhance Impact_Assessments display logic

### Data Validation Enhancement
1. Add dropdown validation for enum fields
2. Add date format validation
3. Add email format validation for owner fields
4. Add numeric range validation for targets

### Advanced Dashboard Features
1. Add trend visualization (requires historical data)
2. Add service health scoring algorithm  
3. Add comparative dashboards (multiple services)
4. Add export to monitoring system integration

## Critical "Don't Break" Rules
1. **Never modify the 5-table structure** without updating all dependent formulas
2. **Always test with both SVC001 and SVC002** before delivering
3. **Maintain helper cell pattern** (A1 = service_id) in dynamic sheets
4. **Use IFERROR wrappers** for all INDEX/MATCH formulas
5. **Avoid conditional formatting** (causes file corruption in current Excel version)
6. **Keep persona field ownership accurate** to CSV documentation
7. **Test file opens cleanly** before delivery (no recovery warnings)

## Success Metrics for Future Versions
- File opens without Excel warnings
- All dynamic lookups work correctly
- Professional appearance maintained
- Field counts match persona assignments
- Management demo-ready quality
- Extensible for new services/requirements

## Formula Reference Guide

### Working Formula Patterns
```excel
# Basic lookup with error handling
=IFERROR(INDEX(Services!B:B,MATCH(A1,Services!A:A,0)), "Not Defined")

# Service name to ID conversion
=INDEX(Services!A:A,MATCH(B3,Services!C:C,0))

# Conditional display for different services
=IF(A1="SVC001","99.2%",IF(A1="SVC002","97.8%","No Data"))

# Target percentage formatting
=IFERROR(INDEX(SLO_Configurations!B:B,MATCH(A1,SLO_Configurations!A:A,0)) & "%", "No Target")
```

### Cell Reference Patterns
- **A1**: Service_id helper cell (converted from display name)
- **B3**: Service display name (user selection)
- **D3**: Alternative helper cell in some sheets
- **Service ranges**: Services!A:A (ids), Services!C:C (display names)

## Dashboard Layout Specifications
```
Row 1: Title (merged A1:I1, navy gradient)
Row 3: Service selector (A3:B3)
Row 5: SERVICE CONTEXT header (merged A5:I5, green)
Row 7-9: Service details (2-column layout)
Row 12: SLI header (merged A12:I12, blue)  
Row 13: SLI name (merged B13:I13)
Row 15-16: Stats boxes (A,C,E,G columns)
Row 19-21: SLI details (merged B:I)
Row 24: Impact header (merged A24:I24, red)
Row 25-28: Impact details (merged B:I)
Row 30-33: Ownership (2-column A-D, E-I)
```

## Testing Data Sets
**SVC001 Complete Data**: All fields populated, multiple impact assessments
**SVC002 Complete Data**: All fields populated, regulatory compliance focus
**SVC003 Minimal Data**: Test error handling with sparse data
**New Service Template**: Empty structure for adding new services

This architecture has been tested and verified to work consistently across service selections while maintaining professional appearance and data integrity.
