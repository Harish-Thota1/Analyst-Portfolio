# Technical Code Examples
## Production-Quality Patterns and Best Practices

**Estimated read time:** ~6 minutes skim, ~20 minutes deep dive

---

## How to Read This Document

Each code sample includes:
- **Problem** - What challenge this solves
- **Approach** - Key technical decisions
- **Trade-offs** - What was optimized for
- **Scale** - Dataset size and performance metrics
- **Business Value** - What improved as a result

**Note:** All identifiers are anonymised. Logic mirrors production patterns used in projects detailed in the case studies.

**Links to Projects:**
- [Microsoft Fabric Implementation](../case-studies/microsoft-fabric-implementation.md)
- [Databricks Compliance Platform](../case-studies/databricks-compliance-platform.md)
- [Manufacturing Supply Chain BI](../case-studies/manufacturing-supply-chain-bi.md)

---

## Table of Contents

1. [SQL/T-SQL Examples](#sqlt-sql-patterns)
2. [Python Data Quality](#python-data-quality)
3. [PySpark ETL](#pyspark-etl-patterns)
4. [DAX Measures](#dax-business-intelligence)
5. [Power Fx Validation](#power-fx-validation)
6. [Testing & Observability](#testing--observability)

---

## SQL/T-SQL Patterns

### 1. Incremental Load with Watermark

**Problem:** Full refresh takes 90 minutes daily, impacting reporting availability  
**Solution:** Incremental load using watermark pattern  
**Scale:** 200,000+ records, reduced runtime from 90min to 25min  
**Business Value:** Real-time reporting instead of day-old data  
**Used in:** Microsoft Fabric, Azure Synapse pipelines

```sql
-- Get last successful load timestamp
DECLARE @LastLoadTimestamp DATETIME2;

SELECT @LastLoadTimestamp = MAX(LoadTimestamp)
FROM Control.DataLoadLog
WHERE TableName = 'FactTransactions' 
  AND Status = 'Success';

-- Extract only changed/new records
SELECT 
    TransactionID,
    TransactionDate,
    Amount,
    VendorID,
    ModifiedDate
FROM Source.Transactions
WHERE ModifiedDate > @LastLoadTimestamp
   OR CreatedDate > @LastLoadTimestamp;

-- UPSERT pattern (merge)
MERGE Target.FactTransactions AS Target
USING Staging.Transactions AS Source
ON Target.TransactionID = Source.TransactionID

WHEN MATCHED AND Source.ModifiedDate > Target.ModifiedDate THEN
    UPDATE SET
        Target.Amount = Source.Amount,
        Target.ModifiedDate = Source.ModifiedDate

WHEN NOT MATCHED BY Target THEN
    INSERT (TransactionID, TransactionDate, Amount, VendorID, ModifiedDate)
    VALUES (Source.TransactionID, Source.TransactionDate, Source.Amount, 
            Source.VendorID, Source.ModifiedDate);

-- Update watermark
INSERT INTO Control.DataLoadLog (TableName, LoadTimestamp, RecordsProcessed, Status)
VALUES ('FactTransactions', GETDATE(), @@ROWCOUNT, 'Success');
```

**Key Trade-offs:**
- Requires reliable ModifiedDate column in source
- Initial load still full refresh
- Control table adds minimal overhead

---

### 2. Slowly Changing Dimension (Type 2)

**Problem:** Need to track historical supplier rating changes for point-in-time analysis  
**Solution:** SCD Type 2 with effective dating  
**Scale:** 200+ suppliers, maintains full history  
**Business Value:** Historical trend analysis, audit compliance  
**Used in:** Power BI data models, warehouses

```sql
-- Dimension table with effective dating
CREATE TABLE Dim.Supplier (
    SupplierKey INT IDENTITY(1,1) PRIMARY KEY,
    SupplierID VARCHAR(50) NOT NULL,
    SupplierName VARCHAR(200),
    SupplierRating VARCHAR(20),
    EffectiveDate DATE NOT NULL,
    ExpiryDate DATE NULL,
    IsCurrent BIT DEFAULT 1
);

-- Update procedure
CREATE PROCEDURE usp_UpdateSupplierDimension
    @SupplierID VARCHAR(50),
    @SupplierName VARCHAR(200),
    @SupplierRating VARCHAR(20)
AS
BEGIN
    DECLARE @EffectiveDate DATE = CAST(GETDATE() AS DATE);
    
    -- Check if rating changed
    IF EXISTS (
        SELECT 1 FROM Dim.Supplier 
        WHERE SupplierID = @SupplierID 
          AND IsCurrent = 1
          AND SupplierRating <> @SupplierRating
    )
    BEGIN
        -- Expire old record
        UPDATE Dim.Supplier
        SET ExpiryDate = DATEADD(DAY, -1, @EffectiveDate),
            IsCurrent = 0
        WHERE SupplierID = @SupplierID AND IsCurrent = 1;
        
        -- Insert new record
        INSERT INTO Dim.Supplier 
        (SupplierID, SupplierName, SupplierRating, EffectiveDate, IsCurrent)
        VALUES (@SupplierID, @SupplierName, @SupplierRating, @EffectiveDate, 1);
    END
END;
```

**Key Trade-offs:**
- Increased storage for history
- Queries need to filter on IsCurrent or date range
- Enables accurate historical analysis

---

### 3. Materialized Reporting Table with Indexes

**Problem:** Real-time dashboard queries taking 30+ seconds  
**Solution:** Pre-aggregated reporting table with optimized indexes  
**Scale:** Sub-5-second dashboard refresh with 200K+ records  
**Business Value:** Operational teams get real-time insights  
**Used in:** Power BI DirectQuery mode

```sql
-- Create aggregated reporting table
CREATE TABLE dbo.SessionDemandAnalysis (
    SessionID INT PRIMARY KEY,
    SessionDate DATE,
    CoordinatorID INT,
    SessionHour INT,
    SessionStatus VARCHAR(50),
    AttendeeCount INT,
    SessionDurationMinutes INT
);

-- ETL process populates this table
INSERT INTO dbo.SessionDemandAnalysis
SELECT 
    s.SessionID,
    s.SessionDate,
    s.CoordinatorID,
    DATEPART(HOUR, s.SessionDate) AS SessionHour,
    CASE 
        WHEN s.ActualStartTime IS NULL THEN 'No Show'
        WHEN DATEDIFF(MINUTE, s.PlannedStartTime, s.ActualStartTime) > 15 THEN 'Late Start'
        ELSE 'On Time'
    END AS SessionStatus,
    s.AttendeeCount,
    s.SessionDurationMinutes
FROM dbo.Sessions s;

-- Optimized indexes for common queries
CREATE NONCLUSTERED INDEX IX_Coordinator_Date
ON dbo.SessionDemandAnalysis (CoordinatorID, SessionDate)
INCLUDE (SessionStatus, AttendeeCount);

CREATE NONCLUSTERED INDEX IX_Status_Hour
ON dbo.SessionDemandAnalysis (SessionStatus, SessionHour)
INCLUDE (AttendeeCount);
```

**Key Trade-offs:**
- Additional storage for materialized table
- ETL overhead to keep synchronized
- Dramatic query performance improvement

---

## Python Data Quality

### Data Quality Validation Framework

**Problem:** 4,000+ data quality defects across 200,000+ records  
**Solution:** 3-tier validation framework  
**Scale:** Processes 200K+ records, identifies 95% of defects automatically  
**Business Value:** Reduced disputed reports from 15-20 to 3-5 weekly  
**Used in:** Microsoft Fabric pipelines

```python
import pandas as pd
from pandas.api.types import is_integer_dtype, is_datetime64_any_dtype
from datetime import datetime

class DataQualityValidator:
    """
    Multi-tier validation framework
    Tier 1: Structural (data types, required fields)
    Tier 2: Business rules (logic, ranges)
    Tier 3: Statistical (anomalies, outliers)
    """
    
    def __init__(self, df, rules_config):
        self.df = df
        self.rules = rules_config
        self.issues = []
        
    def tier1_structural_validation(self):
        """Validate data structure and types"""
        
        # Check required columns
        required = self.rules.get('required_columns', [])
        missing = set(required) - set(self.df.columns)
        if missing:
            self.issues.append({
                'tier': 1,
                'severity': 'CRITICAL',
                'issue': 'Missing columns',
                'details': list(missing)
            })
        
        # Check for nulls in required fields
        for col in required:
            if col in self.df.columns:
                null_count = self.df[col].isnull().sum()
                if null_count > 0:
                    self.issues.append({
                        'tier': 1,
                        'severity': 'HIGH',
                        'issue': f'Null values in {col}',
                        'count': null_count,
                        'percentage': (null_count / len(self.df)) * 100
                    })
        
        return self.issues
    
    def tier2_business_rule_validation(self):
        """Validate business logic"""
        
        # Session duration check
        if 'session_duration_minutes' in self.df.columns:
            invalid = self.df[
                (self.df['session_duration_minutes'] < 15) | 
                (self.df['session_duration_minutes'] > 480)
            ]
            if len(invalid) > 0:
                self.issues.append({
                    'tier': 2,
                    'severity': 'MEDIUM',
                    'issue': 'Invalid session duration',
                    'count': len(invalid),
                    'record_ids': invalid['session_id'].tolist()[:10]
                })
        
        # Duplicate detection
        duplicates = self.df[self.df.duplicated(subset=['session_id'], keep=False)]
        if len(duplicates) > 0:
            self.issues.append({
                'tier': 2,
                'severity': 'CRITICAL',
                'issue': 'Duplicate session IDs',
                'count': len(duplicates)
            })
        
        return self.issues
    
    def tier3_statistical_anomaly(self):
        """Detect statistical outliers"""
        
        if 'attendee_count' in self.df.columns:
            mean = self.df['attendee_count'].mean()
            std = self.df['attendee_count'].std()
            
            outliers = self.df[
                (self.df['attendee_count'] > mean + 3*std) | 
                (self.df['attendee_count'] < mean - 3*std)
            ]
            
            if len(outliers) > 0:
                self.issues.append({
                    'tier': 3,
                    'severity': 'LOW',
                    'issue': 'Attendee count outliers',
                    'count': len(outliers),
                    'threshold': f'±3σ from mean ({mean:.1f})'
                })
        
        return self.issues
    
    def run_all_validations(self):
        """Execute all tiers and return summary"""
        self.tier1_structural_validation()
        self.tier2_business_rule_validation()
        self.tier3_statistical_anomaly()
        
        return {
            'timestamp': datetime.now().isoformat(),
            'total_records': len(self.df),
            'issues': self.issues,
            'critical_count': sum(1 for i in self.issues if i['severity'] == 'CRITICAL'),
            'validation_passed': len(self.issues) == 0
        }
```

**Key Trade-offs:**
- Processing overhead for validation
- False positives require manual review
- Catches 95% of issues automatically

---

## PySpark ETL Patterns

### Databricks Delta Lake Pipeline

**Problem:** 45-minute pipeline runtime impacting deadlines  
**Solution:** Optimized PySpark with Delta Lake, incremental loads, caching  
**Scale:** $42M+ transactions, runtime reduced to 12 minutes  
**Business Value:** Timely compliance reporting  
**Used in:** Databricks compliance platform

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import *
from delta.tables import DeltaTable

spark = SparkSession.builder \
    .appName("Compliance ETL") \
    .config("spark.sql.extensions", "io.delta.sql.DeltaSparkSessionExtension") \
    .getOrCreate()

# BRONZE LAYER: Raw ingestion
def load_to_bronze():
    """Ingest raw data with metadata"""
    
    # Read from source
    source_df = spark.read \
        .format("jdbc") \
        .option("url", "jdbc:sqlserver://server:1433") \
        .option("dbtable", "GL_Transactions") \
        .load()
    
    # Add metadata
    bronze_df = source_df \
        .withColumn("source_system", lit("SAP")) \
        .withColumn("ingestion_timestamp", current_timestamp())
    
    # Write to Delta (append for history)
    bronze_df.write \
        .format("delta") \
        .mode("append") \
        .partitionBy("transaction_date") \
        .save("/mnt/bronze/transactions")

# SILVER LAYER: Quality and standardization
def transform_to_silver():
    """Cleanse, validate, standardize"""
    
    bronze_df = spark.read.format("delta").load("/mnt/bronze/transactions")
    
    # Data quality transformations
    silver_df = bronze_df \
        .filter(col("transaction_id").isNotNull()) \
        .filter(col("amount") > 0) \
        .dropDuplicates(["transaction_id"]) \
        .withColumn("transaction_date_cleaned", to_date(col("transaction_date"))) \
        .withColumn("fiscal_year", 
            when(month(col("transaction_date_cleaned")) >= 7, 
                 year(col("transaction_date_cleaned")))
            .otherwise(year(col("transaction_date_cleaned")) - 1)
        )
    
    # Merge to Silver (idempotent)
    if DeltaTable.isDeltaTable(spark, "/mnt/silver/transactions"):
        deltaTable = DeltaTable.forPath(spark, "/mnt/silver/transactions")
        deltaTable.alias("target") \
            .merge(silver_df.alias("source"), 
                   "target.transaction_id = source.transaction_id") \
            .whenMatchedUpdateAll() \
            .whenNotMatchedInsertAll() \
            .execute()
    else:
        silver_df.write.format("delta").mode("overwrite").save("/mnt/silver/transactions")

# GOLD LAYER: Business logic
def transform_to_gold():
    """Apply business rules and enrichments"""
    
    silver_df = spark.read.format("delta").load("/mnt/silver/transactions")
    
    # Cache frequently used dimensions
    vendors_df = spark.read.format("delta").load("/mnt/dimensions/vendors").cache()
    
    # Business logic transformations
    gold_df = silver_df \
        .join(vendors_df, "vendor_id", "left") \
        .select(
            col("transaction_id"),
            col("transaction_date_cleaned").alias("transaction_date"),
            col("amount"),
            col("vendor_name"),
            # Compliance flags
            when(col("approved_vendor_flag") == False, True).otherwise(False).alias("compliance_flag")
        )
    
    gold_df.write.format("delta").mode("overwrite").save("/mnt/gold/fact_transactions")
    
    # Optimize for query performance
    spark.sql("OPTIMIZE delta.`/mnt/gold/fact_transactions` ZORDER BY (vendor_name)")
```

**Key Trade-offs:**
- Delta Lake adds storage overhead
- Optimization (Z-ORDER) takes time but improves queries
- Enables time travel for audits

---

## DAX Business Intelligence

### Time Intelligence Measures

**Problem:** Need consistent YoY, MoM, rolling average calculations  
**Scale:** Supports trend analysis across 200K+ transactions  
**Business Value:** Enable forecasting and performance tracking  
**Used in:** All Power BI dashboards

```dax
-- Year-over-Year comparison
Total Sales YoY % = 
VAR CurrentYear = [Total Sales]
VAR PreviousYear = 
    CALCULATE([Total Sales], DATEADD('Date'[Date], -1, YEAR))
RETURN
    DIVIDE(CurrentYear - PreviousYear, PreviousYear, 0)

-- Rolling 3-month average
Sales 3M Avg = 
AVERAGEX(
    DATESINPERIOD('Date'[Date], MAX('Date'[Date]), -3, MONTH),
    [Total Sales]
)

-- Year-to-Date (fiscal year ending June 30)
Sales YTD = 
TOTALYTD([Total Sales], 'Date'[Date], "6-30")

-- Context-aware drill-through measure
Transactions in Context = 
VAR SelectedVendor = SELECTEDVALUE(Vendors[Vendor ID])
VAR SelectedCategory = SELECTEDVALUE(Exceptions[Category])
RETURN
    CALCULATE(
        COUNTROWS(Transactions),
        Transactions[Vendor ID] = SelectedVendor,
        Transactions[Exception Category] = SelectedCategory
    )
```

**Key Trade-offs:**
- Complex DAX can impact performance
- Requires good data model design
- Enables powerful self-service analytics

---

## Power Fx Validation

### Power Apps Form Validation

**Problem:** Need data quality enforcement before SharePoint submission  
**Scale:** Validates 100+ escalations monthly  
**Business Value:** Ensures actionable escalation data  
**Used in:** Manufacturing escalation tracking

```powerfx
// Submit button validation logic
OnSelect Property of SubmitButton:

If(
    // Validation 1: Description length
    Len(DescriptionInput.Text) < 50,
    Notify("Description must be at least 50 characters", NotificationType.Error),
    
    // Validation 2: Critical severity requires attachment
    SeverityDropdown.Selected.Value = "Critical" && 
    CountRows(AttachmentsGallery.AllItems) = 0,
    Notify("Critical escalations require attachments", NotificationType.Error),
    
    // All validations passed - Submit
    Patch(
        EscalationsList,
        Defaults(EscalationsList),
        {
            SupplierName: SupplierDropdown.Selected.SupplierName,
            Severity: SeverityDropdown.Selected.Value,
            Description: DescriptionInput.Text,
            AssignedTo: LookUp(
                ProcurementAnalysts,
                Supplier_ID = SupplierDropdown.Selected.ID,
                Analyst_Email
            ),
            Status: "Open",
            CreatedDate: Now(),
            DueDate: Switch(
                SeverityDropdown.Selected.Value,
                "Critical", DateAdd(Today(), 2, Days),
                "High", DateAdd(Today(), 5, Days),
                DateAdd(Today(), 10, Days)
            )
        }
    );
    Navigate(ConfirmationScreen)
)
```

**Key Trade-offs:**
- Client-side validation only (not database constraint)
- Improves data quality significantly
- Better user experience than post-submission errors

---

## Testing & Observability

### Production Monitoring Pattern

**Problem:** Need proactive issue detection before users notice  
**Scale:** Monitors 200K+ daily records  
**Business Value:** 99.2% uptime achieved  
**Used in:** All production pipelines

```python
def monitor_pipeline_health():
    """Track and alert on pipeline metrics"""
    
    metrics = {
        'runtime_minutes': (end_time - start_time).total_seconds() / 60,
        'records_processed': record_count,
        'exceptions_detected': exception_count,
        'data_quality_score': (valid_count / total_count) * 100,
        'failures': error_count
    }
    
    # Alert thresholds
    alerts = []
    
    if metrics['runtime_minutes'] > 20:
        alerts.append({
            'severity': 'WARNING',
            'message': f"Runtime exceeded threshold: {metrics['runtime_minutes']:.1f} min"
        })
    
    if metrics['data_quality_score'] < 95:
        alerts.append({
            'severity': 'HIGH',
            'message': f"Data quality below 95%: {metrics['data_quality_score']:.1f}%"
        })
    
    if metrics['failures'] > 0:
        alerts.append({
            'severity': 'CRITICAL',
            'message': f"Pipeline failures detected: {metrics['failures']}"
        })
    
    # Log metrics
    log_to_database(metrics)
    
    # Send alerts if any
    if alerts:
        send_alert_email(alerts)
    
    return metrics

### Row Count Reconciliation
def validate_row_counts():
    """Ensure source and target counts match"""
    
    source_count = get_source_count()
    target_count = get_target_count()
    
    variance_pct = abs(source_count - target_count) / source_count * 100
    
    if variance_pct > 1:  # >1% variance triggers alert
        raise Exception(f"Row count mismatch: {variance_pct:.2f}% variance")
    
    return True
```

**Key Trade-offs:**
- Monitoring overhead (minimal)
- Prevents issues from reaching users
- Enables data-driven SLA management

---

## Summary: What These Examples Prove

**Production Quality:**
- Error handling and logging
- Performance optimization (70-75% improvements achieved)
- Maintainable, documented patterns

**Technical Depth:**
- SQL optimization (incremental loads, indexing, partitioning)
- Python frameworks (data quality, validation)
- Distributed processing (PySpark)
- Business intelligence (DAX, drill-through)
- Low-code solutions (Power Fx)

**Business Value:**
- Every pattern has quantified impact
- Domain-specific solutions (compliance, manufacturing, finance)
- End-to-end pipeline development

All patterns based on real production implementations detailed in case studies.
