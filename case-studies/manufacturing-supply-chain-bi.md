# Manufacturing Supply Chain BI Suite
## Operational Analytics Platform at Hyundai Mobis

**Project Duration:** Jul 2019 – Jul 2022  
**Role:** BI & Data Platform Analyst  
**Scope of Responsibility:** Designed star schema, built Azure Data Factory incremental pipelines and SQL optimization, developed 8 Power BI dashboards, implemented Power Apps/Automate escalation workflow, and led KPI workshops and adoption training  
**Delivery Model:** Initial MVP delivered in 10 weeks; expanded and supported iteratively across 3 years (new dashboards, KPI refinements, performance tuning, adoption training)  
**Organization:** Hyundai Mobis Technical Centre of India

---

## Executive Summary

**Challenge:** 40+ procurement and warehouse analysts across 3 departments manually reconciling data, with 2-day average resolution time for supplier delivery issues and 8+ hours weekly manual reconciliation per department.

**Solution:** Built Azure SQL data warehouse with Azure Data Factory pipelines, deployed 8 Power BI dashboards with drill-through navigation, and integrated Power Platform solution (Apps, Automate, SharePoint).

**Impact:**
- ✅ Improved supplier on-time delivery from 72% to 89% over 6 months
- ✅ Reduced issue resolution time from 2 days to 4 hours (87% faster)
- ✅ Eliminated 8+ hours weekly manual reconciliation per department (procurement, warehouse, quality)
- ✅ Reduced data refresh from 90 minutes to 25 minutes (72% improvement)
- ✅ Maintained 98.5% platform uptime serving 40+ users across 3 departments
- ✅ Reduced production incidents from 18/year to 6/year through earlier part shortage detection and escalation workflow

**Technologies:** Power BI, Azure SQL Database, Azure Data Factory, Power Apps, Power Automate, SharePoint

**How We Achieved This:**
- Introduced OTD exception queue refreshed every 15 minutes with automated escalation workflow for faster issue detection
- Created supplier and part criticality scoring to prioritize resolution efforts on high-impact items
- Added lead-time variance tracking to prevent production line shortages before they occurred

---

## Business Context

**Organization:** Hyundai Mobis Technical Centre of India  
**Industry:** Automotive manufacturing (Tier 1 supplier)  
**Users:** 40+ procurement analysts, warehouse supervisors, quality engineers  
**Scale:** 200+ suppliers, 10,000+ POs annually, 50,000+ parts

**The Manufacturing Environment:**

Hyundai Mobis supplies components to Hyundai, Kia, and other OEMs. Operations include:
- Procurement: Managing 200+ suppliers for 50,000+ part numbers
- Warehouse: Receiving, inspection, inventory management
- Quality Engineering: Supplier performance, defect tracking, corrective actions

**The Problem:**

**Data Fragmentation:**
- 10+ SAP ERP tables across modules (purchasing, inventory, quality, finance)
- No integrated view of supplier performance
- Analysts manually joining data in Excel (error-prone, time-consuming)

**Operational Inefficiencies:**
- 2-day average to identify and resolve supplier delivery issues
- Late deliveries discovered only when production line ran short
- Warehouse and procurement teams emailing back-and-forth to reconcile data
- No standardized escalation process

**Performance Visibility:**
- Supplier OTD calculated monthly (too late to act)
- No real-time visibility into in-transit shipments
- Quality defects not linked to delivery performance

**Business Impact:**
- Production line stoppages
- Expedited shipping costs
- Supplier relationship strain
- Customer SLA challenges

---

## Technical Architecture

**Platform:** Azure Cloud (SQL Database with columnstore indexes, Data Factory, Power BI Service)

**Source System:** SAP ERP
- MM (Materials Management) - POs, goods receipts
- PP (Production Planning) - Production schedules, consumption
- QM (Quality Management) - Inspection results, defect tracking
- FI (Finance) - Invoice verification, payment status

**Architecture:**

```
SAP ERP (10+ Tables)
    ↓ Azure Data Factory (Incremental Pipelines)
Azure SQL Database (Star Schema with columnstore indexes)
    ↓ Dimensional Model
Power BI Service (8 Dashboards, workspace governance, drill-through, incremental refresh)
    ↓
40+ Users (Procurement, Warehouse, Quality)

Parallel Flow:
SharePoint + Power Apps + Power Automate (Escalation Workflow)
```

**Data Warehouse Design (Star Schema):**

