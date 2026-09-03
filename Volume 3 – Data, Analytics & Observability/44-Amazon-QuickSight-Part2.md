# 44-Amazon-QuickSight-Part2

> AWS DevOps Playbook — Volume 1: Data, Analytics & Observability  
> Chapter 44 — Amazon QuickSight Part 2: Advanced Features, Security, Embedding, APIs, Automation, Performance, Cost Optimization & Production Architecture

---

# Chapter Objectives

In Part 2, we will cover:

- Advanced QuickSight architecture
- SPICE optimization
- Dataset refresh strategies
- Athena optimization for QuickSight
- Row-Level Security in depth
- Column-level security concepts
- User and group management
- Dashboard and dataset permissions
- Cross-account considerations
- QuickSight APIs
- CLI and SDK automation
- Infrastructure as Code
- Dashboard deployment strategy
- Embedding
- Secure dashboard access
- Event-driven automation
- QuickSight monitoring
- CloudTrail integration
- Cost optimization
- Performance optimization
- Production architecture
- Security best practices
- Troubleshooting scenarios
- DevOps implementation examples
- Interview questions

---

# 1. QuickSight Advanced Architecture

A production analytics platform can look like:

```text
                    APPLICATIONS
                         |
             +-----------+-----------+
             |           |           |
            Logs       Events       Data
             |           |           |
             +-----------+-----------+
                         |
                         v
                    Amazon S3
                         |
              +----------+----------+
              |                     |
             Raw                 Curated
              |                     |
              +----------+----------+
                         |
                         v
                   Glue Catalog
                         |
                         v
                      Athena
                         |
              +----------+----------+
              |                     |
         Direct Query             SPICE
              |                     |
              +----------+----------+
                         |
                         v
                    QuickSight
                         |
              +----------+----------+
              |          |          |
            DevOps     Security   Executive
           Dashboard  Dashboard   Dashboard
```

---

# 2. Why Architecture Matters

QuickSight performance is not determined only by QuickSight.

The complete path matters:

```text
S3
 ↓
Glue
 ↓
Athena
 ↓
QuickSight
 ↓
Dashboard
```

If S3 data is poorly organized:

```text
Athena scans too much
```

If Athena queries are inefficient:

```text
QuickSight becomes slow
```

If the dashboard has too many visuals:

```text
Dashboard rendering becomes expensive
```

Therefore:

```text
Optimize the complete data pipeline.
```

---

# 3. SPICE Optimization

SPICE is useful when:

```text
Many users
+
Repeated dashboard queries
+
Large analytical datasets
```

Instead of:

```text
User
 ↓
QuickSight
 ↓
Athena
 ↓
S3
```

you can use:

```text
User
 ↓
QuickSight
 ↓
SPICE
```

This can reduce repeated source queries.

---

# 4. When SPICE Is a Good Choice

Consider SPICE when:

```text
Dashboard requires fast interaction
Many users access the same data
Source database should not receive repeated queries
Data can tolerate scheduled refresh
```

Example:

```text
1000 users
     |
     v
Same dashboard
     |
     v
SPICE
```

Instead of repeatedly hitting the source.

---

# 5. When Direct Query May Be Better

Direct Query can be appropriate when:

```text
Data freshness is critical
Source is optimized for analytical queries
Data is too large or unsuitable for import
Real-time/near-real-time access is required
```

But evaluate:

```text
Source capacity
Query complexity
Concurrency
Latency
Cost
```

---

# 6. SPICE Refresh Strategy

Suppose source data arrives hourly:

```text
10:00 → New data
11:00 → New data
12:00 → New data
```

A suitable refresh strategy may be:

```text
Source
  |
  +--> 10:05 refresh
  +--> 11:05 refresh
  +--> 12:05 refresh
```

The refresh schedule should match the business SLA.

Do not refresh every few minutes if the business only needs hourly data.

---

# 7. Full Refresh vs Incremental Refresh

Full refresh:

```text
Dataset = 1 TB

Refresh
 ↓
Read/process 1 TB
```

Incremental pattern:

```text
Existing = 1 TB
New Data = 5 GB

Refresh
 ↓
Process new/changed data
```

For large datasets, incremental refresh can significantly reduce processing requirements when the dataset and source support it.

---

# 8. Designing for Incremental Data

A common pattern is to have a timestamp or partition column:

```text
event_date
event_time
updated_at
created_at
```

Example:

```text
WHERE event_date >= current_date - interval '1' day
```

The exact implementation depends on the dataset and refresh capabilities.

---

# 9. Athena Optimization

If QuickSight uses Athena, optimize Athena first.

Bad:

