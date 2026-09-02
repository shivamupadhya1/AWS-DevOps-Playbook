# Amazon Athena – Part 2

> AWS DevOps Playbook  
> Volume 1 – Data, Analytics & Observability  
> Chapter 43  
> Amazon Athena – Part 2 (Partitions, Partition Projection, Compression, CTAS, Views & Query Optimization)

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Understand Athena partitions
- Understand why partitions are important
- Create partitioned tables
- Load partitions
- Query partitioned data
- Understand partition pruning
- Understand partition projection
- Understand Parquet and ORC optimization
- Understand compression
- Understand CTAS
- Understand Athena views
- Optimize Athena queries
- Reduce Athena query costs
- Design efficient S3 data layouts
- Troubleshoot common Athena performance problems

---

# 1. Why Athena Optimization Matters

Athena is serverless, but serverless does not mean unlimited or free.

When Athena executes a query, the amount of data that must be read can directly affect:

- Query cost
- Query execution time
- Performance
- S3 data access
- Overall analytics efficiency

Consider:

```text
1 TB of data
    ↓
Query scans 1 TB
    ↓
Higher cost
+
Longer execution
```

The goal is:

```text
1 TB Dataset
    ↓
Read only required data
    ↓
Scan much less data
    ↓
Lower Cost
+
Better Performance
```

---

# 2. The Most Important Athena Optimization

One of the most important Athena concepts is:

> **Partition your data based on common query filters.**

Example:

```text
S3

logs/

├── year=2026/
│   ├── month=01/
│   ├── month=02/
│   └── month=03/
│
└── year=2025/
```

Instead of scanning everything, Athena can read only the relevant partitions.

---

# 3. What is a Partition?

A partition is a way of organizing data into separate sections based on one or more column values.

Common partition columns:

- year
- month
- day
- region
- environment
- application
- customer
- country

Example:

```text
year=2026
month=09
day=02
```

---

# 4. Example Without Partitioning

Suppose your S3 bucket contains:

```text
s3://company-logs/

├── logs-2026-01.csv
├── logs-2026-02.csv
├── logs-2026-03.csv
├── logs-2026-04.csv
├── logs-2026-05.csv
├── logs-2026-06.csv
├── logs-2026-07.csv
├── logs-2026-08.csv
└── logs-2026-09.csv
```

You want:

```text
September 2026 logs
```

Athena may need to inspect a large amount of data.

---

# 5. Example With Partitions

Instead:

```text
s3://company-logs/

year=2026/
│
├── month=01/
├── month=02/
├── month=03/
├── month=04/
├── month=05/
├── month=06/
├── month=07/
├── month=08/
└── month=09/
```

Query:

```sql
SELECT *
FROM logs
WHERE year = 2026
AND month = 9;
```

Athena can identify the relevant partition.

Conceptually:

```text
All Data
   |
   +-- Jan
   +-- Feb
   +-- Mar
   +-- ...
   +-- Sep  ← Read
   +-- ...
```

---

# 6. Partition Pruning

The process of eliminating partitions that don't need to be scanned is commonly called:

> **Partition pruning**

Example:

```sql
SELECT *
FROM logs
WHERE year = 2026
AND month = 9;
```

Athena can avoid scanning unrelated partitions.

Architecture:

```text
S3
 |
 +-- year=2025
 |
 +-- year=2026
       |
       +-- month=08
       |
       +-- month=09  ← Scan
       |
       +-- month=10
```

---

# 7. Why Partition Pruning Saves Money

Without partition pruning:

```text
1 TB
 ↓
Scan
 ↓
Higher cost
```

With effective partition pruning:

```text
1 TB Total
 ↓
Relevant partition = 50 GB
 ↓
Scan 50 GB
 ↓
Lower cost
```

The exact amount scanned depends on table format, file layout, query predicates, and other factors.

---

# 8. Partition Columns

Example table:

```text
logs
```

Columns:

```text
timestamp
client_ip
request
status_code
year
month
day
```

Partition columns:

