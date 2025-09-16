# Business Observability System (BOS) Excel Prototype

> Transform complex observability data models into guided, persona-driven Excel interfaces for management demonstration and stakeholder alignment.

[![Excel Version](https://img.shields.io/badge/Excel-2016%2B-green)](https://www.microsoft.com/excel)
[![Python](https://img.shields.io/badge/Python-3.6%2B-blue)](https://www.python.org/)
[![Status](https://img.shields.io/badge/Status-Production%20Ready-brightgreen)](https://github.com/yourusername/bos-excel-prototype)

## 🎯 Project Overview

The BOS Excel Prototype bridges the gap between technical observability concepts and business stakeholder understanding. Instead of overwhelming users with 53+ technical fields, it provides **persona-driven forms** that collect the same data through guided, business-friendly interfaces.

**Key Value Proposition:**
- **For Management**: Professional dashboard demonstrating business impact of service reliability
- **For Product Owners**: Guided forms to define success criteria in business language
- **For Developers**: Technical implementation fields separated from business context
- **For Operations**: Deployment and alerting configuration in dedicated workflows

## 🏗️ Architecture

### Data Model (5 Normalized Tables)
```
Services (10 fields)
├── Master service definitions
└── Product Owner accountability

SLI_Definitions (16 fields)  
├── Service Level Indicator specifications
└── Mixed Product Owner / Developer accountability

SLO_Configurations (9 fields)
├── Performance targets and thresholds  
└── Mixed Product Owner / Operations accountability

Impact_Assessments (12 fields)
├── Business impact scenarios and queries
└── Mixed Product Owner / Developer accountability

Operational_Metadata (13 fields)
├── Deployment and lifecycle data
└── Operations accountability
```

### Field Distribution by Persona
- **Product Owner**: 22 fields (Business context, success definitions)
- **Developer**: 15 fields (Technical implementation, measurement queries)
- **Operations**: 18 fields (Alerting, deployment, lifecycle management)

## 🚀 Quick Start

### Prerequisites
- Python 3.6+ with pandas, openpyxl, numpy
- Excel 2016+ for viewing/editing output files

### One-Command Build
```bash
python3 build_bos_excel_v3.4.py
```
**Output**: `BOS_Dashboard_Prototype_v3.4.xlsx` (23KB)

### What You Get
- **Professional dashboard** with service selection and dynamic data population
- **3 persona-specific entry forms** (22 PO + 15 Dev + 18 Ops fields)  
- **Complete data model view** showing all 53 fields with relationships
- **5 normalized data tables** matching production database structure
- **Sample data** for 2 services (Treasury funding, Credit checks)

## 📋 Current Features

### ✅ Working Features
- **Dynamic service selection** using business-friendly display names
- **Professional styling** suitable for management presentations
- **Error handling** throughout (no #N/A displays to users)
- **Cross-sheet data relationships** via INDEX/MATCH formulas
- **Persona field separation** based on actual accountability
- **Sample data** demonstrating real-world observability scenarios

### 🔄 Architecture Patterns
- **Service-centric design**: All data linked via service_id foreign keys
- **Display name abstraction**: Users see "Treasury Order Funding Service", system uses "SVC001"
- **Dynamic lookup formulas**: `=IFERROR(INDEX/MATCH())` patterns throughout
- **Helper cell pattern**: Convert display names to service_ids for consistent lookups

## 📊 Sample Data

### SVC001 - Treasury Order Funding Service
- **Business Purpose**: Execute wire transfers to complete home purchase transactions
- **Performance Question**: What percentage of wire transfers complete successfully on scheduled closing date?
- **Business Impact**: 850 daily homebuyers affected, $380K average loss per delay
- **SLI**: funding-success-rate, 99.5% target, SQL data source

### SVC002 - Credit Check Service  
- **Business Purpose**: Validate borrower creditworthiness for loan approval decisions
- **Performance Question**: What percentage of credit checks return valid scores within acceptable time?
- **Business Impact**: 2000 daily applicants, regulatory compliance risk (FCRA 30-day requirement)
- **SLI**: credit-check-success-rate, 99.5% target, SQL data source

## 📖 Documentation Structure

### Core Files
- **`build_bos_excel_v3.4.py`** - Complete build script (988 lines, standalone)
- **`BOS_Dashboard_Prototype_v3.4.xlsx`** - Current Excel output (23KB)

### Documentation Files  
- **`BOS_Development_Context.md`** - Project history, decisions, constraints
- **`Current_Architecture_v3.4.md`** - Technical implementation reference
- **`BOS_Excel_Quick_Start.md`** - Immediate session startup guide
- **`Session_Handoff_Process.md`** - Instructions for creating development continuity packages

## 🔄 Development Continuity

### Session Handoff Process
This project supports **iterative development across multiple AI sessions** using a standardized handoff process:

**End of Session:**
```bash
"Claude, create handoff package per Session_Handoff_Process.md"
```

**Creates 5 updated files:**
- Updated build script with version increment
- Refreshed development context with recent decisions
- Updated architecture reference with new patterns  
- Revised quick start guide with current capabilities
- Updated README.md reflecting current state

**Next Session:**
```bash
"Continue BOS Excel development from handoff files"
```

### Benefits
- ✅ **Zero context loss** between development sessions
- ✅ **Immediate productivity** - new sessions start working in minutes
- ✅ **Professional documentation** maintained automatically
- ✅ **Decision preservation** - architectural choices never re-litigated
- ✅ **Consistent quality** across multiple AI development iterations

## 🔧 Development

### Adding New Services
1. Extend data arrays in `build_bos_excel_v3.4.py`
2. Add service_id (SVC###) and corresponding data to all 5 tables
3. Update dropdown ranges if needed
4. Test with new service selection

### Modifying Field Ownership
1. Check persona accountability in documentation
2. Update entry form field lists
3. Update Service_Data_Model persona sections  
4. Maintain color coding (green=PO, blue=Dev, gray=Ops)
5. Update field count headers

### Extending Dashboard
1. Use `IFERROR()` wrappers for all new lookup formulas
2. Maintain helper cell pattern (A1 = service_id)
3. Test with both SVC001 and SVC002
4. Avoid conditional formatting (causes Excel file corruption)

## ⚠️ Critical Constraints

### Excel Limitations
- **No conditional formatting**: Causes file corruption in current implementation
- **Static styling only**: Colors and formatting applied directly, not dynamically
- **Formula complexity**: INDEX/MATCH patterns proven reliable, avoid alternatives

### Testing Requirements
- **Always test both sample services** (SVC001, SVC002) before delivery
- **Verify file opens cleanly** without Excel recovery warnings
- **Check persona field counts** (22 PO / 15 Dev / 18 Ops)
- **Validate professional appearance** for management presentation

### Architecture Rules
1. **Never modify 5-table structure** without updating dependent formulas
2. **Maintain helper cell pattern** (A1 = service_id) in dynamic sheets
3. **Use IFERROR wrappers** for all INDEX/MATCH formulas
4. **Keep persona field ownership accurate** to documentation
5. **Preserve professional styling** for stakeholder credibility

## 📈 Version History

### v3.4 - Stable Professional Edition (Current)
- ✅ Removed problematic conditional formatting  
- ✅ Maintained all visual improvements
- ✅ File opens cleanly without recovery warnings
- ✅ Production-ready for management demo

### v3.1 - Field Ownership Corrections
- 🔧 Corrected persona field assignments based on CSV documentation
- 🔧 Fixed Service_Data_Model #N/A errors
- 🔧 Resolved circular reference issues

### v2.0 - Dynamic Service Selection
- ✨ Service dropdown using display names
- ✨ INDEX/MATCH lookup formulas
- ✨ Cross-sheet data relationships

### v1.0 - Basic Structure
- 🏗️ Initial 5-table normalized design
- 🏗️ Basic entry forms
- 🏗️ Simple dashboard

## 🎨 Visual Design Standards

### Color Palette
- **Product Owner fields**: Light green (#E8F5E8)
- **Developer fields**: Light blue (#E1F4FD)
- **Operations fields**: Light gray (#F2F2F2)  
- **Section headers**: Themed colors (green=#70AD47, blue=#5B9BD5, red=#C5504B)

### Typography
- **Title**: 18pt, white on navy gradient
- **Section headers**: 12pt bold white on colored background, 25px height
- **Field labels**: 10pt bold on light colored background
- **Values**: 10-12pt normal with text wrapping enabled

## 🤝 Contributing

### Development Workflow
1. **Start session** with existing handoff files for immediate context
2. **Make incremental changes** with testing using established patterns
3. **Document decisions** and constraints as development progresses
4. **Create handoff package** using `Session_Handoff_Process.md` before ending session
5. **Update repository** with new handoff files for next contributor

### Session Continuity
- **Copy current build script** to working directory
- **Read development context** for understanding constraints
- **Reference architecture guide** for implementation patterns  
- **End with handoff creation** for seamless next session transition

### Extension Points
- **Entry form dynamic population**: Add service selection to forms
- **Multi-impact assessment support**: Show multiple impact rows per service
- **Data validation enhancement**: Add dropdown validation for enum fields
- **Advanced dashboard features**: Add trend visualization, comparative views

## 📞 Support

### Quick Troubleshooting
- **File won't open**: Check for conditional formatting issues, revert to v3.4 base
- **#N/A errors**: Verify IFERROR wrappers in formulas, check service_id references
- **Missing data**: Confirm sample data arrays in build script, check INDEX ranges

### Success Validation
- [ ] File opens without Excel recovery warnings
- [ ] Service dropdown shows display names  
- [ ] Switching services updates all dynamic fields
- [ ] No #N/A errors visible to users
- [ ] Professional appearance maintained
- [ ] Field counts correct (22/15/18)

## 📄 License

This project is designed for internal use and management demonstration. Adapt the data model and field definitions to match your organization's observability requirements.

---

**Built for**: Executive stakeholder alignment and management demonstration of Business Observability value proposition

**Architecture**: Excel-based prototype with database-equivalent relationships and persona-driven user experience

**Status**: Production-ready for management presentation, extensible foundation for ongoing development