```sql
SELECT *
FROM application_logs;
```

Better:

```sql
SELECT
    timestamp,
    service,
    status_code,
    latency
FROM application_logs;
```

Why?

```text
Less data read
Less processing
Lower Athena scan cost
Potentially faster QuickSight queries
```

---

# 10. Partitioning

Suppose data is stored as:

```text
year
month
day
hour
```

Example:

```text
s3://logs/
    year=2026/
        month=09/
            day=03/
```

A query filtering on partitions can avoid scanning unrelated data.

---

# 11. Partitioning and QuickSight

Without partition filtering:

```text
QuickSight
   |
   v
Athena
   |
   v
Scan huge amount of S3 data
```

With good partition filtering:

```text
QuickSight
   |
   v
Athena
   |
   v
Relevant partitions
   |
   v
Small scan
```

This can improve both performance and cost.

---

# 12. Columnar Formats

For analytical workloads, formats such as:

```text
Parquet
ORC
```

can be more efficient than raw CSV in many scenarios.

Example:

```text
CSV
 ↓
Large scan

Parquet
 ↓
Column pruning
 ↓
Less data read
```

---

# 13. Why Parquet Helps

Suppose a table has:

```text
100 columns
```

but QuickSight needs:

```text
service
status
latency
timestamp
```

A columnar format can allow the query engine to read only the required columns more efficiently.

---

# 14. Avoid SELECT *

Avoid:

```sql
SELECT *
FROM logs;
```

Prefer:

```sql
SELECT
    timestamp,
    service,
    environment,
    status_code,
    latency
FROM logs;
```

This is especially important for large S3 datasets queried through Athena.

---

# 15. Pre-Aggregation

Suppose raw data contains:

```text
500 million request records
```

Dashboard only needs:

```text
Daily requests by service
```

Instead of querying raw data repeatedly:

```text
500M rows
     |
     v
Aggregate
     |
     v
Daily Service Summary
     |
     v
QuickSight
```

This can dramatically reduce dashboard workload.

---

# 16. Example Aggregated Table

Raw:

```text
timestamp
service
status
latency
request_id
user_id
```

Curated:

```text
date
service
request_count
error_count
avg_latency
p95_latency
```

QuickSight can use the curated table for executive dashboards.

---

# 17. RLS in Production

RLS is critical when:

```text
Multiple customers
Multiple departments
Multiple regions
Multiple business units
```

share a dashboard.

Example:

```text
Dashboard
    |
    v
Dataset
    |
    v
RLS
    |
    +--> customer=A
    +--> customer=B
    +--> customer=C
```

---

# 18. RLS Rule Dataset

Conceptually, create a permissions mapping:

| User | Region |
|---|---|
| alice@example.com | India |
| bob@example.com | USA |
| carol@example.com | UK |

Then the dashboard can enforce:

```text
Alice → India data
Bob → USA data
Carol → UK data
```

The exact QuickSight RLS configuration should follow the current AWS implementation.

---

# 19. Multi-Value RLS

A user may need access to multiple regions.

Example:

```text
Alice
 |
 +-- India
 +-- Singapore
 +-- UAE
```

The security mapping can represent multiple allowed values.

---

# 20. Group-Based Access

Instead of managing every user individually:

```text
Finance Users
Security Users
DevOps Users
Executives
```

Assign dashboard access to groups where appropriate.

Benefits:

```text
Simpler administration
Easier onboarding
Easier offboarding
Consistent access
```

---

# 21. Example Group Model

```text
QuickSight
|
+-- Executive Group
|
+-- Finance Group
|
+-- DevOps Group
|
+-- Security Group
|
+-- Engineering Group
```

Each group receives appropriate dashboard access.

---

# 22. Dashboard Permission Model

Think:

```text
User / Group
      |
      v
Dashboard
      |
      v
Dataset
      |
      v
Data Source
```

All required permission layers must be configured correctly.

---

# 23. Least Privilege

A user who only needs to read a dashboard should not automatically receive:

```text
Dataset modification
Data source administration
QuickSight administration
```

Use:

```text
Reader → consume
Author → create/edit
Admin → administer
```

as the conceptual separation.

---

# 24. Sensitive Data

Do not expose sensitive fields unnecessarily.

Example:

```text
Employee ID
Name
Department
Salary
Personal Email
```

If a dashboard does not need:

```text
Salary
Personal Email
```

do not include them.

Best principle:

```text
Collect less
Expose less
Share less
```

---

# 25. KMS and Encryption

Encryption may involve:

```text
S3
Athena
QuickSight
KMS
```

Example:

```text
Data
 |
 v
S3 Encryption
 |
 v
Athena
 |
 v
QuickSight
```

When customer-managed keys are used, verify:

```text
Key policy
IAM policy
Service permissions
Cross-service access
```

---

# 26. Lake Formation + QuickSight

Governed data lake:

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

Lake Formation can provide centralized governance for supported data lake resources.

This is useful when many teams share a common data lake.

---

# 27. QuickSight APIs

QuickSight provides APIs that can be used for automation.

Automation can be used for:

```text
Users
Groups
Datasets
Data sources
Analyses
Dashboards
Templates
Refreshes
Permissions
```

This allows DevOps-style management rather than manually configuring everything.

---

# 28. AWS CLI

AWS CLI can be used to interact with QuickSight APIs.

Conceptually:

```bash
aws quicksight <command>
```

Examples of operation categories include:

```text
list
describe
create
update
delete
```

Always check the current AWS CLI documentation for exact command syntax.

---

# 29. Boto3

Python automation can use Boto3.

Conceptually:

```python
import boto3

client = boto3.client("quicksight")
```

Then:

```python
client.list_dashboards(...)
```

or other supported API operations.

---

# 30. Why Automate QuickSight?

Manual:

```text
Developer
 ↓
Console
 ↓
Click
 ↓
Click
 ↓
Click
```

Automated:

```text
Git
 ↓
CI/CD
 ↓
AWS CLI / Boto3
 ↓
QuickSight
```

Benefits:

```text
Repeatability
Version control
Consistency
Faster deployments
Reduced manual errors
```

---

# 31. Infrastructure as Code

Where supported and appropriate, infrastructure can be managed using:

```text
AWS CloudFormation
AWS CDK
Terraform
```

Conceptually:

```text
Code
 |
 v
Review
 |
 v
CI/CD
 |
 v
AWS
 |
 v
QuickSight Resources
```

Always verify which QuickSight resource types are supported by the IaC tool/version you use.

---

# 32. Terraform Concept

Terraform workflow:

```text
terraform code
      |
      v
terraform plan
      |
      v
terraform apply
      |
      v
AWS
```

Advantages:

```text
Infrastructure as code
Version control
Reviewable changes
Repeatable deployment
```

QuickSight resource coverage can change, so verify provider documentation before designing a complete Terraform strategy.

---

# 33. CI/CD for Analytics

Example:

```text
Developer
   |
   v
Git Repository
   |
   v
Pull Request
   |
   v
Code Review
   |
   v
CI
   |
   v
Deploy
   |
   v
QuickSight
```

This brings software engineering practices into analytics.

---

# 34. Dev / Test / Prod

Use separate environments when appropriate:

```text
Development
     |
     v
Testing
     |
     v
Production
```

Example:

```text
dev-dashboard
test-dashboard
prod-dashboard
```

Avoid making experimental changes directly to production dashboards.

---

# 35. Dashboard Promotion

Conceptually:

```text
DEV
 |
 | validate
 v
TEST
 |
 | approve
 v
PROD
```

This is similar to application deployment.

---

# 36. Templates and Reusable Content

For organizations with many similar dashboards, reusable assets/templates can reduce duplication.

Example:

```text
Base Dashboard
      |
      +---- Customer A
      +---- Customer B
      +---- Customer C
```

Combine with RLS where appropriate.

---

# 37. Embedded Analytics

QuickSight supports embedded analytics use cases.

Example:

```text
Customer Portal
      |
      v
Embedded QuickSight Dashboard
```

Instead of sending customers to a separate BI portal, analytics can appear inside an application.

---

# 38. Embedded Architecture

```text
User
 |
 v
Web Application
 |
 v
Application Backend
 |
 v
QuickSight API
 |
 v
Embedded Dashboard
```

The backend controls access and generates the appropriate embedded experience.

---

# 39. Why Embed QuickSight?

Useful for:

```text
SaaS applications
Customer portals
Internal applications
Partner portals
Operational platforms
```

Example:

```text
Monitoring SaaS
       |
       +-- Application
       |
       +-- Analytics
               |
               v
          QuickSight
```

---

# 40. Secure Embedding

Do not blindly expose dashboard URLs.

Use:

```text
Authentication
Authorization
Backend-controlled access
RLS where required
Short-lived/session-based mechanisms where supported
```

The exact embedding mechanism depends on the QuickSight embedding model being used.

---

# 41. Embedded Multi-Tenant SaaS

Example:

```text
                    SaaS APP
                       |
          +------------+------------+
          |            |            |
       Tenant A     Tenant B     Tenant C
          |            |            |
          +------------+------------+
                       |
                       v
                  QuickSight
                       |
                       v
                      RLS
```

