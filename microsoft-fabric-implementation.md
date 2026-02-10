# Microsoft Fabric Implementation
## Enterprise Data Platform Migration at Grad Careers

**Project Duration:** Nov 2024 – Present  
**Role:** IT Data Analyst  
**Scope of Responsibility:** End-to-end ownership including architecture design, data engineering, data quality framework, Power BI semantic modelling, stakeholder workshops, and production support  
**Organization:** Grad Careers, Adelaide SA

---

## Executive Summary

**Challenge:** Session coordinators were making staffing decisions based on corrupted booking data from 2-day-old Excel reports, leading to 15-20 disputed weekly exception reports and 65% staffing forecast accuracy.

**Solution:** Delivered a production-grade Microsoft Fabric data platform within 30 days, integrating 3 disparate systems with governed data quality framework and real-time Power BI dashboards.

**Impact:**
- ✅ Improved staffing forecast accuracy from 65% to 85-90%
- ✅ Reduced disputed reports from 15-20 to 3-5 per cycle (75% reduction)
- ✅ Fixed 4,000+ timestamp defects across 200,000+ session records
- ✅ Enabled real-time operational decisions (replacing 2-day-old data)
- ✅ Delivered production platform within 30 days

**Technologies:** Microsoft Fabric (Lakehouse/Warehouse), SQL Server, Python, Power BI, Power Automate

---

## Business Context

**Organization:** Grad Careers  
**Users:** 25+ session coordinators, finance team, operations managers  
**Scale:** 200,000+ annual session bookings

**The Problem:**

Session coordinators responsible for capacity planning were working with:
- Excel reports exported 2 days after session closures
- Corrupted timestamp data (missing check-ins, duplicate sessions, timezone errors)
- Conflicting data between booking system, CRM, and finance
- 18 months of unresolved disputes between finance and operations

**Business Impact:**
- Staffing decisions based on 65% accurate demand forecasts
- 15-20 weekly exception reports requiring manual investigation
- Inability to trust session data for operational planning
- Cross-departmental friction over reporting methodology

---

## Technical Architecture

**Platform:** Microsoft Fabric (Lakehouse + Warehouse)

**Data Sources:**
1. Booking Platform - Session timestamps, attendee data, coordinator assignments
2. Salesforce CRM - Client information, service agreements, program details
3. Finance Database - Billing records, payment status, revenue attribution

**Architecture:**

```
Data Sources (3 Systems)
    ↓ ELT Pipelines
Microsoft Fabric Lakehouse (Bronze/Silver)
    ↓ Transformation Logic
Fabric Warehouse (Gold Layer)
    ↓ Power BI Semantic Model
5 Interactive Dashboards
```

**Key Technical Decisions:**

**Why Microsoft Fabric?**
- Unified platform reducing licensing complexity
- Native Power BI integration for real-time dashboards
- OneLake eliminating data movement between layers
- Lakehouse flexibility with Warehouse performance

**ELT vs ETL:**
- ELT chosen to preserve raw data for audit trails
- Transformations documented as SQL views enabling transparency
- Finance and operations teams could verify transformation logic

---

## Data Quality Framework

**The Challenge:** 4,000+ timestamp defects discovered:
- Missing check-in timestamps
- Duplicate session records
- Timezone inconsistencies
- Late session closures affecting revenue attribution

**Solution: 3-Tier Validation Framework**

**Tier 1 - Structural Validation:**
- Session ID uniqueness checks
- Required field presence validation
- Data type validation
- Referential integrity checks

**Tier 2 - Business Rule Validation:**
- Check-in time within 2 hours of session start
- Session duration between 15 minutes and 8 hours
- Session close date within 48 hours of completion
- No duplicate session IDs per date/coordinator

**Tier 3 - Statistical Anomaly Detection:**
- Fuzzy matching for duplicate detection
- Outlier detection for session counts
- Pattern analysis for coordinator workload

**Remediation:**
- 70% automatically fixed (timezone, duplicates)
- 25% flagged for coordinator review
- 5% escalated to IT for upstream fixes

**Result:** Reduced disputed weekly reports from 15-20 to 3-5

---

## Stakeholder Collaboration

**Challenge:** 18-month dispute between finance and operations about attribution rules

**Solution: Structured Requirements Gathering**

**6 Working Sessions:**

**Sessions 1-2:** Current state mapping
- Documented existing manual processes
- Identified pain points and discrepancies

**Sessions 3-4:** Rule definition
- Defined 18 attribution rules covering:
  - Late closures (>48 hours after delivery)
  - No-shows (marked attended but no check-in)
  - Partial attendance
  - Cancellations

**Session 5:** SQL logic translation
- Translated business rules into SQL transformation logic
- Reviewed T-SQL views with stakeholders
- Validated sample outputs

**Session 6:** UAT and sign-off
- Tested edge cases and historical disputes
- Achieved consensus on final logic
- Documented rules in shared wiki

**Impact:** Ended 18-month dispute through transparent, governed transformation logic

---

## Power BI Dashboard Design

**Audience:** 25+ session coordinators (non-technical operational users)

**Design Principles:**
- Exception-based navigation (show what needs attention)
- Drill-through from summary to detail
- Sub-5-second dashboard refresh
- Mobile-friendly for tablet access during sessions

**5 Dashboard Suite:**

1. **Session Demand Dashboard**
   - Peak demand windows by day/time
   - Capacity utilization percentage
   - 15-minute aggregation bins

