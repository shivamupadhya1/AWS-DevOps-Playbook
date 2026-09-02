# 43-Amazon-Athena-Part4

> AWS DevOps Playbook  
> Volume 1 – Data, Analytics & Observability  
> Chapter 43  
> Amazon Athena – Part 4: Advanced SQL, Performance Engineering, Data Lake Design & Real-World DevOps Scenarios

---

# Chapter Objectives

In this part, we will go deeper into:

- Athena query execution concepts
- Views and reusable SQL
- CTEs
- Window functions
- Joins and join optimization
- Handling NULL values
- Date and string functions
- JSON functions
- Query-result reuse
- Materialized views
- Result reuse
- Partition projection
- Bucketing
- Small-file problems
- Data compaction
- Parquet optimization
- Workgroup governance
- Query limits and controls
- Athena APIs
- Athena automation with AWS CLI
- Athena with Python/Boto3
- DevOps monitoring
- Production data-lake design
- Troubleshooting scenarios
- Interview questions

---

# 1. Athena Query Execution – High Level

At a high level, an Athena query follows a flow similar to:

```text
User / Application
        |
        v
    Athena API
        |
        v
   SQL Parser
        |
        v
 Query Planning
        |
        v
Metadata / Glue Catalog
        |
        v
 Read Required S3 Data
        |
        v
 Filter / Join / Aggregate
        |
        v
 Query Result
        |
        v
       S3
```

The exact internal implementation is AWS-managed, but this mental model is useful for troubleshooting.

---

# 2. What Happens When You Run a Query?

Suppose:

```sql
SELECT COUNT(*)
FROM orders
WHERE year = 2026
  AND month = 9;
```

Conceptually:

```text
SQL
 |
 v
Athena parses query
 |
 v
Catalog lookup
 |
 v
Determine table schema
 |
 v
Identify matching partitions
 |
 v
Read relevant S3 objects
 |
 v
Filter records
 |
 v
Aggregate
 |
 v
Return result
```

The biggest performance mistake is often forcing Athena to read much more data than necessary.

---

# 3. Query Planning

Query planning determines how Athena can execute a SQL statement.

Important factors include:

```text
Columns selected
Partitions
Filters
Joins
Aggregations
Data format
File layout
Statistics / metadata
```

Therefore:

```text
Good SQL
+
Good table design
+
Good file layout
=
Better Athena performance
```

---

# 4. Athena Views

A view is a saved SQL query.

Example:

```sql
CREATE VIEW active_users AS
SELECT
    user_id,
    name,
    email
FROM users
WHERE status = 'ACTIVE';
```

Then:

```sql
SELECT *
FROM active_users;
```

Views are useful for:

- Reusable logic
- Simplifying complex queries
- Standardizing reporting logic
- Hiding implementation details
- Providing controlled datasets

---

# 5. Views Do Not Usually Store the Result

A normal view should be thought of as:

```text
Saved SQL Definition
        |
        v
Query Underlying Data
```

It is not the same as storing a physical copy of the query result.

For repeated expensive queries, consider other approaches such as:

```text
CTAS
Materialized View
Pre-aggregated Dataset
```

depending on the workload.

---

# 6. CTE – Common Table Expression

CTEs make complex SQL easier to understand.

Example:

```sql
WITH successful_requests AS (
    SELECT *
    FROM logs
    WHERE status_code BETWEEN 200 AND 299
)
SELECT
    COUNT(*)
FROM successful_requests;
```

Structure:

```text
WITH temporary_result AS (...)
SELECT ...
FROM temporary_result;
```

CTEs are primarily a query-organization mechanism. Do not automatically assume that a CTE creates a persisted intermediate dataset.

---

# 7. Multiple CTEs

Example:

```sql
WITH requests AS (
    SELECT *
    FROM logs
    WHERE year = 2026
),

errors AS (
    SELECT *
    FROM requests
    WHERE status_code >= 500
)

SELECT
    COUNT(*) AS error_count
FROM errors;
```

This makes complex analytics easier to read and maintain.

---

# 8. Window Functions

Window functions calculate values across related rows without collapsing the result into one row per group.

Example:

```sql
SELECT
    user_id,
    order_date,
    amount,
    SUM(amount) OVER (
        PARTITION BY user_id
        ORDER BY order_date
    ) AS running_total
FROM orders;
```

Useful functions include:

```text
ROW_NUMBER()
RANK()
DENSE_RANK()
SUM()
AVG()
COUNT()
LAG()
LEAD()
```