Result:

```text
Tenant A → A data
Tenant B → B data
Tenant C → C data
```

---

# 42. API Automation – Refresh

A DevOps pipeline may trigger a dataset refresh after new data is available.

Conceptually:

```text
ETL complete
     |
     v
Trigger refresh
     |
     v
SPICE updated
     |
     v
Dashboard updated
```

This avoids waiting for a manually initiated refresh.

---

# 43. Event-Driven Analytics

Example:

```text
S3 new data
     |
     v
Event
     |
     v
Lambda
     |
     v
QuickSight API
     |
     v
Dataset refresh
```

This creates an automated analytics pipeline.

---

# 44. EventBridge

A broader event-driven architecture can be:

```text
EventBridge
     |
     v
Lambda / Step Functions
     |
     v
QuickSight API
```

Useful for orchestration and automation.

---

# 45. Step Functions

For complex workflows:

```text
Data Arrives
     |
     v
Validate Data
     |
     v
Run Athena
     |
     v
Validate Results
     |
     v
Refresh QuickSight
     |
     v
Notify Team
```

Step Functions can coordinate these steps where appropriate.

---

# 46. Lambda Use Case

Example:

```text
S3 Object Created
       |
       v
Lambda
       |
       +--> Validate file
       |
       +--> Start analytics process
       |
       +--> Trigger QuickSight refresh
```

Lambda should have only the required IAM permissions.

---

# 47. Monitoring QuickSight

Monitor:

```text
Dataset refreshes
Failures
Dashboard usage
Source query performance
Athena query performance
SPICE capacity/usage
Access patterns
API activity
```

Monitoring should cover the entire analytics stack.

---

# 48. CloudTrail + QuickSight

CloudTrail can provide API activity for supported AWS operations.

Conceptually:

```text
QuickSight API
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
QuickSight Security Dashboard
```

This creates a feedback loop for analytics and auditing.

---

# 49. Security Audit Dashboard

Example:

```text
QuickSight API Calls
        |
        +-- Create
        +-- Update
        +-- Delete
        +-- Permission changes
```

Visualize:

```text
API activity over time
Changes by user
Changes by resource
Failed actions
```

---

# 50. Data Freshness Monitoring

A production dashboard should show when data was last updated.

Example:

```text
Last Data Refresh
2026-09-03 08:05 IST

Data Status
HEALTHY
```

Useful for preventing users from trusting stale information.

---

# 51. Dashboard Health Indicator

Example:

```text
Data Freshness

< 15 min       HEALTHY
15–60 min      WARNING
> 60 min       STALE
```

The thresholds should match the actual business SLA.

---

# 52. QuickSight Performance Troubleshooting

When a dashboard is slow:

```text
Step 1
 ↓
SPICE or Direct Query?
```

Then:

```text
Step 2
 ↓
Dataset size?
```

Then:

```text
Step 3
 ↓
Number of visuals?
```

Then:

```text
Step 4
 ↓
Calculated fields?
```

Then:

```text
Step 5
 ↓
Joins?
```

Then:

```text
Step 6
 ↓
Source query?
```

---

# 53. Athena Troubleshooting

If QuickSight uses Athena:

```text
QuickSight
    |
    v
Athena
    |
    +-- SQL
    +-- Partitions
    +-- File format
    +-- Data scanned
    |
    v
S3
```

Check:

```text
Query execution time
Bytes scanned
Partition pruning
File format
Number of files
Table statistics where applicable
```

---

# 54. Small Files Problem

Suppose S3 contains:

```text
10 million tiny files
```

Athena may have significant overhead processing them.

Better:

```text
Many small files
      |
      v
Compaction
      |
      v
Larger optimized files
```

This can improve analytical performance.

---

# 55. Dashboard Too Complex

Problem:

```text
Dashboard
 |
 +-- 30 visuals
 +-- 15 filters
 +-- Complex calculations
 +-- Multiple joins
```

Possible solution:

```text
Dashboard
 |
 +-- Executive KPIs
 +-- Main trend
 +-- Top issues
 +-- Drill-down dashboard
```

Split complex analysis into logical views.

---

# 56. Slow Production Database

Problem:

```text
QuickSight
    |
    v
Production RDS
    |
    X
High load
```

Better:

```text
Production RDS
      |
      v
Read Replica / ETL
      |
      v
Analytics Store
      |
      v
QuickSight
```

Do not let analytics unnecessarily impact production workloads.

---

# 57. QuickSight Cost Optimization

Consider:

```text
User licensing
SPICE usage
Dataset size
Refresh frequency
Underlying data-source cost
Athena scan cost
Dashboard complexity
```