**Fact Tables:**
- Fact_PurchaseOrders (PO line item grain)
- Fact_GoodsReceipts (Receipt transaction grain)
- Fact_QualityInspections (Inspection event grain)
- Fact_SupplierPayments (Payment transaction grain)

**Dimension Tables:**
- Dim_Supplier (200+ suppliers with ratings, payment terms)
- Dim_Part (50,000+ parts with category, criticality, lead time)
- Dim_Date (with fiscal periods, production calendars)
- Dim_Warehouse, Dim_ProcurementAnalyst

---

## Azure Data Factory ETL Pipelines

**Challenge:** 90-minute nightly data refresh impacting morning reporting

**Optimization Strategies:**

**Parallel Processing:**
Before: Sequential extraction (one table at a time) = 65 minutes
After: Parallel extraction (4 simultaneous pipelines) = 20 minutes

**Incremental Loading:**
Only extract changed records since last load
Reduced from 100,000+ rows to 2,000-5,000 daily

**Index Optimization:**
Created columnstore and nonclustered indexes
Query performance: 30 seconds → 3 seconds

**Partitioning:**
Partition fact tables by month
Archive partitions >2 years to blob storage

**Final Result:** 90 minutes → 25 minutes (72% improvement)

---

## Power BI Dashboard Suite

**Design Philosophy:** Exception-based navigation from summary to transaction detail

**Key Dashboards:**

### 1. Supplier On-Time Delivery (OTD) Dashboard

**Audience:** Procurement analysts, procurement manager

**Key Visuals:**
- OTD trend by supplier (rolling 12 months)
- Top offenders with OTD <70%
- Risk heat map (supplier criticality × OTD performance)
- Delivery variance distribution

**Critical Feature: Drill-Through**
- Summary view: Supplier OTD 68%
- Right-click → Drill Through
- Transaction detail: All POs for that supplier
  - Part number, quantity, dates (planned vs actual), status
  - Shipment tracking details
  - Quality inspection results

**Business Value:** Identify problem suppliers in <1 minute (vs 2 hours manual)

### 2. Warehouse Goods Receipt Dashboard

**Audience:** Warehouse supervisors

**Key Visuals:**
- Daily receipt volume (planned vs actual)
- Inspection queue with aging analysis
- Storage utilization by warehouse zone
- Quantity/quality discrepancy tracking

**Real-Time Updates:**
- Dashboard refreshes every 15 minutes during shift hours
- Visual alerts for receipts delayed >3 hours

### 3. Part Availability Dashboard

**Audience:** Production planners, procurement

**Key Visuals:**
- Stock status (on-hand, on-order, in-transit) by part criticality
- Reorder point alerts
- Lead time analysis (actual vs standard)
- Consumption forecast

**Power Automate Integration:**
- Stock level < reorder point → Email procurement analyst
- Critical part delayed >2 days → Escalate to manager

**Business Value:** Prevented production line stoppages, reduced expedited shipping

---

## Power Platform Integration

**Challenge:** No standardized process for escalating supplier delivery issues

**Solution:** SharePoint-based escalation tracking with Power Apps + Power Automate

**Architecture:**

```
Power BI Exception (e.g., Supplier OTD <70%)
    ↓
Power Apps Form (Auto-populated)
    ↓
SharePoint List (Escalation records)
    ↓
Power Automate (Notification workflow)
    ↓
Procurement Analyst (Resolution)
```

**Power Apps Implementation:**

Form with auto-populated fields from Power BI context:
- Supplier Name, Part Number, Current OTD, Affected POs [READ-ONLY]
- Issue Category, Severity, Description, Attachments [USER INPUT]

Power Fx validation logic:
- Description minimum 50 characters
- Critical escalations require attachments
- Auto-assign based on supplier
- Auto-calculate due date based on severity

**SharePoint List Schema:**
- EscalationID, SupplierName, PartNumber
- IssueCategory, Severity, Description
- CreatedBy, AssignedTo, Status
- ResolutionDate, ResolutionNotes
- Aging (calculated field)

**Power Automate Workflows:**

**New Escalation Created:**
- Send email to assigned analyst (+ manager if Critical/High)
- Create task in Planner
- Set due date based on severity

**Escalation Aging Alert:**
- Daily check for open items >5 days
- Send reminders (CC manager if >7 days)

**Weekly Summary Report:**
- Aggregate metrics from past 7 days
- Generate report and email to leadership

---

## Deployment Workflow

**Challenge:** Manual dashboard updates causing errors

**Solution:** Version-controlled deployment workflow using Azure DevOps