---

# 9. ROW_NUMBER

Example:

```sql
SELECT
    user_id,
    order_id,
    order_date,
    ROW_NUMBER() OVER (
        PARTITION BY user_id
        ORDER BY order_date DESC
    ) AS rn
FROM orders;
```

This can help identify the latest order for each user.

For example:

```sql
WITH ranked AS (
    SELECT
        user_id,
        order_id,
        order_date,
        ROW_NUMBER() OVER (
            PARTITION BY user_id
            ORDER BY order_date DESC
        ) AS rn
    FROM orders
)
SELECT *
FROM ranked
WHERE rn = 1;
```

---

# 10. RANK vs DENSE_RANK

Example values:

```text
100
100
90
80
```

`RANK()` can produce:

```text
1
1
3
4
```

`DENSE_RANK()` produces:

```text
1
1
2
3
```

Choose according to the business meaning of ranking.

---

# 11. LAG and LEAD

These functions compare a row with previous or next rows.

Example:

```sql
SELECT
    event_time,
    value,
    LAG(value) OVER (
        ORDER BY event_time
    ) AS previous_value
FROM metrics;
```

Useful for:

```text
Trend analysis
Change detection
Before/after comparisons
Time-series analysis
```

---

# 12. Aggregation

Common aggregate functions:

```sql
COUNT(*)
SUM(amount)
AVG(amount)
MIN(amount)
MAX(amount)
```

Example:

```sql
SELECT
    region,
    COUNT(*) AS users
FROM users
GROUP BY region;
```

---

# 13. Conditional Aggregation

Example:

```sql
SELECT
    COUNT(*) AS total_requests,
    SUM(
        CASE
            WHEN status_code >= 500 THEN 1
            ELSE 0
        END
    ) AS server_errors
FROM logs;
```

This can calculate multiple metrics in one query.

---

# 14. NULL Handling

NULL is not the same as:

```text
0
''
'NULL'
```

Check for NULL:

```sql
WHERE email IS NULL;
```

Not:

```sql
WHERE email = NULL;
```

Use:

```sql
COALESCE(column_name, 'unknown')
```

to provide a fallback value.

---

# 15. COALESCE

Example:

```sql
SELECT
    user_id,
    COALESCE(phone, 'NOT_PROVIDED') AS phone
FROM users;
```

If `phone` is NULL:

```text
NOT_PROVIDED
```

is returned.

---

# 16. CASE Expressions

Example:

```sql
SELECT
    status_code,
    CASE
        WHEN status_code BETWEEN 200 AND 299 THEN 'SUCCESS'
        WHEN status_code BETWEEN 400 AND 499 THEN 'CLIENT_ERROR'
        WHEN status_code >= 500 THEN 'SERVER_ERROR'
        ELSE 'OTHER'
    END AS status_group
FROM logs;
```

This is very useful for operational analytics.

---

# 17. Date Filtering

Prefer explicit date filtering.

Example:

```sql
SELECT *
FROM logs
WHERE event_time >= TIMESTAMP '2026-09-01 00:00:00'
  AND event_time < TIMESTAMP '2026-09-02 00:00:00';
```

Using an inclusive start and exclusive end is often easier to reason about:

```text
>= start
< end
```

---

# 18. Date Functions

Common analytical operations include:

```text
Extract year
Extract month
Extract day
Date arithmetic
Timestamp comparison
Date truncation
```

Example:

```sql
SELECT
    year(event_time),
    month(event_time)
FROM logs;
```

Always use the functions supported by the Athena engine version in your environment.

---

# 19. String Functions

Common string operations include:

```text
length()
lower()
upper()
trim()
regexp_extract()
regexp_replace()
split()
concat()
```

Example:

```sql
SELECT
    lower(email) AS normalized_email
FROM users;
```

---

# 20. Regex Analysis

Athena can be useful for parsing logs.

Example:

```sql
SELECT
    regexp_extract(request, 'GET ([^ ]+)', 1) AS path
FROM access_logs;
```

Regex is powerful but can be expensive on very large datasets.

For repeated analytics, consider transforming raw data into structured columns.

---

# 21. JSON Functions

For JSON stored inside a column, functions can extract fields.

Conceptually:

```text
JSON document
      |
      v
Extract field
      |
      v
Structured value
```

Example patterns include JSON extraction functions supported by Athena.

For heavily queried fields, consider transforming the raw JSON into structured Parquet columns.

---