```text
year
month
day
```

S3 layout:

```text
logs/

year=2026/
    month=09/
        day=01/
        day=02/
        day=03/
```

---

# 9. Creating a Partitioned Table

Example:

```sql
CREATE EXTERNAL TABLE logs (
    timestamp STRING,
    client_ip STRING,
    request STRING,
    status_code INT
)
PARTITIONED BY (
    year INT,
    month INT,
    day INT
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
LOCATION 's3://company-logs/';
```

Important:

```text
PARTITIONED BY
```

defines the partition columns.

---

# 10. Adding a Partition

Example:

```sql
ALTER TABLE logs
ADD PARTITION (
    year = 2026,
    month = 9,
    day = 2
)
LOCATION 's3://company-logs/year=2026/month=9/day=2/';
```

Now Athena knows where that partition exists.

---

# 11. Querying a Partition

Example:

```sql
SELECT *
FROM logs
WHERE year = 2026
AND month = 9
AND day = 2;
```

This is much better than querying the entire dataset.

---

# 12. Why Partition Filters Matter

Bad query:

```sql
SELECT *
FROM logs;
```

Potentially scans a large amount of data.

Better:

```sql
SELECT *
FROM logs
WHERE year = 2026
AND month = 9;
```

Even better when appropriate:

```sql
SELECT *
FROM logs
WHERE year = 2026
AND month = 9
AND day = 2;
```

The more precisely your partition predicate identifies relevant partitions, the more unnecessary data Athena can avoid scanning.

---

# 13. Important Rule

Do not blindly partition by every column.

For example:

```text
Bad:

country
user_id
request_id
timestamp
IP
```

Creating huge numbers of tiny partitions can create operational overhead and poor performance.

Choose partition columns based on:

> **How the data is commonly queried.**

---

# 14. High Cardinality Partition Problem

Suppose you partition by:

```text
user_id
```

And you have:

```text
10 million users
```

You could end up with an enormous number of partitions.

This is generally a bad design.

Instead, consider lower-cardinality columns such as:

```text
year
month
day
region
environment
```

depending on the workload.

---

# 15. Partition Size

You should also think about partition size.

Very small partitions can result in:

```text
Thousands / millions of small files
        ↓
More metadata
        ↓
More overhead
        ↓
Poor performance
```

Very large partitions may reduce the benefit of partition pruning.

The correct size depends on:

- Data volume
- Query patterns
- File format
- Number of files
- Query frequency

---

# 16. Partitioning Strategy Example

For application logs:

```text
s3://logs/

year=2026/
month=09/
day=02/
```

For multi-region environments:

```text
s3://logs/

region=ap-south-1/
year=2026/
month=09/
day=02/
```

For environments:

```text
s3://logs/

environment=prod/
year=2026/
month=09/
day=02/
```

Choose the structure according to how users query the data.

---

# 17. Partition Projection

Partition management can become difficult when you have a large number of partitions.

Example:

```text
Years
 ↓
Months
 ↓
Days
 ↓
Hours
```

Thousands or millions of partitions can exist.

Athena provides:

> **Partition projection**

Partition projection allows Athena to determine partition information from table properties instead of requiring every partition to be explicitly registered in the catalog.

---

# 18. Why Partition Projection Helps

Traditional approach:

```text
S3 Data
 ↓
Create partition
 ↓
Register partition
 ↓
Glue Catalog
 ↓
Athena
```

With partition projection:

```text
S3 Data
 ↓
Projection Rules
 ↓
Athena calculates partitions
 ↓
Query
```

This can reduce partition-management overhead for suitable datasets.

---

# 19. Partition Projection Example

Suppose your S3 structure is:

```text
s3://company-logs/

year=2025/
year=2026/
```

Instead of manually adding every partition, projection can define:

```text
year range:
2020 → 2030
```

Athena can use this information when evaluating queries.

---

# 20. Partition Projection Concepts

Projection commonly defines:

- Projection type
- Range
- Format
- Interval
- Storage location template

Example concepts:

```text
year
 ↓
range
 ↓
2020,2030
```

or:

```text
month
 ↓
range
 ↓
1,12
```

---

# 21. When to Use Partition Projection

Partition projection can be useful when:

- There are many partitions
- Partition values follow predictable patterns
- Data is regularly added
- Partition registration becomes expensive
- The S3 directory structure follows predictable rules

---

# 22. When Not to Use Partition Projection

It may not be appropriate when:

- Partition values are irregular
- Data layout is unpredictable
- You have only a small number of partitions
- Existing partition metadata is simpler to manage

Use it when it actually simplifies the workload.

---

# 23. File Formats

Athena can query multiple file formats.

Common examples:

```text
CSV
JSON
Parquet
ORC
Avro
```

For large analytical workloads:

```text
Parquet
ORC
```

are often preferred.

---

# 24. Row-Based vs Columnar Storage

CSV is essentially row-oriented.

Example:

```text
ID | Name | Country | Age
1  | A    | India   | 30
2  | B    | USA     | 35
3  | C    | India   | 28
```

Columnar format:

```text
ID column
Name column
Country column
Age column
```

If you only need:

```text
Country
```

a columnar format can allow the engine to read only the necessary column data.

---

# 25. Parquet

Parquet is a popular columnar file format.

Benefits:

- Columnar storage
- Compression support
- Efficient analytics
- Reduced data scanned
- Good integration with Athena
- Good performance for analytical queries

---

# 26. ORC

ORC stands for:

> Optimized Row Columnar

It is another columnar storage format commonly used in big-data systems.

It provides:

- Compression
- Columnar storage
- Predicate pushdown
- Efficient analytical processing

---

# 27. CSV vs Parquet

| CSV | Parquet |
|---|---|
| Text format | Columnar binary format |
| Easy to read | Optimized for analytics |
| Larger files | Usually smaller |
| Limited type information | Stores schema/type information |
| Less efficient for large analytics | Highly efficient for analytics |
| Good for simple data exchange | Good for analytical workloads |

---

# 28. Compression

Compression reduces the physical size of files.

Example:

```text
Uncompressed

1 TB

↓

Compression

250 GB
```

The actual compression ratio depends heavily on the data.

Benefits:

- Less storage
- Less data transferred
- Less data scanned
- Potentially lower Athena query cost
- Faster query execution

---

# 29. Common Compression Formats

Depending on the file format, you may encounter:

- GZIP
- Snappy
- ZSTD
- LZ4

For analytical workloads, the appropriate compression codec depends on:

- Compression ratio
- CPU requirements
- File format
- Query workload

---

# 30. GZIP

GZIP provides strong compression.

However, compressed files may not always provide the same level of parallel processing benefits as splittable formats.

Therefore, simply compressing files is not enough.

You should consider:

```text
File Format
+
Compression
+
File Size
+
Query Pattern
```

together.

---

# 31. Snappy

Snappy is designed for:

- Fast compression
- Fast decompression
- Analytics workloads

It is commonly used with formats such as Parquet.

---

# 32. ZSTD

Zstandard, commonly called ZSTD, provides strong compression with good performance.

It can be useful when you want a balance between:

```text
Compression Ratio
        +
Performance
```

---

# 33. Small File Problem

One of the biggest data-lake problems is:

> **Too many small files.**

Example:

```text
1 TB data

Instead of:

1000 × 1 GB files

you have:

10,000,000 × 100 KB files
```

This can create significant overhead.

---

# 34. Why Small Files Are Bad

Small files can cause:

```text
Many Files
   ↓
More Metadata
   ↓
More File Operations
   ↓
More Overhead
   ↓
Poor Query Performance
```

Therefore:

> Prefer appropriately sized files rather than millions of tiny files.

---

# 35. Compaction

Compaction means combining many small files into larger files.

Example:

```text
Before:

file1.parquet
file2.parquet
file3.parquet
...
file100000.parquet

↓

Compaction

↓

larger-file-001.parquet
larger-file-002.parquet
larger-file-003.parquet
```

