# 44-Amazon-QuickSight-Part1

> AWS DevOps Playbook — Volume 1: Data, Analytics & Observability  
> Chapter 44 — Amazon QuickSight Part 1: Fundamentals, Architecture, Data Sources, SPICE, Datasets, Analyses & Dashboards

---

## Chapter Objectives

Learn:

- What Amazon QuickSight is and why it is used
- QuickSight architecture
- Data sources and datasets
- Analyses and dashboards
- Visuals, filters, parameters and calculated fields
- Dimensions and measures
- Direct Query vs SPICE
- Dataset refresh concepts
- Athena + QuickSight architecture
- S3 + Athena + QuickSight architecture
- RLS and security basics
- DevOps, security and cost dashboards
- Troubleshooting and interview questions

---

# 1. What is Amazon QuickSight?

Amazon QuickSight is AWS's cloud-based business intelligence and visualization service.

It allows organizations to:

```text
Connect to data
      |
      v
Prepare / model data
      |
      v
Create visualizations
      |
      v
Build dashboards
      |
      v
Share insights
```

QuickSight can work with AWS services and supported external data sources.

Common sources include:

```text
Amazon Athena
Amazon Redshift
Amazon S3
Amazon RDS
Amazon Aurora
Files such as CSV / Excel
Supported external databases and services
```

---

# 2. Why Do We Need QuickSight?

Raw data is difficult for business and engineering users to understand.

For example:

```text
Application Logs
CloudTrail
WAF Logs
Cost Data
Incident Data
Database Data
```

QuickSight turns this into useful information:

```text
Raw Data
   |
   v
Dataset
   |
   v
Analysis
   |
   v
Charts / KPIs
   |
   v
Dashboard
```

Example:

```text
Total Requests       15.2M
5xx Errors             2.1K
Error Rate             0.014%
P95 Latency            220 ms
Availability           99.98%
```

---

# 3. QuickSight Architecture

Simplified architecture:

```text
                 DATA SOURCES
                      |
        +-------------+-------------+
        |             |             |
       S3           Athena       Redshift
        |             |             |
        +-------------+-------------+
                      |
                      v
                 QUICKSIGHT
                      |
             +--------+--------+
             |                 |
          Dataset           Dataset
             |                 |
             v                 v
          Analysis          Analysis
             |                 |
             +--------+--------+
                      |
                      v
                  Dashboard
                      |
                      v
                    Users
```

---

# 4. Main QuickSight Concepts

Remember:

```text
Data Source
    ↓
Dataset
    ↓
Analysis
    ↓
Dashboard
```

Mental model:

```text
Data Source
= Where data comes from

Dataset
= Data model prepared for analysis

Analysis
= Authoring workspace

Dashboard
= Published/shareable analytical content
```

---

# 5. Data Source

A data source represents the system from which QuickSight obtains data.

Examples:

```text
Athena
Redshift
RDS
S3
CSV
Excel
Supported external sources
```

Common AWS architecture:

```text
QuickSight
    |
    v
Athena
    |
    v
Glue Data Catalog
    |
    v
S3
```

---

# 6. Dataset

A dataset is the data model used by an analysis.

Conceptually:

```text
Data Source
     |
     v
 Dataset
     |
     +---- Fields
     +---- Data types
     +---- Joins
     +---- Calculated fields
     +---- Filters
```

The dataset provides the structure that visuals consume.

---

# 7. Analysis

An analysis is where authors build analytical content.

Example:

```text
Analysis
 |
 +-- KPI
 +-- Line Chart
 +-- Bar Chart
 +-- Table
 +-- Filters
 +-- Parameters
 +-- Calculated Fields
```

It is the working/authoring environment.

---

# 8. Dashboard

A dashboard is the published, shareable analytical view.

Typical flow:

```text
Create Analysis
      |
      v
Build Visuals
      |
      v
Publish
      |
      v
Dashboard
```

Users commonly consume dashboards without modifying the underlying analysis.

---

# 9. Visuals

A visual represents data graphically.

Examples:

```text
Bar chart
Line chart
Pie chart
Table
Pivot table
KPI
Gauge
Scatter plot
Map
Tree map
Heat map
```

Choose the visual according to the question being answered.

---

# 10. KPI Visual

A KPI is useful for displaying an important metric.

Example:

```text
TOTAL ORDERS

1,245,890
```

Other examples:

```text
Revenue
Error Count
Availability
Latency
Active Users
```

---

# 11. Line Chart

Line charts are useful for trends over time.

Example:

```text
Requests
  |
  |       /\
  |      /  \      /\
  |  /\ /    \____/  \
  +------------------------> Time
```

Useful for:

```text
Traffic
CPU usage
Revenue
Errors
Latency
```

---

# 12. Bar Chart

Bar charts are useful for category comparison.

```text
Payments   █████████████
Orders     █████████
Users      ██████
Search     ████
```

Useful for:

```text
Top services
Top regions
Top error types
Revenue by product
```

---

# 13. Tables

Tables are useful when exact values matter.

| Service | Requests | Errors | Error Rate |
|---|---:|---:|---:|
| Payments | 500000 | 1000 | 0.20% |
| Orders | 350000 | 400 | 0.11% |
| Search | 250000 | 150 | 0.06% |

---

# 14. Filters

Filters restrict the data displayed by a visual or analysis.

Example:

```text
Environment = Production
Region = ap-south-1
Date = Last 7 Days
```

Conceptually:

```text
Dataset
   |
   v
Filter
   |
   v
Reduced Data
   |
   v
Visual
```

---

# 15. Filter Example

Suppose data contains:

```text
dev
test
stage
prod
```

The user selects:

```text
environment = prod
```

The dashboard displays production data only.

---

# 16. Filter Types

Common filtering concepts include:

```text
Field filters
Date filters
Numeric filters
String filters
Top/bottom filters
Relative date filters
```

Available options depend on the field and visual.

---

# 17. Date Filters

Operational dashboards may use:

```text
Last 24 hours
Last 7 days
Last 30 days
```

Business dashboards may use:

```text
This month
Previous month
Quarter
Year
```

Date filtering is particularly important for large datasets.

---

# 18. Parameters

Parameters allow users to dynamically control analysis behavior.

Example:

```text
Select Metric:

Revenue
Orders
Users
```

Conceptually:

```text
User Selection
      |
      v
Parameter
      |
      v
Calculated Logic
      |
      v
Visual
```

---

# 19. Calculated Fields

A calculated field derives a value from existing fields.

Example:

```text
Error Rate =
Errors / Requests
```

Other examples:

```text
Profit = Revenue - Cost

Average Order Value =
Revenue / Orders
```

---

# 20. Example – Error Rate

Suppose:

```text
Requests = 1,000,000
Errors   = 2,000
```

Then:

```text
Error Rate
=
2,000 / 1,000,000
=
0.002
=
0.2%
```

This can be represented as a calculated field or metric.

---

# 21. Aggregations

Common analytical aggregations include:

```text
SUM
COUNT
COUNT DISTINCT
AVG
MIN
MAX
```

Example:

```text
SUM(revenue)
```

calculates total revenue.

```text
AVG(latency)
```

calculates average latency.

---

# 22. COUNT vs COUNT DISTINCT

Data:

```text
user_id
-------
100
100
101
102
102
```

`COUNT`:

```text
5
```

`COUNT DISTINCT`:

```text
3
```

Unique users:

```text
100
101
102
```

---

# 23. Dimensions vs Measures

### Dimension

A field used for grouping or categorization.

Examples:

```text
Service
Region
Environment
Date
Product
```

### Measure

A numeric value being measured.

Examples:

```text
Revenue
Requests
Errors
Latency
Quantity
```

Mental model:

```text
Dimension
    +
Measure
    =
Visual
```

---

# 24. Example – Dimension and Measure

Question:

```text
How many requests does each service receive?
```

Dimension:

```text
Service
```

Measure:

```text
Request Count
```

Result:

```text
Payments   500K
Orders     350K
Search     250K
```

---

# 25. Dataset Preparation

Before building dashboards, inspect:

```text
Column names
Data types
Null values
Duplicate records
Incorrect values
Date fields
Numeric fields
Relationships
```

Poor data quality produces poor dashboards.

```text
Garbage In
    ↓
Garbage Dashboard
```

---

# 26. Data Types

Typical types include:

```text
String
Integer
Decimal
Date
Datetime
Boolean
```

Example:

```text
service        → String
request_count  → Integer
error_rate     → Decimal
event_time     → DateTime
is_success     → Boolean
```