# 22. Joins

Example:

```sql
SELECT
    o.order_id,
    u.name,
    o.amount
FROM orders o
JOIN users u
    ON o.user_id = u.user_id;
```

Join performance depends heavily on:

```text
Data size
Join keys
Filtering
File format
Partitioning
Data distribution
```

---

# 23. Filter Before Join

Instead of:

```sql
SELECT *
FROM huge_orders o
JOIN users u
    ON o.user_id = u.user_id;
```

consider filtering early when appropriate:

```sql
SELECT
    o.order_id,
    u.name
FROM (
    SELECT *
    FROM orders
    WHERE year = 2026
      AND month = 9
) o
JOIN users u
    ON o.user_id = u.user_id;
```

The objective is:

```text
Reduce data
      ↓
Then join
```

---

# 24. Join Explosion

A bad join can produce far more rows than expected.

Example:

```text
Table A = 1 million rows
Table B = 1 million rows
```

If the join condition is incorrect, the result can become enormous.

Always verify:

```text
Join key
Cardinality
Duplicate keys
Filters
```

before running expensive production queries.

---

# 25. SELECT * Problem

Avoid:

```sql
SELECT *
FROM very_large_table;
```

when you only need:

```text
3 columns
```

Prefer:

```sql
SELECT
    user_id,
    status,
    event_time
FROM very_large_table;
```

This can reduce unnecessary data processing, especially with columnar formats.

---

# 26. Partitioning

Partitioning divides data into logical groups.

Example:

```text
logs/
  year=2026/
    month=09/
      day=01/
      day=02/
      day=03/
```

A query filtering:

```sql
WHERE year = 2026
  AND month = 9
  AND day = 2
```

can potentially avoid scanning unrelated partitions.

---

# 27. Partitioning Strategy

Do not blindly partition by every column.

Good partition candidates are usually:

```text
Frequently filtered
Low/moderate cardinality
Stable
Useful for data organization
```

Common examples:

```text
year
month
day
region
environment
```

---

# 28. Bad Partitioning

Avoid partitioning on extremely high-cardinality columns such as:

```text
request_id
user_id
transaction_id
timestamp-to-second
```

This can create huge numbers of tiny partitions/files.

Example:

```text
1 million users
   ↓
1 million partitions
   ↓
Operational nightmare
```

---

# 29. Partition Projection

Partition projection allows Athena to calculate partition locations based on configured rules.

Conceptually:

```text
Partition Metadata
        |
        v
Projection Rules
        |
        v
Athena determines relevant paths
```

This can reduce the need to register very large numbers of partitions individually.

---

# 30. When Partition Projection Helps

It can be useful when:

```text
Partitions follow predictable patterns
+
There are many partitions
+
You have well-defined partition ranges
```

Example:

```text
year=2025
year=2026
year=2027
```

or:

```text
date=2026-09-01
date=2026-09-02
...
```

---

# 31. Small Files Problem

Suppose:

```text
1 TB data
```

is stored as:

```text
10 million files
```

Athena may spend substantial effort dealing with file and metadata overhead.

Compare:

```text
10 million tiny files
        vs
reasonably sized larger files
```

The second layout is generally much healthier for analytical workloads.

---

# 32. Why Small Files Hurt

Small files can cause:

```text
More object operations
More metadata overhead
More task scheduling
Poor throughput
Slower queries
Higher operational complexity
```

The exact impact depends on workload and file sizes.

---

# 33. Data Compaction

Compaction combines many small files into fewer larger files.

Example:

```text
Before:

file1.parquet
file2.parquet
file3.parquet
...
file1000000.parquet

          ↓

      Compaction

          ↓

After:

part-0001.parquet
part-0002.parquet
part-0003.parquet
```

Compaction can be performed using appropriate AWS analytics/data-processing services or pipelines.

---

# 34. Parquet

For large analytical workloads, Parquet is commonly preferred over CSV.

CSV:

```text
Row-oriented
Large
Weak typing
More data scanned
```

Parquet:

```text
Columnar
Compressed
Typed
Column pruning
Better analytical performance
```

---

# 35. Compression

Common compression codecs depend on the file format and workload.

For Parquet, commonly encountered codecs include:

```text
Snappy
GZIP
ZSTD
```

The best choice depends on:

```text
Compression ratio
CPU cost
Query workload
Data characteristics
```

Do not assume the most compressed option is always the fastest.

---

# 36. Column Pruning

Suppose Parquet contains:

```text
100 columns
```