Do not optimize only one component.

---

# 58. Athena Cost Optimization

If QuickSight uses Athena:

```text
Cost
≈
Data scanned
```

Reduce data scanned using:

```text
Partitioning
Columnar formats
Column pruning
Predicate filtering
Pre-aggregation
Curated datasets
```

The exact Athena pricing model should be checked for the region and workload.

---

# 59. SPICE Cost Consideration

SPICE can improve dashboard performance, but imported data consumes SPICE capacity according to the current QuickSight pricing model.

Therefore:

```text
More unnecessary data
       ↓
More capacity
       ↓
Potentially higher cost
```

Remove unnecessary columns and rows.

---

# 60. Reduce Dataset Size

Suppose source has:

```text
100 columns
1 billion rows
```

Dashboard needs:

```text
10 columns
100 million relevant rows
```

Create a curated dataset:

```text
Raw
 |
 v
Filter + Select + Aggregate
 |
 v
Analytics Dataset
 |
 v
QuickSight
```

---

# 61. Refresh Frequency Optimization

Bad:

```text
Refresh every 5 minutes
```

when the business only needs:

```text
Hourly
```

Better:

```text
Refresh frequency = business requirement
```

Avoid unnecessary refresh processing.

---

# 62. QuickSight Security Checklist

```text
[ ] Least privilege
[ ] Strong authentication
[ ] Correct user/group permissions
[ ] RLS where required
[ ] Sensitive fields restricted
[ ] S3 permissions reviewed
[ ] Athena permissions reviewed
[ ] KMS policies reviewed
[ ] CloudTrail enabled
[ ] Embedded dashboards protected
[ ] Production changes controlled
```

---

# 63. QuickSight Production Checklist

```text
[ ] Data source optimized
[ ] Dataset optimized
[ ] SPICE / Direct Query decision documented
[ ] Refresh schedule configured
[ ] RLS tested
[ ] Permissions tested
[ ] Dashboard performance tested
[ ] Cost monitored
[ ] CloudTrail activity reviewed
[ ] Ownership documented
[ ] Backup / deployment strategy documented
```

---

# 64. Disaster Recovery Considerations

Think about:

```text
Dashboard definitions
Dataset definitions
Data source configuration
Permissions
Underlying data
Deployment automation
```

Do not assume that a dashboard alone is the complete recovery solution.

Your recovery design should consider how QuickSight resources and underlying datasets are recreated.

---

# 65. Backup Strategy

Conceptually:

```text
QuickSight configuration
        |
        v
Export / template / IaC strategy
        |
        v
Git / secure storage
        |
        v
Recovery
```

The exact export/import options depend on the QuickSight resource and current AWS capabilities.

---

# 66. Real-World DevOps Architecture

```text
                     APPLICATION
                          |
                          v
                    LOG / EVENT DATA
                          |
                          v
                         S3
                          |
                 +--------+--------+
                 |                 |
                Raw             Curated
                 |                 |
                 +--------+--------+
                          |
                          v
                    Glue Catalog
                          |
                          v
                        Athena
                          |
                 +--------+--------+
                 |                 |
             Direct Query         SPICE
                 |                 |
                 +--------+--------+
                          |
                          v
                     QuickSight
                          |
        +-----------------+-----------------+
        |                 |                 |
        v                 v                 v
   DevOps Board      Security Board    Executive Board
```

---

# 67. Automated Production Pipeline

```text
                    Developer
                        |
                        v
                    Git Repo
                        |
                        v
                       CI
                        |
                        v
                    IaC / API
                        |
                        v
                  QuickSight DEV
                        |
                        v
                    Validation
                        |
                        v
                  QuickSight TEST
                        |
                        v
                    Approval
                        |
                        v
                  QuickSight PROD
```

---

# 68. End-to-End Automated Analytics

```text
Application
    |
    v
S3
    |
    v
Glue
    |
    v
Athena
    |
    v
Data Validation
    |
    v
QuickSight Refresh
    |
    v
Dashboard
    |
    v
Users
```

Monitoring:

```text
CloudTrail
CloudWatch / operational monitoring
Athena query monitoring
QuickSight refresh monitoring
```

---

# 69. Scenario – Executive Dashboard Is Slow

### Problem

CEO opens dashboard and it takes 30 seconds.

### Investigation

```text
QuickSight
   |
   v
Direct Query
   |
   v
Athena
   |
   v
Huge raw table
```

### Solution

```text
Raw Data
   |
   v
Curated Data
   |
   v
Aggregated Dataset
   |
   v
SPICE
   |
   v
QuickSight
```

