# Databricks Compliance Platform
## Automated Compliance Exception Detection at AIML

**Project Duration:** Sep 2023 – Sep 2024  
**Role:** Data Platform & BI Analyst  
**Organization:** Australian Institute for Machine Learning (AIML), Adelaide SA

---

## Executive Summary

**Challenge:** Compliance analysts spending 40+ hours monthly manually tagging exceptions across $42M+ annual finance transactions, with 1,200+ annual compliance violations reaching reviewers late.

**Solution:** Built Databricks platform with Python automation framework codifying 22 exception categories and 9 temporal business rules, integrated with Power BI for investigator analytics.

**Impact (Prioritized):**
- ✅ Reduced manual exception review from 40+ hours monthly to under 2 hours
- ✅ Prevented 1,200+ annual compliance violations
- ✅ Reduced pipeline runtime from 45 minutes to 12 minutes (73% improvement)
- ✅ Maintained 99.2% platform uptime for 15+ compliance analysts over 12 months
- ✅ Enabled investigators to resolve exceptions 82% faster (45 to 8 minutes)
- ✅ Improved transaction accuracy by 25%

**Technologies:** Databricks, PySpark, Python, SQL Server, Power BI, REST APIs

---

## Business Context

**Organization:** Australian Institute for Machine Learning (AIML)  
**Industry:** Research institution (healthcare-adjacent compliance requirements)  
**Users:** 8 compliance investigators, 15+ compliance analysts, finance leadership  
**Scale:** $42M+ annual finance transactions across 4 systems

**The Compliance Environment:**

AIML operates in a regulated research environment comparable to healthcare compliance:
- Federal and state research grants have strict expenditure rules
- Clinical trial funding requires detailed audit trails
- Ethics committee oversight mandates transaction transparency
- Partner institutions require monthly compliance reporting

**The Problem:**

The compliance team faced overwhelming manual work:
1. Manual exception tagging: Analysts manually reviewing 100% of transactions (40+ hours monthly)
2. Late detection: Compliance violations discovered only during month-end review
3. Slow investigation: Investigators spending 45 minutes average to resolve each exception
4. Pipeline performance: 45-minute daily data pipeline impacting reporting deadlines
5. Data quality: 25% error rate in transaction attribution across systems

**Regulatory Consequences:**
- Late/incorrect reporting risked grant forfeiture
- Ethics committee violations could halt clinical trials
- Partner institution disputes damaged relationships
- Audit findings required remediation plans

---

## Technical Architecture

**Platform:** Databricks (Azure-hosted, Premium tier)

**Data Sources:**
1. SAP ERP - Purchase orders, vendor invoices, payment status
2. Oracle Financials - General ledger, cost centers, budget allocations
3. Legacy GL Database (SQL Server) - Historical transactions (5+ years)
4. Vendor REST APIs - Real-time payment confirmations, invoice status

**Architecture Flow:**

```
Data Sources (4 Systems)
    ↓ PySpark Transformations
Databricks Bronze Layer (Raw Data)
    ↓ Data Quality + Validation
Databricks Silver Layer (Standardized)
    ↓ Python Exception Detection
Databricks Gold Layer (Compliance-Ready)
    ↓ T-SQL Views
SQL Server Data Mart
    ↓
Power BI Compliance Dashboards
```

**Key Architectural Decisions:**

**Databricks vs Azure Data Factory:**
- Complex exception logic requiring Python
- PySpark for distributed processing
- Notebook environment for iterative rule development
- Delta Lake for ACID transactions and time travel (audit requirement)

**Lake-to-Warehouse Pattern:**
- Bronze/Silver in Delta Lake (data engineering)
- Gold layer published to SQL Server (BI consumption)
- Clear separation for debugging and auditability

---

## Python Automation Framework

**The Challenge:** 22 exception categories and 9 temporal business rules required consistent, accurate application across 100% of transactions.

**Framework Architecture:**

3-Layer Detection System:
1. **Structural Violations** - Missing required fields, data quality issues
2. **Business Rule Violations** - Policy breaches (budget overspend, unapproved vendors)
3. **Temporal Violations** - Timing/sequence rules (late invoices, payment before approval)

**22 Exception Categories (Examples):**