but your query needs:

```text
3 columns
```

Columnar storage can allow the engine to read only the required columns.

Therefore:

```text
Parquet
+
SELECT only needed columns
=
Efficient analytical reads
```

---

# 37. Predicate Pushdown

Filtering as early as possible can reduce unnecessary processing.

Example:

```sql
SELECT user_id
FROM logs
WHERE status_code = 500;
```

The engine may be able to apply filtering closer to the data scan depending on the format and execution plan.

This is one reason structured columnar formats are valuable.

---

# 38. Bucketing

Bucketing organizes records based on a hash of one or more columns.

Conceptually:

```text
user_id
   |
   v
Hash
   |
   +--> Bucket 1
   +--> Bucket 2
   +--> Bucket 3
   +--> Bucket 4
```

Bucketing can sometimes help specific workloads, but it should not be added automatically.

Evaluate actual query patterns before using it.

---

# 39. Partitioning vs Bucketing

| Partitioning | Bucketing |
|---|---|
| Directory-level organization | Hash-based organization |
| Excellent for common filters | Useful for certain joins/access patterns |
| Can create many directories if misused | Number of buckets must be planned |
| Often based on date | Often based on keys |
| Strong data-layout decision | More specialized optimization |

---

# 40. Query Result Reuse

Athena supports mechanisms that can avoid repeating identical work in suitable scenarios.

The general idea is:

```text
Same query
   |
   v
Existing valid result
   |
   v
Reuse result
```

This can improve latency and reduce unnecessary query processing.

Always check current Athena capabilities and workgroup configuration when implementing result reuse.

---

# 41. Materialized Views

A materialized view stores a precomputed representation of query results for supported use cases.

Conceptually:

```text
Raw Data
   |
   v
Complex Query
   |
   v
Materialized View
   |
   v
Fast Reporting Query
```

This is useful for repeatedly requested aggregations.

---

# 42. Materialized View Example

Raw table:

```text
billions of events
```

Common query:

```text
Daily request count by application
```

Instead of repeatedly calculating everything:

```text
Billions of rows
      |
      v
Aggregation
      |
      v
Smaller materialized result
```

Dashboards can then query the smaller result where supported.

---

# 43. Workgroups for DevOps

A practical environment might use:

```text
athena-dev
athena-test
athena-prod
security-analytics
finance-analytics
```

This provides separation between workloads.

---

# 44. Workgroup Governance

Workgroups can help standardize:

```text
Query result location
Encryption
Usage controls
Query execution settings
Access policies
```

A production workgroup should be controlled rather than allowing every user to configure arbitrary settings.

---

# 45. Cost Controls

A production Athena environment should have controls around:

```text
Who can query
What data can be queried
Where results are stored
How much data can be processed
Which workgroup is used
```

Use organizational AWS governance features where appropriate.

---

# 46. Athena API

Athena provides APIs that allow automation.

A common workflow is:

```text
StartQueryExecution
       |
       v
GetQueryExecution
       |
       v
Check status
       |
       v
GetQueryResults
```

Typical query states include:

```text
QUEUED
RUNNING
SUCCEEDED
FAILED
CANCELLED
```

---

# 47. AWS CLI – Start Query

Example:

```bash
aws athena start-query-execution \
  --query-string "SELECT count(*) FROM logs" \
  --query-execution-context Database=analytics \
  --result-configuration OutputLocation=s3://company-athena-results/
```

The exact CLI parameters should be adapted to your environment.

---

# 48. AWS CLI – Check Query

First capture:

```text
QueryExecutionId
```

Then:

```bash
aws athena get-query-execution \
  --query-execution-id <QUERY_EXECUTION_ID>
```

This can be used in automation pipelines.

---

# 49. AWS CLI – Get Results

Example:

```bash
aws athena get-query-results \
  --query-execution-id <QUERY_EXECUTION_ID>
```

This is useful for:

```text
Scripts
CI/CD pipelines
Operational automation
Scheduled reporting
```

---

# 50. Athena with Python / Boto3

Typical Python workflow:

```python
import boto3

athena = boto3.client("athena")

response = athena.start_query_execution(
    QueryString="SELECT count(*) FROM logs",
    QueryExecutionContext={
        "Database": "analytics"
    },
    ResultConfiguration={
        "OutputLocation": "s3://company-athena-results/"
    }
)

query_id = response["QueryExecutionId"]

print(query_id)
```

Then poll for completion:

```python
response = athena.get_query_execution(
    QueryExecutionId=query_id
)

state = response["QueryExecution"]["Status"]["State"]

print(state)
```

Production code should implement polling, retries, timeouts, and error handling appropriately.

---

# 51. Athena in CI/CD

Athena can be integrated into data pipelines.

Example:

```text
Git Push
   |
   v
CI Pipeline
   |
   v
Deploy SQL / Infrastructure
   |
   v
Run Athena Validation Query
   |
   v
Validate Result
   |
   v
Pipeline Pass / Fail
```

This can be useful for data-quality checks.

---

# 52. Athena Data Quality Check

Example:

```sql
SELECT COUNT(*)
FROM orders
WHERE order_id IS NULL;
```

Pipeline logic:

```text
Count = 0
   |
   v
PASS

Count > 0
   |
   v
FAIL
```

This turns Athena into part of a data-validation workflow.

---

# 53. Athena and Event-Driven Architecture

A data pipeline might look like:

```text
Application
    |
    v
S3
    |
    v
Event
    |
    v
Lambda / Step Functions
    |
    v
Glue / Transformation
    |
    v
Athena
    |
    v
Validation / Reporting
```

This is useful for serverless data workflows.

---

# 54. Athena + Step Functions

Step Functions can orchestrate multi-step analytics workflows.

Example:

```text
Start
 |
 v
Validate S3 data
 |
 v
Run transformation
 |
 v
Update catalog
 |
 v
Run Athena validation
 |
 v
Publish result
 |
 v
Notify
```

Athena becomes one step in a larger workflow.

---

# 55. Athena + Lambda

Lambda can trigger Athena queries.

Example:

```text
EventBridge
    |
    v
Lambda
    |
    v
Athena StartQueryExecution
    |
    v
S3 Result
```

Useful for lightweight scheduled analytics.

Be careful with:

```text
Lambda timeout
Long-running queries
Polling
Retries
Idempotency
```

For complex workflows, consider orchestration services.

---

# 56. Athena + EventBridge

A scheduled architecture:

```text
EventBridge Schedule
        |
        v
Lambda / Step Functions
        |
        v
Athena
        |
        v
S3
        |
        v
Notification / Dashboard
```

Example use cases:

```text
Daily security report
Daily failed-request report
Weekly cost analytics
Daily data-quality validation
```

---

# 57. Athena Monitoring

Monitor:

```text
Query failures
Query duration
Data processed
Query count
Workgroup usage
S3 result growth
```

Also monitor upstream data quality and downstream consumers.

---

# 58. Athena Audit Trail

Use CloudTrail to understand API activity.

Questions:

```text
Who started a query?
Which API was called?
When was it called?
Which identity performed the action?
```

CloudTrail is especially useful for security investigations and governance.

---

# 59. Athena Cost Troubleshooting

Problem:

```text
Athena bill unexpectedly increased
```

Investigate:

```text
1. Query volume
2. Data processed per query
3. New dashboards
4. SELECT *
5. Missing partition filters
6. Changed file format
7. Small-file growth
8. Repeated expensive queries
9. New users/workloads
```

---

# 60. Cost Optimization Example

Bad:

```sql
SELECT *
FROM cloudtrail_logs;
```

Better:

```sql
SELECT
    eventtime,
    eventname,
    useridentity,
    sourceipaddress
FROM cloudtrail_logs
WHERE year = 2026
  AND month = 9;
```

Better table design:

```text
CloudTrail
   |
   v
S3
   |
   v
Partitioned Parquet
   |
   v
Athena
```

---

# 61. Production Architecture – Security Analytics

```text
                AWS Accounts
                     |
          +----------+----------+
          |          |          |
      CloudTrail    WAF       VPC Logs
          |          |          |
          +----------+----------+
                     |
                     v
                Amazon S3
                     |
              Raw / Landing
                     |
                     v
              Transformation
                     |
                     v
             Parquet / Iceberg
                     |
                     v
             Glue Data Catalog
                     |
                     v
                  Athena
                 /      \
                v        v
         Security       QuickSight
          Queries       Dashboards
```

Governance:

```text
IAM
KMS
Lake Formation
CloudTrail
Workgroups
```

---

# 62. Production Architecture – Application Analytics

```text
Application
    |
    +----> Logs
    |
    +----> Events
    |
    v
Amazon S3
    |
    v
Glue / ETL
    |
    v
Parquet
    |
    v
Glue Catalog
    |
    v
Athena
    |
    +----> QuickSight
    |
    +----> Reports
    |
    +----> Data Quality
```