Result:

```text
Less source work
+
Less data
+
Faster dashboard
```

---

# 70. Scenario – Customer Sees Another Customer's Data

### Problem

Tenant A can see Tenant B's records.

### Root Cause

Missing or incorrectly configured:

```text
RLS
```

### Solution

```text
User / Tenant
      |
      v
RLS Mapping
      |
      v
Dataset
      |
      v
Dashboard
```

Test with multiple tenant accounts before production.

---

# 71. Scenario – Dashboard Shows Yesterday's Data

### Possible causes

```text
SPICE refresh failed
Refresh schedule incorrect
Source pipeline failed
Athena table not updated
Glue metadata stale
Wrong dataset
```

Troubleshooting:

```text
Source
 ↓
Athena
 ↓
Dataset refresh
 ↓
SPICE
 ↓
Dashboard
```

Check each layer.

---

# 72. Scenario – Athena Cost Is Too High

Problem:

```text
QuickSight
 ↓
Athena
 ↓
Scan TBs every day
```

Solutions:

```text
Partition data
Use Parquet
Select required columns
Filter partitions
Pre-aggregate
Use curated tables
Reduce unnecessary Direct Query traffic
```

---

# 73. Scenario – Production RDS CPU Is High

If QuickSight directly queries production RDS:

```text
QuickSight
   |
   v
RDS
   |
   X
High CPU
```

Solution:

```text
RDS
 |
 +--> Application
 |
 +--> Replica / ETL
          |
          v
     Analytics Store
          |
          v
      QuickSight
```

---

# 74. Scenario – 500 Dashboards

Problem:

```text
500 dashboards
```

Nobody knows:

```text
Owner
Purpose
Users
Data source
Last updated
```

Solution:

Create governance:

```text
Dashboard Catalog
 |
 +-- Owner
 +-- Business purpose
 +-- Data source
 +-- SLA
 +-- Users
 +-- Last review
 +-- Retention status
```

Remove unused dashboards.

---

# 75. Scenario – Customer Portal Analytics

Requirement:

```text
Customers need analytics inside SaaS application.
```

Architecture:

```text
Customer
   |
   v
SaaS Portal
   |
   v
Backend
   |
   v
QuickSight Embedded Analytics
   |
   v
RLS
   |
   v
Customer Dataset
```

Security must be enforced server-side and through appropriate QuickSight access controls.

---

# 76. Scenario – Security Analytics

Requirement:

```text
Show WAF and CloudTrail activity.
```

Architecture:

```text
WAF Logs --------+
                 |
CloudTrail ------+
                 |
                 v
                 S3
                 |
                 v
               Athena
                 |
                 v
             QuickSight
                 |
                 v
          Security Dashboard
```

---

# 77. Scenario – Incident Analytics

Requirement:

```text
Track MTTR and incident trends.
```

Architecture:

```text
Incident System
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
Incidents by Severity
Incidents by Service
MTTA
MTTR
SLA Breaches
```

---

# 78. Scenario – FinOps Dashboard

Architecture:

```text
AWS Billing / Cost Data
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
Monthly Spend
Daily Spend
Spend by Account
Spend by Service
Spend by Region
Forecast
Budget Variance
```

---

# 79. Interview Question – Explain QuickSight Architecture

Strong answer:

```text
QuickSight is a managed AWS BI service.

Data can come from services such as Athena,
Redshift, RDS, S3 and supported external sources.

The data is modeled as datasets and consumed through
analyses and dashboards.

For imported datasets, SPICE can provide fast in-memory
analytics. Direct Query can query the underlying source.

Security can include QuickSight permissions, IAM,
RLS, encryption and data-lake governance depending
on the architecture.
```

---

# 80. Interview Question – Why SPICE?

Answer:

```text
SPICE provides an in-memory analytical layer for
supported imported datasets.

It can provide fast dashboard performance and reduce
repeated queries against the underlying source.

The trade-off is that imported data needs refreshes
and consumes SPICE capacity.
```

---

# 81. Interview Question – When Would You Use Direct Query?

Answer:

```text
When source freshness is important and the underlying
source can handle the analytical workload.

I would evaluate query latency, concurrency, source
capacity, cost and data size before selecting it.
```

---

# 82. Interview Question – How Do You Optimize QuickSight With Athena?

Answer:

```text
1. Partition S3 data.
2. Use Parquet/ORC where appropriate.
3. Select only required columns.
4. Avoid SELECT *.
5. Filter partition columns.
6. Pre-aggregate frequently used metrics.
7. Reduce small-file problems.
8. Use curated datasets.
9. Use SPICE when appropriate.
10. Monitor Athena scan volume.
```