This can improve query efficiency.

---

# 36. CTAS

CTAS means:

> **CREATE TABLE AS SELECT**

It allows you to create a new table from the results of a query.

Example:

```sql
CREATE TABLE users_parquet
WITH (
    format = 'PARQUET',
    external_location = 's3://company-data/processed/users/'
)
AS
SELECT *
FROM users;
```

---

# 37. Why CTAS is Useful

Suppose your source data is:

```text
CSV
```

You can convert it into:

```text
Parquet
```

using CTAS.

Architecture:

```text
CSV
 ↓
Athena
 ↓
CTAS
 ↓
Parquet
 ↓
S3
```

---

# 38. CTAS Real-World Example

Raw data:

```text
s3://company-data/raw/
```

Processed data:

```text
s3://company-data/processed/
```

Flow:

```text
Raw CSV
   ↓
Athena CTAS
   ↓
Parquet
   ↓
Compressed
   ↓
Partitioned
   ↓
S3
```

This is a common data-lake optimization pattern.

---

# 39. CTAS With Partitioning

Example:

```sql
CREATE TABLE processed_logs
WITH (
    format = 'PARQUET',
    partitioned_by = ARRAY['year', 'month'],
    external_location = 's3://company-data/processed/'
)
AS
SELECT
    timestamp,
    client_ip,
    status_code,
    year,
    month
FROM raw_logs;
```

This can create an optimized analytical dataset.

---

# 40. Athena Views

A view is a saved SQL query.

Example:

```sql
CREATE VIEW successful_requests AS
SELECT *
FROM logs
WHERE status_code BETWEEN 200 AND 299;
```

Now you can query:

```sql
SELECT *
FROM successful_requests;
```

---

# 41. Why Use Views?

Views can simplify:

- Repeated queries
- Complex SQL
- Reporting
- Data access
- QuickSight integration

Instead of repeatedly writing complex SQL, you can create a view.

---

# 42. View Architecture

```text
Raw Data
   ↓
Athena Table
   ↓
View
   ↓
Analytics
   ↓
QuickSight
```

---

# 43. Important View Concept

A normal Athena view does not mean that the complete query result is physically stored as a new dataset.

Conceptually:

```text
View
 ↓
Saved Query Definition
 ↓
Query Executes Against Underlying Data
```

Therefore, views should not automatically be treated as materialized data.

---

# 44. Query Optimization

Consider:

```sql
SELECT *
FROM logs;
```

This may scan a large amount of data.

Instead:

```sql
SELECT client_ip,
       status_code
FROM logs
WHERE year = 2026
AND month = 9;
```

This is generally better because:

- Fewer columns
- Partition filtering
- Less unnecessary data

---

# 45. Avoid SELECT *

Prefer:

```sql
SELECT client_ip,
       status_code,
       request_time
FROM logs;
```

instead of:

```sql
SELECT *
FROM logs;
```

This is particularly important for large datasets.

---

# 46. Filter Early

Bad:

```sql
SELECT *
FROM logs;
```

Better:

```sql
SELECT *
FROM logs
WHERE status_code = 500;
```

Even better when partition columns are available:

```sql
SELECT *
FROM logs
WHERE year = 2026
AND month = 9
AND status_code = 500;
```

---

# 47. Partition Predicate

Always try to include partition columns when applicable.

Example:

```sql
WHERE year = 2026
AND month = 9
AND day = 2
```

This allows Athena to eliminate irrelevant partitions.

---

# 48. Data Type Selection

Choose appropriate data types.

Example:

```text
Age → INT
Status Code → INT
Timestamp → TIMESTAMP
Name → STRING
```

Avoid treating everything as:

```text
STRING
```

Proper data types improve correctness and can improve query efficiency.

---

# 49. Avoid Unnecessary Functions

Be careful with functions applied to columns used in filtering.

For example:

```sql
WHERE year(timestamp) = 2026
```

may be less effective than using partition columns directly when those partitions exist.

Prefer:

```sql
WHERE year = 2026
```