---

# 63. Production Architecture – Multi-Account

Example:

```text
Account A
Production
   |
   v
S3 Data

Account B
Analytics
   |
   v
Athena

Account C
Security
   |
   v
Security Analytics
```

Important controls:

```text
IAM
S3 Bucket Policies
KMS Key Policies
Lake Formation
Cross-account permissions
```

Do not assume IAM alone is sufficient for every cross-account design.

---

# 64. DevOps Scenario – Query Suddenly Became Slow

### Problem

A query used to run in:

```text
30 seconds
```

Now:

```text
15 minutes
```

### Investigation

Check:

```text
1. Data volume increased?
2. Partition filter removed?
3. New columns/files?
4. File format changed?
5. Small files increased?
6. Join cardinality changed?
7. Table schema changed?
8. Query plan changed?
```

---

# 65. DevOps Scenario – Athena Cost Increased

### Symptoms

```text
Athena usage increased 5x
```

### Investigation

```text
Query history
    |
    v
Find top data-consuming queries
    |
    v
Identify users/workgroups
    |
    v
Check partition filters
    |
    v
Check SELECT *
    |
    v
Check dashboard refresh frequency
```

Then optimize.

---

# 66. DevOps Scenario – New Data Not Visible

Problem:

```text
New files exist in S3
but Athena does not return them.
```

Check:

```text
S3 path
Partition structure
Glue Catalog
Partition metadata
Crawler
Partition projection
Table location
```

Typical flow:

```text
New S3 Files
     |
     X
Athena cannot see them
     |
     v
Check catalog / partition metadata
```

---

# 67. DevOps Scenario – AccessDenied

Problem:

```text
Athena query
       |
       v
AccessDenied
```

Check:

```text
IAM role
S3 bucket policy
S3 object permissions
KMS key policy
Lake Formation permissions
Cross-account configuration
```

Do not immediately modify all permissions to `AdministratorAccess`.

Use least privilege.

---

# 68. DevOps Scenario – HIVE_BAD_DATA

Problem:

```text
HIVE_BAD_DATA
```

Investigation:

```text
Declared schema
      |
      v
Compare with actual file
      |
      v
Check:
- data type
- delimiter
- quotes
- malformed rows
- JSON
- SerDe
```

---

# 69. DevOps Scenario – Dashboard Is Slow

Architecture:

```text
QuickSight
    |
    v
Athena
    |
    v
Huge Raw Dataset
```

Possible solution:

```text
Raw Dataset
    |
    v
Curated Parquet
    |
    v
Pre-aggregation
    |
    v
Athena
    |
    v
QuickSight
```

Do not make dashboards repeatedly scan raw multi-terabyte data if a smaller curated dataset can answer the business question.

---

# 70. Athena Best-Practice Architecture

Recommended conceptual layout:

```text
             DATA SOURCES
                  |
                  v
             S3 RAW ZONE
                  |
                  v
        Transformation / ETL
                  |
                  v
       CURATED PARQUET / ICEBERG
                  |
                  v
          GLUE DATA CATALOG
                  |
                  v
               ATHENA
          /       |       \
         /        |        \
        v         v         v
     Reports   QuickSight  Security
```

Governance surrounds the platform:

```text
IAM
KMS
Lake Formation
CloudTrail
Workgroups
```

---

# 71. Athena Design Checklist

Before putting Athena into production:

```text
[ ] S3 architecture defined
[ ] Raw/processed/curated zones defined
[ ] File format selected
[ ] Compression selected
[ ] Partition strategy designed
[ ] Small-file strategy designed
[ ] Catalog strategy defined
[ ] IAM roles created
[ ] KMS encryption configured
[ ] Workgroups configured
[ ] Query result bucket configured
[ ] Cost controls defined
[ ] Monitoring configured
[ ] CloudTrail auditing enabled
[ ] Data-quality checks defined
[ ] Retention policies defined
[ ] Disaster/recovery requirements reviewed
```

---

# 72. Interview Question – What is a View?

A view is a saved SQL definition that provides a reusable logical query over underlying data.

---

# 73. Interview Question – What is a CTE?

A CTE, or Common Table Expression, is a named query expression defined using `WITH` that can make complex SQL easier to organize and reuse within a query.

---

# 74. Interview Question – What are Window Functions?

Window functions calculate values across a related set of rows while retaining individual rows in the result.

Examples:

```text
ROW_NUMBER
RANK
DENSE_RANK
LAG
LEAD
SUM OVER
AVG OVER
```

---

# 75. Interview Question – Why Are Partitions Important?

Partitions can allow Athena to restrict data scanning to relevant portions of an S3 dataset.

Example:

```sql
WHERE year = 2026
  AND month = 9
```

instead of scanning every historical partition.

---

# 76. Interview Question – What Is the Small-File Problem?

The small-file problem occurs when data is split across a very large number of small objects.

This can increase metadata and scheduling overhead and reduce query efficiency.

Solution:

```text
Compaction
Better file sizing
Better ingestion design
```

---

# 77. Interview Question – Parquet vs CSV?

Parquet is a columnar, typed, compressed format optimized for analytical workloads.

CSV is simpler and widely interoperable but generally less efficient for large analytical queries.

---

# 78. Interview Question – What Is Predicate Pushdown?

Predicate pushdown means applying filters as close to the data scan as possible so unnecessary records can be avoided or reduced during processing.

---

# 79. Interview Question – What Is Column Pruning?

Column pruning means reading only the columns required by a query rather than reading every column.

It is especially valuable with columnar formats such as Parquet.

---

# 80. Interview Question – What Is Partition Projection?

Partition projection lets Athena derive partition information using configured projection rules rather than requiring all partition metadata to be individually registered.

---

# 81. Interview Question – Partitioning vs Bucketing?

Partitioning organizes data into logical partitions, commonly based on frequently filtered attributes such as date.

Bucketing distributes records into a fixed number of buckets based on hashing one or more columns.

---

# 82. Interview Question – How Do You Reduce Athena Cost?

Answer:

```text
1. Use partitioning.
2. Filter partitions.
3. Use Parquet/ORC.
4. Compress data.
5. Select only required columns.
6. Avoid SELECT *.
7. Compact small files.
8. Pre-aggregate repeated workloads.
9. Use appropriate workgroups.
10. Monitor processed data.
```

---

# 83. Interview Question – How Would You Secure Athena?

Use:

```text
IAM
+
S3 bucket policies
+
KMS
+
Lake Formation where required
+
Workgroups
+
CloudTrail
+
Least privilege
```

Also secure the underlying data and query-result locations.

---

# 84. Interview Question – How Would You Monitor Athena?

Monitor:

```text
Query success/failure
Execution duration
Data processed
Workgroup usage
Query frequency
S3 query-result growth
```

Use CloudTrail for API auditing and appropriate CloudWatch/AWS service telemetry for operational monitoring.

---

# 85. Interview Question – Athena vs Glue

```text
Glue
→ Data integration / ETL / catalog

Athena
→ Serverless SQL query engine
```

They commonly work together:

```text
S3
 |
 +--> Glue
 |     |
 |     v
 |  Data Catalog
 |     |
 +-----v
     Athena
```

---

# 86. Interview Question – Athena vs EMR

```text
Athena
→ Serverless SQL analytics

EMR
→ Managed big-data cluster/platform for workloads such as Spark, Hive, and other supported frameworks
```

Use Athena when you want serverless interactive SQL without managing clusters.

Use EMR when you need broader compute/framework control or workloads that fit a cluster-based processing model.

---

# 87. Interview Question – Athena vs Redshift

Athena:

```text
S3-centric
Serverless
Ad-hoc analytics
Data-lake queries
```

Redshift:

```text
Data warehouse
Repeated analytical workloads
Managed warehouse architecture
Advanced warehouse capabilities
```

They can complement each other.

---

# 88. Interview Question – Athena vs CloudWatch Logs Insights

CloudWatch Logs Insights:

```text
Operational log investigation
CloudWatch Logs
Fast troubleshooting workflows
```

Athena:

```text
S3 data lake
Long-term analytical datasets
SQL analytics
Cross-dataset analysis
```

---

# 89. Golden Production Rules

Remember:

```text
Rule 1:
Do not treat Athena as a traditional database.

Rule 2:
S3 data layout matters.

Rule 3:
Glue Catalog contains metadata.

Rule 4:
Partition according to query patterns.

Rule 5:
Do not over-partition.

Rule 6:
Avoid millions of tiny files.

Rule 7:
Prefer columnar formats for large analytical workloads.

Rule 8:
Avoid SELECT * for expensive queries.

Rule 9:
Filter early.

Rule 10:
Monitor processed data.

Rule 11:
Use Workgroups for governance.

Rule 12:
Use least privilege.

Rule 13:
Encrypt sensitive data.

Rule 14:
Use CTAS/pre-aggregation for repeated workloads.

Rule 15:
Consider Iceberg for advanced table-management requirements.
```