---

# 83. Interview Question – How Do You Secure Multi-Tenant QuickSight?

Answer:

```text
Use appropriate QuickSight sharing and permissions,
combine them with row-level security so each tenant
only sees its own records.

For embedded analytics, enforce authorization in the
application/backend and configure QuickSight access
controls appropriately.
```

---

# 84. Interview Question – How Do You Automate QuickSight?

Answer:

```text
Use AWS APIs through:

AWS CLI
Boto3
CloudFormation/CDK where supported
Terraform where supported

Integrate automation into CI/CD so resources and
configuration can be deployed consistently.
```

---

# 85. Interview Question – How Do You Trigger Refresh Automatically?

Answer:

```text
A data pipeline can complete first.

Then an event-driven component such as Lambda can
invoke the appropriate QuickSight API to start or
coordinate a dataset refresh.

For complex workflows, Step Functions or EventBridge
can be used where appropriate.
```

---

# 86. Interview Question – Dashboard Slow. What Do You Check?

Answer:

```text
1. SPICE vs Direct Query.
2. Dataset size.
3. Number of visuals.
4. Calculated fields.
5. Joins.
6. Athena/Redshift/RDS query performance.
7. Data layout.
8. Partitioning.
9. File format.
10. Data scanned.
11. Pre-aggregation opportunities.
```

---

# 87. Interview Question – Dashboard Has Stale Data

Answer:

```text
First identify whether the dataset uses SPICE or Direct Query.

For SPICE:
- Check refresh status.
- Check refresh schedule.
- Check source data.
- Check dataset configuration.

For Direct Query:
- Validate the source query and source data.

Then trace the pipeline:
Source → Athena/warehouse → QuickSight → Dashboard.
```

---

# 88. Interview Question – How Do You Reduce Athena Costs?

Answer:

```text
Use partitioning, columnar formats, column pruning,
predicate filtering, curated datasets and pre-aggregation.

I also monitor bytes scanned and identify dashboards
that generate unnecessary repeated queries.
```

---

# 89. Interview Question – QuickSight vs Grafana

Answer:

```text
QuickSight is primarily a BI and analytics platform.

Grafana is strongly focused on observability,
metrics, logs and operational monitoring.

I would use Grafana for real-time operational visibility
and QuickSight for historical/business analytics,
while using both when the organization needs both.
```

---

# 90. Interview Question – QuickSight vs CloudWatch

Answer:

```text
CloudWatch provides AWS monitoring capabilities,
metrics, logs, alarms and operational dashboards.

QuickSight provides richer BI-style analysis,
visualization and cross-source reporting.

They solve different but complementary problems.
```

---

# 91. Interview Question – QuickSight vs Athena

Answer:

```text
Athena is the query engine.

QuickSight is the BI and visualization layer.

A common architecture is:

S3 → Glue → Athena → QuickSight.
```

---

# 92. Interview Question – What Is RLS?

Answer:

```text
Row-Level Security controls which records a user can
see in a dataset.

For example:

User A → India
User B → USA
User C → UK

All users can use the same dashboard while seeing
only authorized records.
```

---

# 93. Interview Question – What Is Embedded Analytics?

Answer:

```text
Embedded analytics allows QuickSight analytical
content to be presented inside an application or
portal.

A common architecture is:

Application
   ↓
Backend
   ↓
QuickSight
   ↓
Embedded Dashboard
```

Authentication and authorization must be designed securely.

---

# 94. Interview Question – How Would You Design a Production QuickSight Platform?

Answer:

```text
I would separate raw and curated data in S3,
use Glue for cataloging, Athena or a warehouse for
querying, and QuickSight for visualization.

For high-concurrency dashboards, I would evaluate SPICE.
For freshness-sensitive use cases, I would evaluate
Direct Query.

I would implement RLS for multi-tenant requirements,
least-privilege permissions, encryption, CloudTrail
auditing, automated refreshes, CI/CD and monitoring.

I would also optimize partitioning, file formats,
dataset size and query patterns to control cost and latency.
```

---

# 95. QuickSight Production Architecture – Final

```text
                           USERS
                             |
             +---------------+---------------+
             |               |               |
          Executive        DevOps         Security
             |               |               |
             +---------------+---------------+
                             |
                             v
                        QUICKSIGHT
                             |
                    +--------+--------+
                    |                 |
                  SPICE          Direct Query
                    |                 |
                    |          +------+------+
                    |          |             |
                    |       Athena        Redshift
                    |          |             |
                    |          v             |
                    |          S3 <-----------+
                    |          |
                    |       Glue Catalog
                    |
                    v
                  Dataset
                    |
             +------+------+
             |             |
            RLS        Calculations
             |
             v
          Authorized
             Data
```