Correct data types are important for filtering and visualization.

---

# 27. Dataset Joins

Sometimes data is split across multiple tables.

Example:

```text
orders
   |
   +-- customer_id
          |
          v
customers
```

Join condition:

```text
orders.customer_id = customers.customer_id
```

A good data model makes reporting easier.

---

# 28. Example Dataset Model

```text
                    Customers
                        |
                    customer_id
                        |
                        v
Orders ------------ customer_id
   |
order_id
   |
   v
Order Items
```

Avoid unnecessary joins when a curated analytical table can serve the dashboard efficiently.

---

# 29. Direct Query

In Direct Query mode, QuickSight queries the underlying data source when data is requested.

Conceptually:

```text
User
 |
 v
QuickSight
 |
 v
Data Source
 |
 v
Result
```

Benefits:

```text
Fresh data
No imported copy required for the query path
Useful when source freshness is important
```

Trade-offs:

```text
Source performance matters
Repeated queries can load the source
Network/service latency can matter
```

---

# 30. SPICE

SPICE stands for:

```text
Super-fast, Parallel, In-memory Calculation Engine
```

It is QuickSight's in-memory data store for supported imported datasets.

Conceptually:

```text
Source
  |
  v
SPICE
  |
  v
QuickSight
  |
  v
Dashboard
```

---

# 31. Why Use SPICE?

Potential benefits:

```text
Fast dashboard interaction
Reduced repeated source queries
Scalable analytical performance
Reduced source workload
```

For dashboards with many repeated readers, SPICE can be valuable.

---

# 32. Direct Query vs SPICE

| Feature | Direct Query | SPICE |
|---|---|---|
| Data freshness | Source-dependent | Refresh-dependent |
| Query source | Underlying source | SPICE |
| Dashboard speed | Source-dependent | Often very fast |
| Source load | Can increase | Reduced |
| Refresh | Usually no import refresh | Required |
| Good for | Freshness | Interactive performance |

Actual choice should consider data size, freshness, concurrency, source capabilities and cost.

---

# 33. Example Decision

Requirement:

```text
Dashboard needs very fresh source data
```

Possible choice:

```text
Direct Query
```

Requirement:

```text
Many users repeatedly query a large analytical dataset
```

Possible choice:

```text
SPICE
```

Always evaluate the complete workload before deciding.

---

# 34. SPICE Refresh

Imported datasets need refreshes to update their data.

Conceptually:

```text
S3 / Athena
     |
     v
Refresh
     |
     v
SPICE
     |
     v
Dashboard
```

Refresh frequency should match business requirements.

---

# 35. Full vs Incremental Refresh

For large datasets, reloading everything can be inefficient.

Conceptually:

```text
Full Refresh

1 TB
 |
 v
Reload 1 TB
```

Versus an incremental pattern:

```text
Existing 1 TB
+
New 5 GB
 |
 v
Process new data
```

Use incremental refresh where supported and appropriate for the dataset.

---

# 36. QuickSight + Athena

A common serverless analytics architecture:

```text
Application
    |
    v
S3
    |
    v
Glue Data Catalog
    |
    v
Athena
    |
    v
QuickSight
    |
    v
Dashboard
```

This separates:

```text
Storage
Query
Visualization
```

---

# 37. QuickSight + S3

QuickSight can work with S3 data through supported dataset mechanisms.

For larger data-lake environments, a common pattern is:

```text
S3
 |
 v
Glue Catalog
 |
 v
Athena
 |
 v
QuickSight
```

---

# 38. QuickSight + Redshift

Example:

```text
Applications
     |
     v
ETL / ELT
     |
     v
Redshift
     |
     v
QuickSight
     |
     v
Dashboards
```

Redshift is often used for warehouse-oriented workloads.

---

# 39. QuickSight + RDS

Possible architecture:

```text
Application
    |
    v
RDS
    |
    v
QuickSight
```

However, heavy BI workloads should not unnecessarily impact a production database.

A safer architecture can be:

```text
Production RDS
      |
      v
Replica / ETL
      |
      v
Analytics Store
      |
      v
QuickSight
```

---

# 40. DevOps Use Cases

QuickSight can visualize:

```text
Infrastructure metrics
Application logs
CloudTrail activity
WAF events
Security events
Cost data
Deployment metrics
Incident metrics
SLA / SLO metrics
```