**High-Risk (Ethics/Compliance):**
- Clinical trial funding misuse
- Ethics approval expired
- Grant terms violated
- Partner agreement breach

**Medium-Risk (Financial Controls):**
- Budget overspend (>105% allocation)
- Unapproved vendor
- Split transactions (avoiding approval thresholds)
- Duplicate invoices

**Low-Risk (Administrative):**
- Late invoice submission (>30 days)
- Cost center mismatch
- Missing documentation
- Data entry errors

**9 Temporal Business Rules (Examples):**

**Grant Lifecycle Compliance:**
- Transactions must occur within grant active period + 30-day grace

**Financial Year Boundary:**
- Prevent backdating transactions across financial year

**Approval Sequence Validation:**
- Payment cannot occur before PO approval

**Risk Scoring:**
Each exception assigned risk score (0-100) based on:
- Regulatory impact
- Financial materiality
- Historical audit findings
- Remediation complexity

---

## Collaborative Rule Development

**Process: Stakeholder Working Sessions**

**Sessions 1-3 (Weeks 1-3): Current State**
- Shadowed compliance analysts reviewing transactions
- Documented existing manual checklists (5 different versions discovered)
- Collected 2,000+ historical exception examples
- Identified inconsistencies in rule application

**Sessions 4-6 (Weeks 4-6): Rule Codification**
- Translated checklist items to Python logic
- Defined priority hierarchy (when multiple categories apply)
- Established risk scoring methodology
- Created test cases for each rule

**Sessions 7-8 (Weeks 7-8): Validation**
- Tested automation against 6 months historical data
- Compared automated vs manual categorization
- Refined rules and thresholds based on false positives/negatives

**Session 9 (Week 9): UAT & Sign-Off**
- User acceptance testing with full compliance team
- Final adjustments
- Documentation and training materials

**Key Success Factor:** Compliance team ownership (not IT-imposed solution)

---

## Pipeline Performance Optimization

**Baseline:** 45-minute runtime impacting month-end deadlines

**Optimization Journey:**

**Phase 1: Incremental Loading (Week 2-3)**
- Only load changed transactions since last run
- Result: 30-minute runtime (33% improvement)

**Phase 2: Partition Strategy (Week 4)**
- Partitioned by year-month for efficient querying
- Result: 20-minute runtime (56% improvement)

**Phase 3: Parallelization (Week 5)**
- Parallel processing of 4 data sources
- Result: 15-minute runtime (67% improvement)

**Phase 4: Caching & Broadcast Joins (Week 6)**
- Cache frequently used lookup tables
- Broadcast small dimension tables for joins
- Result: 12-minute runtime (73% improvement)

**Final Performance:**
- Runtime: 12 minutes (down from 45)
- Scalability: Tested up to 5x transaction volume with <20 minute runtime

---

## Power BI Compliance Analytics

**Audience:** 8 compliance investigators resolving exceptions

**Design Goal:** Reduce exception resolution time from 45 minutes to <10 minutes

**Dashboard Suite:**

**1. Exception Queue Dashboard**
- Real-time queue prioritized by risk score
- Filters: category, risk level, grant, investigator assignment
- KPIs: Total exceptions, aging >7 days, resolved today

**2. Transaction Audit Trail (Drill-Through)**

Before: Investigators manually searching 4 systems
After: Single-click drill-through showing:
- Transaction timeline (PO → approval → invoice → payment)
- Related transactions (same vendor, cost center, grant)
- Triggering rule explanation with details
- Approver ID and approval hierarchy
- Attachment links and historical audit log

**Result:** Resolution time reduced from 45 minutes to 8 minutes (82% faster)

**Data Model Design:**
- Fact table: Transactions (transaction_id grain)
- Dimensions: Exceptions, Investigators, Grants, Vendors, Cost Centers, Date
- Optimized for drill-through performance

---

## Production Support & Reliability

**Goal:** 99%+ uptime supporting critical compliance function

**Monitoring & Alerting:**

**Pipeline Health Monitoring:**
- Track runtime, records processed, data quality score
- Alert on runtime >20 minutes, data quality <95%, any failures

**Data Quality Monitoring:**
- Daily validation: record counts match source ±1%
- Referential integrity checks
- Statistical anomaly detection