---

# 90. Final Athena Architecture

```text
                         USERS
                           |
             +-------------+-------------+
             |             |             |
           CLI           Python       QuickSight
             |             |             |
             +-------------+-------------+
                           |
                           v
                     AMAZON ATHENA
                           |
            +--------------+--------------+
            |              |              |
            v              v              v
       Glue Catalog      S3 Data      Federated Sources
            |              |
            |              v
            |       +------+------+
            |       |             |
            |      Raw         Curated
            |       |             |
            |       +------+------+
            |              |
            +--------------+
                           |
                           v
                  Query Processing
                           |
             +-------------+-------------+
             |             |             |
             v             v             v
            SQL          CTAS          UNLOAD
             |             |             |
             +-------------+-------------+
                           |
                           v
                      S3 Results
```

Security and governance:

```text
IAM
 |
 +---- S3
 +---- Athena
 +---- Glue
 +---- KMS
 +---- Lake Formation

CloudTrail
    |
    v
Audit

Workgroups
    |
    v
Governance / Workload Separation
```

---

# 91. Final Revision Sheet

## Athena

```text
Serverless SQL analytics
```

## S3

```text
Primary data storage layer
```

## Glue Catalog

```text
Metadata layer
```

## Partitions

```text
Reduce unnecessary data scanning
```

## Parquet

```text
Columnar analytical format
```

## Compression

```text
Reduce storage and data processing
```

## CTAS

```text
Query → New Table + Data
```

## UNLOAD

```text
Query → Result Files
```

## View

```text
Saved SQL definition
```

## Materialized View

```text
Precomputed query result for supported workloads
```

## Workgroup

```text
Workload / governance boundary
```

## Federated Query

```text
Query supported external sources
```

## Iceberg

```text
Advanced analytical table format
```

## Lake Formation

```text
Data-lake governance
```

## KMS

```text
Encryption key management
```

## CloudTrail

```text
API auditing
```

---

# 92. The Senior DevOps Mental Model

When an interviewer gives you an Athena problem, think in this order:

```text
                    ATHENA PROBLEM
                          |
                          v
                  What data source?
                          |
                          v
                         S3?
                          |
                          v
                  What file format?
                          |
                          v
               CSV / JSON / Parquet?
                          |
                          v
                   Is it partitioned?
                          |
                          v
                Is partition filtered?
                          |
                          v
                  Are files too small?
                          |
                          v
                 Are we selecting * ?
                          |
                          v
                 Are joins efficient?
                          |
                          v
                Is data pre-aggregated?
                          |
                          v
                 Is result reusable?
                          |
                          v
                 Is access permitted?
                          |
                          v
              IAM / S3 / KMS / Lake Formation
                          |
                          v
                   Can we monitor it?
                          |
                          v
                 CloudTrail / Metrics
                          |
                          v
                  Production Solution
```

---

# 93. Final Takeaways

1. Athena is a serverless SQL query service, not a traditional database.
2. S3 is commonly the storage layer.
3. Glue Data Catalog provides metadata.
4. Partitioning is one of the most important optimization techniques.
5. Partition filters are critical for large datasets.
6. Parquet is usually preferable for large analytical workloads.
7. Compression reduces storage and unnecessary data processing.
8. Small files can seriously affect performance.
9. Compaction is important for large data lakes.
10. Avoid `SELECT *` for expensive analytical queries.
11. Filter data before expensive joins when appropriate.
12. Window functions are powerful for advanced analytics.
13. Views simplify reusable SQL.
14. CTEs improve query readability.
15. CTAS can create optimized derived datasets.
16. Materialized views can help repeated aggregation workloads where supported.
17. Workgroups help with governance and workload separation.
18. Athena APIs make automation possible.
19. Lambda, EventBridge, and Step Functions can orchestrate Athena workflows.
20. IAM, S3, KMS, and Lake Formation are important security components.
21. CloudTrail helps audit Athena-related API activity.
22. Iceberg is useful for advanced analytical table-management requirements.
23. Good Athena performance starts with good data-lake design.
24. Good Athena cost control starts with reducing unnecessary data processed.
25. For senior DevOps roles, think beyond SQL: **storage + metadata + security + performance + cost + automation + observability**.

---

# End of Part 4