---

# 41. DevOps Dashboard Example

```text
                 PRODUCTION HEALTH

Requests          15.2M
Errors             2.1K
Error Rate         0.014%
P95 Latency        220 ms
Availability       99.98%

Top Error Services
Payments           900
Orders             500
Search             300
```

---

# 42. Security Dashboard Example

```text
WAF BLOCKS             1.2M
5XX EVENTS              4.5K
SUSPICIOUS IPs            320
FAILED LOGINS           8.1K
CLOUDTRAIL ALERTS          92
```

---

# 43. Cost Dashboard Example

```text
AWS ACCOUNT COST

EC2       $12,000
RDS        $4,500
S3         $1,200
Athena       $800
EKS        $6,500
```

Useful views:

```text
Cost by account
Cost by service
Cost by region
Daily trend
Monthly trend
Forecast
```

---

# 44. QuickSight User Roles

QuickSight has different user capabilities depending on account configuration, edition and current AWS feature set.

Common concepts include:

```text
Administrators
Authors
Readers
```

Always verify current AWS documentation and pricing when designing access.

---

# 45. Author vs Reader

Conceptually:

### Author

Can create and modify analytical content subject to permissions.

### Reader

Primarily consumes dashboards and reports.

Example:

```text
Data Engineer
   ↓
Author

Business Manager
   ↓
Reader
```

---

# 46. IAM and QuickSight

Think in layers:

```text
IAM
 |
 v
AWS account / API permissions
 |
 v
QuickSight permissions
 |
 v
Dataset / Dashboard access
```

Giving someone QuickSight access does not automatically mean they can access every underlying AWS data source.

Use least privilege.

---

# 47. Least Privilege

Production environments should provide only required access.

Avoid giving every analytics user:

```text
AdministratorAccess
```

Prefer controlled access:

```text
QuickSight Author
   |
   +--> Required datasets
   +--> Required dashboards
   +--> Required data sources
```

---

# 48. Row-Level Security

Row-Level Security (RLS) controls which rows a user can see.

Example:

| Region | Sales |
|---|---:|
| India | 100000 |
| USA | 150000 |
| UK | 80000 |

User A:

```text
Allowed region = India
```

sees India data.

User B:

```text
Allowed region = USA
```

sees USA data.

---

# 49. RLS Mental Model

```text
Dashboard
    |
    v
Dataset
    |
    v
RLS Rule
    |
    +---- User A → India
    |
    +---- User B → USA
    |
    +---- User C → UK
```

RLS is useful for multi-tenant and regional dashboards.

---

# 50. Column-Level Security

Column-level controls can restrict sensitive fields where supported.

Example:

```text
Employee
Department
Salary
Email
```

A manager may need:

```text
Employee
Department
```

while:

```text
Salary
```

is restricted.

Use the security features available for the QuickSight edition and current AWS configuration.

---

# 51. Multi-Tenant Dashboard

SaaS example:

```text
Company A
Company B
Company C
```

One dashboard:

```text
Customer Dashboard
```

RLS:

```text
Company A → A rows
Company B → B rows
Company C → C rows
```

This can avoid creating separate dashboards for every tenant.

---

# 52. Dashboard Sharing

Share dashboards only with authorized users/groups.

A practical model:

```text
Admin
 |
 +-- Authors
 |
 +-- Readers
 |
 +-- Groups
```

Groups are generally easier to manage than hundreds of individual permissions.

---

# 53. Dashboard Design Principles

A good dashboard should answer:

```text
What happened?
Why did it happen?
Is action required?
```

Avoid:

```text
50 random charts
```

Prefer:

```text
KPIs
+
Trend
+
Top contributors
+
Filters
+
Drill-down
```

---

# 54. Example Executive Dashboard

```text
-------------------------------------------------
| Revenue | Orders | Users | Conversion Rate   |
-------------------------------------------------

Revenue Trend
-----------------------------------------------

Orders by Region
-----------------------------------------------

Top Products
-----------------------------------------------

Filters:
Date | Region | Product
-------------------------------------------------
```

---

# 55. Example DevOps Dashboard