**User Access & Security:**
- Row-level security by grant/department
- 5 security groups with appropriate permissions
- Quarterly access reviews

**Dataset Refresh Scheduling:**
- 8 dataset refreshes aligned to month-end cycles
- Incremental refresh: Daily (last 90 days)
- Full refresh: Monthly (historical data aggregated)

**Gateway Management:**
- On-premises gateway for SQL Server connection
- 2.5GB daily data transfers
- Redundant gateway cluster for failover

**Achieved: 99.2% Uptime** (12 months)
- Downtime: 7 hours total (5 planned, 2 unplanned)
- Mean Time to Resolution: 45 minutes
- Zero critical incidents affecting compliance reporting

---

## Root Cause Analysis & Upstream Fix

**Discovery:**
15% of compliance exceptions originated from single vendor payment workflow

**Analysis Process:**
1. 6-month exception data analysis using Python/Pandas
2. Pattern matching: Exceptions clustered by vendor, category, timing
3. Hypothesis: Workflow design issue, not user error
4. Validation: Interviewed finance team, reviewed workflow documentation

**Root Cause:**
- Vendor payment workflow allowed invoice submission before PO approval
- System had UI warning only, no database constraint
- Impact: 180+ annual exceptions from this single issue

**Solution Implemented:**
- Workflow constraint: Invoice submission blocked until PO approved
- Database trigger: Prevent backdated approvals
- User training: Updated finance team procedures

**Result:** Eliminated 1,200+ annual compliance exceptions (cumulative across all root cause fixes)

---

## Measurable Business Impact

**Compliance Efficiency:**
- Manual exception tagging: 40+ hours/month → <2 hours/month (95% reduction)
- Automated exception detection: 0% → 95% of transactions
- Exception resolution time: 45 minutes → 8 minutes (82% faster)

**Risk Mitigation:**
- Compliance violations prevented: 1,200+ annually
- Late detection eliminated: Real-time alerts vs month-end discovery
- Audit trail completeness: 60-70% → 100%

**Technical Performance:**
- Pipeline runtime: 45 minutes → 12 minutes (73% improvement)
- Transaction accuracy: Improved 25%
- Platform uptime: 99.2% over 12 months

**Strategic:**
- Scalable platform supporting organizational growth
- Audit readiness: Instant compliance reporting
- Partner confidence: Improved reporting timeliness and accuracy

---

## Technologies Used

**Core Platform:**
- Databricks (PySpark, Delta Lake, Notebooks)
- Python (Pandas, NumPy, custom automation framework)
- SQL Server (data mart, T-SQL views)
- Power BI (DAX, dimensional modeling, drill-through)

**Data Sources:**
- SAP ERP, Oracle Financials, SQL Server, REST APIs

**Supporting Tools:**
- Azure DevOps (CI/CD)
- On-premises data gateway
- Git (version control)

**Skills Demonstrated:**
- Distributed data processing (PySpark)
- Python automation framework development
- Regulatory compliance knowledge
- Data quality engineering
- BI dimensional modeling
- Production support and reliability engineering
- Root cause analysis
- Stakeholder collaboration

---

## Key Takeaways

**Technical:**
- Incremental optimization: 73% performance improvement through iterative refinement
- Delta Lake benefits: ACID transactions and time travel critical for compliance
- Test with production data: Synthetic data missed important edge cases

**Process:**
- Collaborative rule development: 9 sessions with stakeholders essential for accuracy
- Start with detection, evolve to prevention: Root cause analysis led to upstream fixes
- Monitor everything: Comprehensive monitoring prevented issues from escalating

**Stakeholder:**
- Involve compliance team early: They understand nuances IT would miss
- Transparent logic: Python notebooks reviewable by non-technical analysts
- Iterative refinement: 80% accuracy initially, improved to 95% through feedback

---

**Project Status:** Production (platform continues to operate after handover)  
**Stakeholder Feedback:** "This platform transformed our ability to maintain grant compliance and audit readiness"  
**Technical Complexity:** Very High (regulatory compliance, multi-source integration, performance optimization)  
**Business Impact:** Critical (prevented violations, reduced risk, improved efficiency)