**Workflow:**
```
Development Workspace
    ↓ Git commit
Azure DevOps Repository
    ↓ Automated workflow
Test Workspace (UAT)
    ↓ Manual approval
Production Workspace
```

**Version Control Benefits:**
- Rollback capability
- Change tracking
- Regression testing
- Collaborative development

**Result:** Deployment errors reduced from 3-4/month to <1/quarter

---

## Stakeholder Collaboration

**Challenge:** Procurement, warehouse, and quality teams had different KPI definitions

**Discovery (Weeks 1-2):**
- Shadowed analysts in each department
- Documented manual processes and pain points
- Collected existing Excel reports

**Workshop #1 - KPI Alignment (Week 3):**
Brought 12 stakeholders together

**Conflict discovered:** "On-Time Delivery" defined 3 different ways:
- Procurement: "Date goods received at warehouse"
- Warehouse: "Date goods passed quality inspection"
- Quality: "Date goods entered into inventory system"

**Resolution:** Agreed on single definition + tracked all 3 milestones
- OTD = Goods received by requested date (procurement definition)
- Additional metrics: Inspection OTD, Inventory OTD

**Workshop #2 - Dashboard Design (Week 4):**
- Sketched layouts on whiteboard
- Validated visual choices
- Prioritized features (MVP vs nice-to-have)

**Iterative Development (Weeks 5-8):**
- 2-week sprints with demo sessions
- Incorporated feedback iteratively

**UAT (Week 9):**
- 5 power users tested with real data
- 23 feedback items logged and prioritized

**Training & Go-Live (Week 10):**
- 4 training sessions
- User guides and video tutorials
- 95% adoption rate (38 of 40 users) within 3 months

---

## Measurable Business Impact

**Supplier Performance:**
- On-Time Delivery: 72% → 89% (6-month improvement)
- Top 10 suppliers identified and improvement plans implemented

**Operational Efficiency:**
- Issue resolution time: 2 days → 4 hours (87% faster)
- Manual reconciliation: ~8 hours/week per team → eliminated

**Technical Performance:**
- Data refresh time: 90 minutes → 25 minutes (72% improvement)
- Platform uptime: 98.5% over 3 years
- Query performance: 30 seconds → 3 seconds (90% improvement)

**Cost Impact:**
- Expedited shipping: Reduced significantly
- Production incidents: 18/year → 6/year
- Analyst productivity: Manual tasks automated, focus shifted to analysis

**Strategic:**
- Data-driven supplier evaluation replacing subjective assessments
- Scalable platform supporting growth (200 → 250+ suppliers)
- Foundation for advanced analytics

---

## Technologies Used

**Cloud Platform:**
- Azure SQL Database (data warehouse, star schema)
- Azure Data Factory (ETL pipelines, incremental loads)
- Azure Blob Storage (archival)

**Business Intelligence:**
- Power BI Service (enterprise workspace, 8 dashboards)
- Power Query (data transformation)
- DAX (calculated measures, time intelligence)

**Power Platform:**
- Power Apps (escalation tracking canvas app)
- Power Automate (workflows for alerts and reporting)
- SharePoint (escalation list, document library)

**DevOps:**
- Azure DevOps (repository, version control)
- Deployment workflow with testing

**Skills Demonstrated:**
- Manufacturing/supply chain domain expertise
- Azure cloud data warehousing
- ETL development and optimization
- Power BI dimensional modeling
- Power Platform integration
- Version-controlled deployment practices
- Stakeholder collaboration
- Production support (98.5% uptime)

---

## Key Takeaways

**Technical:**
- Incremental loading essential for performance
- Indexing strategy critical (90% query boost)
- Parallel processing reduced pipeline runtime dramatically
- Version control for BI prevented deployment errors

**Process:**
- KPI alignment first (different definitions caused confusion)
- Iterative development (2-week sprints) kept stakeholders engaged
- Power user champions helped drive adoption
- Training investment paid off in adoption rates

**Stakeholder:**
- Shadowing revealed insights we'd have missed
- Visual mockups (whiteboard) prevented rework
- Comprehensive training drove 95% adoption
- Monthly feedback clinics sustained engagement

---

**Project Status:** Production (handed over to local team)  
**User Feedback:** "This transformed how we manage suppliers. We can't imagine working without it now."  
**Technical Complexity:** High (multi-source integration, performance optimization, Power Platform suite)  
**Business Impact:** Very High (cost savings, efficiency gains, strategic supplier management)