```text
-------------------------------------------------
| Requests | 5XX | P95 | Availability           |
-------------------------------------------------

Traffic Trend
-----------------------------------------------

5XX Errors by Service
-----------------------------------------------

Latency by Service
-----------------------------------------------

Top Failing APIs
-----------------------------------------------
```

---

# 56. Common QuickSight Mistakes

### Mistake 1

Using raw, unoptimized data.

### Mistake 2

Running heavy BI queries directly against production databases.

### Mistake 3

No partition strategy in Athena.

### Mistake 4

No SPICE refresh strategy.

### Mistake 5

Giving excessive permissions.

### Mistake 6

Creating too many dashboards.

### Mistake 7

No RLS for multi-tenant data.

### Mistake 8

Putting too many visuals on one dashboard.

---

# 57. Troubleshooting – Dashboard Slow

Check:

```text
1. Direct Query or SPICE?
2. Dataset size?
3. Number of visuals?
4. Complex calculations?
5. Expensive joins?
6. Athena query performance?
7. Redshift query performance?
8. Filters?
9. Data refresh status?
```

If using Athena:

```text
QuickSight
   |
   v
Athena
   |
   v
S3
```

Troubleshoot every layer.

---

# 58. Troubleshooting – Data Not Updated

If a dashboard shows old data:

```text
Check:
   |
   +-- SPICE refresh status
   |
   +-- Dataset refresh schedule
   |
   +-- Source data
   |
   +-- Athena query
   |
   +-- Glue Catalog
```

For Direct Query:

```text
Check the underlying source.
```

---

# 59. Troubleshooting – Access Denied

Check:

```text
QuickSight user/group
IAM permissions
Dataset permissions
Data source permissions
S3 permissions
Athena permissions
Glue permissions
KMS permissions
```

For S3-based architectures, verify bucket policies and encryption configuration.

---

# 60. Troubleshooting – Athena + QuickSight

Architecture:

```text
QuickSight
    |
    X
Athena
    |
    v
S3
```

Check:

```text
Athena workgroup
Glue database
Glue table
S3 location
S3 permissions
KMS permissions
Query result location
```

---

# 61. QuickSight Security Architecture

Simplified:

```text
                    IAM
                     |
              +------+------+
              |             |
         QuickSight        S3
              |             |
              v             v
           Dataset        Data
              |
              v
          RLS / Access
              |
              v
          Dashboard
```

Encryption should be designed according to organizational requirements.

---

# 62. QuickSight with KMS

Sensitive analytics environments may require encryption controls.

Conceptually:

```text
Data
 |
 v
S3 / Athena / Warehouse
 |
 v
Encrypted
 |
 v
QuickSight
```

KMS can be part of the broader AWS encryption architecture.

Ensure key policies and service permissions are correctly configured.

---

# 63. QuickSight + Lake Formation

In a governed data lake:

```text
S3
 |
 v
Lake Formation
 |
 v
Glue Catalog
 |
 v
Athena
 |
 v
QuickSight
```

Lake Formation can provide centralized data permissions for supported lake architectures.

---

# 64. QuickSight + CloudTrail

A common audit analytics flow:

```text
AWS API Activity
       |
       v
CloudTrail
       |
       v
S3
       |
       v
Athena
       |
       v
QuickSight
```

Possible dashboard metrics:

```text
API Calls
Failed API Calls
IAM Changes
Console Logins
Security Group Changes
KMS Events
```

---

# 65. QuickSight + WAF

Security analytics architecture:

```text
AWS WAF
   |
   v
Logs
   |
   v
S3
   |
   v
Athena
   |
   v
QuickSight
```

Dashboard:

```text
Requests
Blocked Requests
Top IPs
Top Rules
Top Countries
Attack Trends
```

---

# 66. QuickSight + Incident Management

Example:

```text
Incident System
      |
      v
Incident Data
      |
      v
S3 / Database
      |
      v
Athena
      |
      v
QuickSight
```

Dashboard:

```text
Open Incidents
MTTR
MTTA
Incidents by Service
Incidents by Severity
Incidents by Environment
```

---

# 67. MTTR

MTTR may mean Mean Time To Recovery or Mean Time To Resolve depending on the organization's definition.

Conceptually:

```text
Resolution Time
-
Incident Start Time
```

Then:

```text
MTTR =
Total Resolution Time
/
Number of Resolved Incidents
```

Standardize the definition within the organization.

---