when `year` is an actual partition column.

---

# 50. Query Result Reuse

Athena supports mechanisms that can help avoid repeatedly executing identical queries, depending on configuration and query characteristics.

This can help:

```text
Repeated Query
      ↓
Reuse Result
      ↓
Less Processing
```

Always verify current Athena query-result reuse behavior and limitations for your workload.

---

# 51. Workgroups and Cost Control

Workgroups can help organizations manage Athena usage.

Example:

```text
Athena

├── Development
├── Testing
└── Production
```

You can configure settings such as:

- Query result location
- Encryption
- Usage controls
- Query-related settings

This provides better operational governance.

---

# 52. Bytes Scanned

One of the most important Athena metrics is:

> **Data scanned**

Example:

```text
Query 1 → 5 GB scanned
Query 2 → 500 GB scanned
```

The second query is much more expensive under scan-based pricing.

Always inspect query execution statistics.

---

# 53. Query Execution Plan

Use:

```sql
EXPLAIN
```

to inspect the logical/physical query plan supported by Athena.

Example:

```sql
EXPLAIN
SELECT *
FROM logs
WHERE year = 2026
AND month = 9;
```

This can help understand how Athena plans to execute the query.

---

# 54. Athena Query Optimization Checklist

Before running a large query, ask:

```text
1. Am I selecting only required columns?

2. Am I filtering partitions?

3. Is the data stored in Parquet or ORC?

4. Is the data compressed?

5. Are there too many small files?

6. Is the table correctly partitioned?

7. Can CTAS create a better analytical dataset?

8. Can I use a view?

9. Is the query scanning unnecessary data?

10. Is the query running in the correct Workgroup?
```

---

# 55. Real-World Architecture

A production data pipeline may look like:

```text
                Application
                     |
                     v
                   Logs
                     |
                     v
                Amazon S3
                  Raw Data
                     |
                     v
                  Athena
                  /                      /                     CTAS      SQL
               |          |
               v          v
          Parquet       Analysis
               |
               v
        Partitioned Data
               |
               v
            QuickSight
               |
               v
            Dashboard
```

---

# 56. DevOps Example – ALB Logs

Suppose ALB logs arrive in S3.

Raw:

```text
s3://company-logs/alb/raw/
```

You create:

```text
Athena Table
```

Then optimize:

```text
Raw CSV
   ↓
CTAS
   ↓
Parquet
   ↓
Partition by year/month/day
   ↓
S3
```

Then:

```text
Athena
   ↓
QuickSight
```

---

# 57. DevOps Example – CloudTrail

Architecture:

```text
CloudTrail
     ↓
S3
     ↓
Partitioned Dataset
     ↓
Athena
     ↓
SQL
```

Example query:

```sql
SELECT
    eventname,
    COUNT(*) AS event_count
FROM cloudtrail_logs
WHERE year = 2026
AND month = 9
GROUP BY eventname
ORDER BY event_count DESC;
```

---

# 58. DevOps Example – VPC Flow Logs

Architecture:

```text
VPC
 ↓
VPC Flow Logs
 ↓
S3
 ↓
Athena
```

You can analyze:

```text
Top source IPs
Top destination IPs
Rejected connections
Top ports
Traffic volume
```

Example:

```sql
SELECT
    srcaddr,
    COUNT(*) AS rejected_connections
FROM vpc_flow_logs
WHERE action = 'REJECT'
AND year = 2026
AND month = 9
GROUP BY srcaddr
ORDER BY rejected_connections DESC;
```

---

# 59. Athena Cost Optimization Strategy

A strong strategy is:

```text
Raw Data
   ↓
S3
   ↓
Partition
   ↓
Convert to Parquet
   ↓
Compress
   ↓
Compact Small Files
   ↓
Query Only Required Columns
   ↓
Filter Partitions
   ↓
Lower Data Scanned
```

---

# 60. Golden Rule

Remember:

> **The best Athena query is often the query that reads the least amount of unnecessary data.**

Think:

```text
Less Data Scanned
        ↓
Lower Cost
        +
Better Performance
```

---

# 61. Common Mistakes

## Mistake 1

Partitioning by extremely high-cardinality columns.

Example:

```text
user_id
request_id
transaction_id
```

## Mistake 2

Creating millions of tiny files.

## Mistake 3

Using CSV for every large analytics workload.

## Mistake 4

Running:

```sql
SELECT *
```

on huge datasets unnecessarily.

## Mistake 5

Ignoring partition filters.

## Mistake 6

Creating too many partitions without considering query patterns.

## Mistake 7

Ignoring compression.

## Mistake 8

Not monitoring data scanned.

---

# 62. Interview Questions

## Question 1

### What is partitioning in Athena?

Partitioning organizes data into separate S3 paths based on partition columns so Athena can avoid scanning irrelevant data.

---

## Question 2

### What is partition pruning?

Partition pruning is the process of eliminating partitions that do not satisfy the query's partition predicates.

---

## Question 3

### Why is partitioning important?

It can significantly reduce the amount of data scanned, improving query performance and reducing cost.

---

## Question 4

### Should you partition by user ID?

Usually not when the user ID has extremely high cardinality.

This can create an excessive number of partitions.

---

## Question 5

### What is partition projection?

Partition projection allows Athena to derive partition information from configured rules instead of requiring every partition to be explicitly registered in the catalog.

---

## Question 6

### What is CTAS?

CTAS means:

```text
CREATE TABLE AS SELECT
```

It creates a new table from the result of a SQL query.

---

## Question 7

### Why is CTAS useful?

It can be used to transform data into a more efficient analytical format such as Parquet and can also be used to create partitioned datasets.

---

## Question 8

### Why is Parquet better than CSV for analytics?

Parquet is columnar, supports efficient compression and allows analytical engines to read only required columns.

---

## Question 9

### What is the small-file problem?

Having a very large number of tiny files can increase metadata and file-management overhead and negatively affect query performance.

---

## Question 10

### How can you reduce Athena costs?

Common techniques include:

- Partition data
- Filter partitions
- Use Parquet or ORC
- Compress data
- Avoid unnecessary columns
- Avoid `SELECT *`
- Compact small files
- Monitor data scanned
- Use appropriate Workgroups

---

## Question 11

### What is the difference between a table and a view?

A table represents a dataset definition, commonly pointing to data in S3.

A view is a saved SQL query over one or more underlying tables.

---

## Question 12

### Can Athena convert CSV to Parquet?

Yes.

One common method is using CTAS.

Example:

```text
CSV
 ↓
Athena CTAS
 ↓
Parquet
```

---

# 63. Hands-on Lab – Partitioned Dataset

Create the following structure:

```text
s3://my-athena-lab/

year=2026/
    month=09/
        day=01/
        day=02/
        day=03/
```

Create the table:

```sql
CREATE EXTERNAL TABLE logs (
    timestamp STRING,
    client_ip STRING,
    status_code INT
)
PARTITIONED BY (
    year INT,
    month INT,
    day INT
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
LOCATION 's3://my-athena-lab/';
```

---

# 64. Add Partitions

```sql
ALTER TABLE logs
ADD PARTITION (
    year = 2026,
    month = 9,
    day = 1
)
LOCATION 's3://my-athena-lab/year=2026/month=09/day=01/';
```

Add another:

```sql
ALTER TABLE logs
ADD PARTITION (
    year = 2026,
    month = 9,
    day = 2
)
LOCATION 's3://my-athena-lab/year=2026/month=09/day=02/';
```

---

# 65. Query Partition

```sql
SELECT *
FROM logs
WHERE year = 2026
AND month = 9
AND day = 2;
```

Observe the amount of data scanned.

---

# 66. Compare With Full Scan

Run:

```sql
SELECT *
FROM logs;
```

Then compare the query execution statistics.

The objective is to understand:

```text
Partition Filter
      ↓
Less Data Scanned
      ↓
Lower Cost
```

---