---

# 96. End-to-End DevOps + Security Analytics

```text
                         AWS ENVIRONMENT
                               |
             +-----------------+-----------------+
             |                 |                 |
            WAF             CloudTrail        Apps
             |                 |                 |
             +-----------------+-----------------+
                               |
                               v
                              S3
                               |
                    +----------+----------+
                    |                     |
                   Raw                 Curated
                    |                     |
                    +----------+----------+
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
             +-----------------+-----------------+
             |                 |                 |
             v                 v                 v
         WAF Board       Audit Board        DevOps Board
```

---

# 97. Golden Rules

Remember these:

```text
1. QuickSight = BI / visualization.
2. Athena = query engine.
3. S3 = scalable data lake storage.
4. Glue = catalog / metadata.
5. SPICE = in-memory analytics.
6. Direct Query = query source.
7. RLS = restrict rows.
8. IAM = AWS authorization layer.
9. Least privilege = default security principle.
10. Partitioning = reduce unnecessary scans.
11. Parquet = efficient analytical storage format.
12. Pre-aggregation = reduce repeated computation.
13. CI/CD = repeatable dashboard deployment.
14. APIs = automate QuickSight operations.
15. Embedding = analytics inside applications.
16. CloudTrail = audit AWS API activity.
17. Optimize the entire data path.
18. Monitor data freshness.
19. Control dashboard sprawl.
20. Test permissions before production.
```

---

# 98. Quick Revision – Part 1 + Part 2

```text
Data Source
    ↓
Dataset
    ↓
Analysis
    ↓
Dashboard
```

Data execution:

```text
                 +--> Direct Query --> Source
                /
QuickSight ----
                \
                 +--> SPICE --> Imported Data
```

AWS data lake:

```text
S3
 ↓
Glue
 ↓
Athena
 ↓
QuickSight
```

Security:

```text
IAM
 +
QuickSight permissions
 +
RLS
 +
Encryption
 +
Lake governance
```

Automation:

```text
Git
 ↓
CI/CD
 ↓
AWS API / IaC
 ↓
QuickSight
```

Embedded analytics:

```text
Application
 ↓
Backend
 ↓
QuickSight
 ↓
Embedded Dashboard
```

---

# 99. Final Interview Cheat Sheet

| Question | Short Answer |
|---|---|
| What is QuickSight? | AWS cloud BI and visualization service |
| What is SPICE? | In-memory analytical engine/data store |
| Direct Query? | Queries underlying source |
| Dataset? | Data model used by analysis |
| Analysis? | Authoring workspace |
| Dashboard? | Published/shareable analytical content |
| RLS? | Controls visible rows |
| Dimension? | Category/grouping field |
| Measure? | Numeric metric |
| Athena role? | SQL query engine |
| S3 role? | Data lake storage |
| Glue role? | Catalog/metadata |
| Why Parquet? | Efficient columnar analytics |
| Why partition? | Reduce unnecessary data scanning |
| How reduce Athena cost? | Partition, filter, column pruning, Parquet, aggregation |
| How secure multi-tenancy? | Permissions + RLS + secure application authorization |
| How automate? | CLI, Boto3, APIs, IaC where supported |
| How embed? | Integrate QuickSight analytics into an application |
| How troubleshoot stale data? | Trace source → query → refresh → dataset → dashboard |
| How troubleshoot slow dashboard? | Check SPICE/Direct Query, dataset, visuals, calculations, joins and source |

---

# 100. Final Takeaways

Amazon QuickSight should be viewed as the visualization and BI layer of a larger analytics platform.

A strong AWS architecture commonly looks like:

```text
                 DATA SOURCES
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
              +-------+-------+
              |               |
            SPICE       Direct Query
              |               |
              +-------+-------+
                      |
                      v
                  Dashboards
```

For production:

```text
Security
+
Governance
+
Performance
+
Cost Optimization
+
Automation
+
Monitoring
```

should all be considered together.

The most important concepts to remember are:

```text
QuickSight
    ↓
BI / Visualization

SPICE
    ↓
Fast imported analytics

Direct Query
    ↓
Source-based querying

Athena
    ↓
Serverless SQL

S3
    ↓
Data lake storage

Glue
    ↓
Metadata/catalog

RLS
    ↓
Row-level authorization

API / CLI / Boto3 / IaC
    ↓
Automation

Embedding
    ↓
Analytics inside applications
```

---

# End of 44-Amazon-QuickSight-Part2