# 68. SLO Dashboard

Example:

```text
Availability SLO        99.95%
Current Availability    99.98%

P95 Latency              210 ms
P99 Latency              450 ms

Error Budget Remaining   72%
```

This provides engineering teams an operational view.

---

# 69. Cost Optimization for QuickSight

Consider:

```text
SPICE capacity
Dataset size
Refresh frequency
Number of users
Dashboard complexity
Underlying source cost
Athena scanned data
```

Optimize the whole analytics system:

```text
S3
+
Glue
+
Athena
+
QuickSight
```

---

# 70. QuickSight Performance Checklist

```text
[ ] Use efficient source tables
[ ] Use Parquet where appropriate
[ ] Partition Athena datasets
[ ] Avoid SELECT *
[ ] Reduce unnecessary columns
[ ] Reduce expensive joins
[ ] Use SPICE when appropriate
[ ] Use incremental refresh where supported
[ ] Avoid excessive visuals
[ ] Use filters intelligently
[ ] Pre-aggregate large datasets
```

---

# 71. QuickSight vs Grafana

QuickSight:

```text
Business intelligence
Analytics
Reporting
AWS data
Interactive dashboards
```

Grafana:

```text
Observability
Metrics
Logs
Time-series
Operational monitoring
```

They can coexist.

```text
Grafana
→ Real-time infrastructure monitoring

QuickSight
→ Historical business/operational analytics
```

---

# 72. QuickSight vs CloudWatch Dashboard

CloudWatch Dashboard:

```text
AWS operational metrics
Near-real-time monitoring
Infrastructure visibility
```

QuickSight:

```text
Business analytics
Historical analysis
Cross-source reporting
Interactive BI
```

---

# 73. QuickSight vs Athena

They solve different problems:

```text
Athena
→ Query data

QuickSight
→ Visualize and analyze data
```

Common architecture:

```text
S3
 |
 v
Athena
 |
 v
QuickSight
```

---

# 74. Interview Question – What is QuickSight?

Amazon QuickSight is AWS's cloud-based business intelligence service used to analyze data and create interactive dashboards and visualizations.

---

# 75. Interview Question – What is SPICE?

SPICE is QuickSight's in-memory analytical engine and data store for supported imported datasets, designed for fast analytics.

---

# 76. Interview Question – SPICE vs Direct Query?

Direct Query reads from the underlying data source when queries are executed.

SPICE imports supported data into QuickSight's in-memory engine and uses refreshes to update that imported data.

---

# 77. Interview Question – Analysis vs Dashboard?

```text
Analysis
→ Authoring environment

Dashboard
→ Published/shareable analytical content
```

---

# 78. Interview Question – Dataset vs Data Source?

```text
Data Source
→ Connection/source configuration

Dataset
→ Data model prepared for analysis
```

---

# 79. Interview Question – What Is RLS?

Row-Level Security restricts which rows of a dataset a user can access.

Example:

```text
User A → India
User B → USA
User C → UK
```

Same dashboard, different visible data.

---

# 80. Interview Question – Build QuickSight Dashboard From S3

Answer:

```text
1. Store data in S3.
2. Organize the data efficiently.
3. Register metadata in Glue when using Athena.
4. Create/validate Athena tables.
5. Test SQL in Athena.
6. Connect QuickSight to Athena.
7. Create dataset.
8. Choose SPICE or Direct Query.
9. Create visuals.
10. Add filters and calculated fields.
11. Configure permissions/RLS.
12. Publish dashboard.
```

---

# 81. Interview Question – Troubleshoot Slow QuickSight

Answer:

```text
1. Determine SPICE vs Direct Query.
2. Check dataset size.
3. Check source query performance.
4. Check Athena data scanned if applicable.
5. Check joins.
6. Check calculated fields.
7. Reduce unnecessary visuals.
8. Optimize source data.
9. Consider pre-aggregation.
10. Consider SPICE where appropriate.
```

---

# 82. Interview Question – Secure QuickSight

Use:

```text
IAM
+
QuickSight permissions
+
Dataset permissions
+
RLS where required
+
Column-level controls where supported
+
S3/Athena permissions
+
KMS
+
Lake Formation where applicable
```

Follow least privilege.

---

# 83. Interview Question – QuickSight + Athena Architecture