# 67. Hands-on Lab – CTAS

Create a Parquet table:

```sql
CREATE TABLE logs_parquet
WITH (
    format = 'PARQUET',
    external_location = 's3://my-athena-lab/processed/'
)
AS
SELECT *
FROM logs;
```

Now compare:

```text
CSV Dataset
vs
Parquet Dataset
```

---

# 68. Hands-on Lab – View

Create a view:

```sql
CREATE VIEW server_errors AS
SELECT *
FROM logs
WHERE status_code >= 500;
```

Query:

```sql
SELECT *
FROM server_errors;
```

---

# 69. Hands-on Lab – Query Optimization

Start with:

```sql
SELECT *
FROM logs;
```

Then optimize:

```sql
SELECT client_ip,
       status_code
FROM logs
WHERE year = 2026
AND month = 9
AND day = 2;
```

Compare:

```text
Data Scanned
Query Duration
Query Cost
```

---

# 70. Production Best Practices

For large Athena environments:

```text
Raw S3 Data
      ↓
Partition
      ↓
Convert to Parquet/ORC
      ↓
Compress
      ↓
Compact Small Files
      ↓
Glue Data Catalog
      ↓
Athena
      ↓
Workgroups
      ↓
Monitoring
```

Use partition columns that match your common query patterns.

Avoid excessive partition cardinality.

Monitor query performance and scanned bytes.

---

# 71. One-Page Revision

```text
ATHENA OPTIMIZATION

                Athena
                   |
          +--------+--------+
          |        |        |
          v        v        v
      Partition  Parquet  Compression
          |        |        |
          +--------+--------+
                   |
                   v
             Less Data Read
                   |
                   v
             Lower Cost
                   |
                   v
          Better Performance
```

Remember:

```text
Partitioning
     ↓
Partition Pruning
     ↓
Less Data Scanned
     ↓
Lower Cost
```

And:

```text
CSV
 ↓
CTAS
 ↓
Parquet
 ↓
Compression
 ↓
Partitioning
 ↓
Athena
```

---

# 72. Senior DevOps Mental Model

When designing an Athena-based analytics system, don't just ask:

> "Can Athena query this data?"

Ask:

```text
Where is the data?

↓

How is it partitioned?

↓

What file format is being used?

↓

Is it compressed?

↓

Are there too many small files?

↓

What columns are queried most often?

↓

Can CTAS optimize the dataset?

↓

Can partition projection simplify management?

↓

How much data does the query scan?

↓

Can QuickSight consume the result?
```

A good architecture looks like:

```text
             AWS Services
                  |
                  v
                Logs
                  |
                  v
             Amazon S3
                  |
            Raw Dataset
                  |
                  v
             ETL / CTAS
                  |
                  v
       Parquet + Compression
                  |
                  v
             Partitioning
                  |
                  v
        Glue Data Catalog
                  |
                  v
              Athena
                  |
          +-------+-------+
          |               |
          v               v
       SQL Query      QuickSight
                          |
                          v
                      Dashboard
```

---

# Key Takeaways

1. **Partitioning is one of the most important Athena optimization techniques.**
2. **Partition pruning reduces unnecessary data scanning.**
3. **Choose partitions based on actual query patterns.**
4. **Avoid extremely high-cardinality partition columns.**
5. **Partition projection can reduce partition-management overhead for suitable predictable datasets.**
6. **Parquet and ORC are generally better suited to large analytical workloads than raw CSV.**
7. **Compression can reduce storage and data-read requirements.**
8. **Too many small files can hurt performance.**
9. **CTAS can transform raw data into optimized analytical datasets.**
10. **Views are saved SQL definitions over underlying data.**
11. **Avoid unnecessary `SELECT *`.**
12. **Always use partition filters when applicable.**
13. **Monitor the amount of data scanned.**
14. **Less unnecessary data scanned generally means better performance and lower scan-based query cost.**
15. **Athena + S3 + Glue Data Catalog + QuickSight is a powerful serverless analytics architecture.**

---

# End of Part 2