2. **Late Closure Tracker**
   - Sessions closed >48 hours after delivery
   - Impact on financial reporting
   - Coordinator-level late closure rates

3. **Exception Management**
   - Real-time exception queue
   - Drill-through to session details
   - Exception aging analysis

4. **Capacity Planning**
   - 4-week rolling forecast
   - Historical demand patterns
   - Staffing recommendations

5. **Finance Reconciliation**
   - Session count by attribution month
   - Revenue attribution breakdown
   - Variance explanations

**Technical Implementation:**
- 40+ DAX measures for calculations
- Drill-through navigation from summary to transaction detail
- Incremental refresh (last 90 days full grain)
- Optimized relationships (star schema)

---

## Power Automate Integration

**Automation Implemented:**

**Exception Alerting:**
- Trigger: Validation failure detected
- Action: Email coordinator within 15 minutes
- Escalation: Notify manager if unresolved in 24 hours

**SLA Breach Notifications:**
- Trigger: Session open >48 hours after delivery
- Action: Automated reminder to coordinator
- Escalation: Operations manager notification after 72 hours

**Daily Refresh Monitoring:**
- Trigger: Warehouse refresh completion
- Action: Success/failure notification to data team
- Dependency: Power BI dataset refresh triggered automatically

**Impact:** Removed 6+ hours weekly manual coordinator follow-up

---

## Upstream System Improvements

**Discovery Through Analysis:**

Exception pattern analysis identified upstream booking system bug:
- Issue: 2,000+ monthly sessions with missing/invalid coordinator IDs
- Root cause: UI validation only, no database constraint

**Collaboration with IT Development:**
1. Provided 6-month trend analysis as evidence
2. Identified specific session types affected
3. Proposed database constraint + UI enhancement
4. Validated fix in test environment

**Result:** Reduced future defect rate by 75%

---

## Rapid Production Platform Delivery: 30-Day Timeline

**Week 1: Foundation**
- Microsoft Learn modules and Fabric trial setup
- Proof-of-concept with sample booking data
- Architecture design and planning

**Week 2: Architecture Design**
- Designed bronze/silver/gold layers
- Planned ELT pipeline structure
- Defined security model and data retention policies

**Week 3: Development**
- Built ELT pipelines (Fabric pipelines for orchestration)
- Developed T-SQL transformation views
- Created Power BI semantic model
- Initial dashboard prototypes

**Week 4: Testing & Deployment**
- UAT with 5 coordinators
- Performance testing and query optimization
- Data validation against legacy Excel reports
- Production deployment

**Key Success Factors:**
- Prior SQL/Azure experience transferred well
- Stakeholder patience during implementation
- Iterative approach (MVP → enhancements)

---

## Measurable Business Impact

**Core Metrics** (see Executive Summary for full quantified impact):

**Operational Efficiency:**
- Coordinators now make staffing decisions with real-time data instead of waiting 2 days
- Automated exception handling eliminated 6+ hours of weekly manual follow-up
- Self-service analytics adoption increased from 30% to 65-70% of coordinators

**Data Quality & Governance:**
- Systematic defect remediation across 200,000+ session records
- Attribution rules documented and version-controlled (ending 18-month dispute)
- Upstream system bug fix preventing 75% of future defects

**Strategic Foundation:**
- Modern data platform enabling future capabilities (forecasting, ML, advanced analytics)
- Scalable architecture supporting organizational growth
- Transformation logic auditable and maintainable

---

## Technical Challenges & Solutions

**Challenge 1: Performance with 200K+ records**
- Solution: 15-minute aggregation bins, incremental refresh, query folding
- Result: Sub-5-second dashboard refresh

**Challenge 2: Data quality at scale**
- Solution: Tiered validation (structural → business → statistical)
- Result: 75% reduction in disputed reports

**Challenge 3: Stakeholder trust after 18-month dispute**
- Solution: 6 collaborative sessions, transparent SQL logic, historical validation
- Result: Consensus achieved, trust established

**Challenge 4: Delivering new platform under deadline**
- Solution: Structured 4-week plan, MVP approach, leveraged existing skills
- Result: Production deployment on schedule

---

## Key Takeaways

**Technical:**
- Start with data quality before building dashboards (fix foundational issues first)
- Incremental approach works better than big-bang delivery
- Performance testing early prevents late-stage surprises
- Version control for SQL transformation logic enables confidence and auditing

**Stakeholder:**
- Collaborative rule definition (6 sessions) was time well spent
- Transparency builds trust (show SQL logic, not just results)
- Train power users who can help onboard others
- Celebrate wins when disputes are resolved

**Platform:**
- Microsoft Fabric learning curve manageable with prior Azure experience
- Unified platform benefits reduced context switching
- Start small with POC, scale up gradually to reduce risk

**What I'd Improve Next Time:**
- Engage finance earlier in data quality rule definition to shorten dispute resolution cycle
- Formalize data contracts with source systems earlier to prevent upstream defects
- Implement more comprehensive data profiling in Week 1 to surface quality issues faster

---

**Project Status:** Production (ongoing support and enhancement)  
**Stakeholder Satisfaction:** High (ended 18-month dispute, achieved consensus)  
**Technical Complexity:** High (new platform, multi-source integration, data quality at scale)  
**Business Impact:** Very High (operational efficiency, data trust, strategic foundation)