```text
S3
 |
 v
Glue Catalog
 |
 v
Athena
 |
 v
QuickSight
 |
 v
Dashboard
```

Advantages:

```text
Serverless
Scalable
S3 data-lake compatible
SQL analytics
BI visualization
```

---

# 84. Interview Question – Reduce Dashboard Cost

Answer:

```text
1. Optimize source data.
2. Reduce Athena data scanned.
3. Partition datasets.
4. Use columnar formats.
5. Avoid unnecessary refreshes.
6. Use SPICE appropriately.
7. Reduce unnecessary dataset size.
8. Pre-aggregate repeated workloads.
9. Remove unused dashboards/datasets.
10. Monitor usage and capacity.
```

---

# 85. Real-World DevOps Architecture

```text
                         APPLICATIONS
                              |
                 +------------+------------+
                 |            |            |
               Logs        Metrics       Events
                 |            |            |
                 +------------+------------+
                              |
                              v
                             S3
                              |
                    +---------+---------+
                    |                   |
                  Raw               Curated
                    |                   |
                    +---------+---------+
                              |
                              v
                         Glue Catalog
                              |
                              v
                            Athena
                              |
                              v
                         QuickSight
                              |
               +--------------+--------------+
               |              |              |
               v              v              v
          DevOps Board   Security Board   Executive
```

---

# 86. Production Best Practices

```text
1. Separate raw and curated data.
2. Optimize S3 data layout.
3. Use Athena for data-lake SQL.
4. Use Parquet for suitable analytical workloads.
5. Partition according to query patterns.
6. Use SPICE when it provides value.
7. Do not overload production databases.
8. Implement RLS for multi-tenant use cases.
9. Apply least privilege.
10. Monitor refreshes and source queries.
11. Keep dashboards simple.
12. Remove unused datasets.
13. Control sharing.
14. Encrypt sensitive data.
15. Document dashboard ownership.
```

---

# 87. QuickSight Mental Model

```text
              QUICKSIGHT

Data Source
     |
     v
Dataset
     |
     +---- Fields
     +---- Joins
     +---- Calculations
     +---- Filters
     |
     v
Analysis
     |
     +---- Visuals
     +---- Parameters
     +---- Filters
     |
     v
Dashboard
     |
     v
Users
```

Execution:

```text
                 +--> Direct Query --> Source
                /
Dashboard --> QuickSight
                                 +--> SPICE --> Imported Data
```

---

# 88. Final Revision Sheet

## QuickSight

```text
AWS Business Intelligence / Visualization
```

## Data Source

```text
Where the data originates
```

## Dataset

```text
Data model used for analysis
```

## Analysis

```text
Authoring workspace
```

## Dashboard

```text
Published/shareable analytics
```

## Visual

```text
Graphical representation of data
```

## Filter

```text
Restricts displayed data
```

## Parameter

```text
User-controlled input to analysis logic
```

## Calculated Field

```text
Derived value
```

## SPICE

```text
QuickSight in-memory analytical engine
```

## Direct Query

```text
Query underlying source
```

## RLS

```text
Restrict rows by user/context
```

---

# 89. Final Takeaways

1. QuickSight is AWS's cloud BI service.
2. Data Source represents where data comes from.
3. Dataset provides the data model for analysis.
4. Analysis is the authoring environment.
5. Dashboard is the shareable analytical view.
6. Visuals turn data into charts, KPIs, tables and other views.
7. Filters restrict data shown to users.
8. Parameters enable dynamic analytical behavior.
9. Calculated fields derive new metrics.
10. SPICE provides fast in-memory analytics for imported datasets.
11. Direct Query queries the underlying data source.
12. SPICE requires refreshes to keep imported data current.
13. Athena + S3 + Glue + QuickSight is a common serverless data-lake BI architecture.
14. RLS is important for multi-tenant dashboards.
15. IAM and QuickSight permissions should follow least privilege.
16. KMS and Lake Formation can be part of a governed analytics architecture.
17. Optimize the entire data path, not just QuickSight.
18. QuickSight can visualize incidents, WAF events, CloudTrail activity, costs, SLOs and application analytics.
19. Grafana and CloudWatch are generally more operationally focused, while QuickSight is strongly BI/analytics focused.
20. A good dashboard should answer what happened, why it happened, and whether action is required.

---

# End of Part 1
