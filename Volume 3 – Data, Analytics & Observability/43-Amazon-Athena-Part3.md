# 43-Amazon-Athena-Part3

> AWS DevOps Playbook  
> Volume 1 – Data, Analytics & Observability  
> Chapter 43  
> Amazon Athena – Part 3: Advanced Athena, Glue Catalog, Security, Federated Queries, Iceberg & Production Troubleshooting

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Understand Athena and AWS Glue Data Catalog integration
- Understand schema-on-read
- Work with JSON and nested data
- Understand SerDe
- Understand schema evolution
- Understand CTAS and UNLOAD
- Understand Athena Workgroups
- Secure Athena using IAM
- Understand encryption
- Understand Lake Formation integration
- Understand Athena federated queries
- Understand Apache Iceberg with Athena
- Troubleshoot common Athena errors
- Design production-grade Athena architectures
- Answer common Athena interview questions

---

# 1. Athena and AWS Glue Data Catalog

Amazon Athena is a serverless interactive query service. It commonly queries data stored in Amazon S3 and uses the AWS Glue Data Catalog to understand the structure of that data.

Architecture:

```text
Amazon S3
   |
   | Actual Data
   v
Glue Data Catalog
   |
   | Metadata
   v
Amazon Athena
   |
   v
SQL Query
```

The Glue Data Catalog can contain:

- Database names
- Table names
- Column names
- Data types
- Partition information
- S3 locations
- Table properties
- SerDe configuration

The important distinction is:

```text
S3              → Data
Glue Catalog    → Metadata
Athena          → Query Engine
```

---

# 2. Metadata vs Data

Suppose:

```text
s3://company-data/logs/
```

contains:

```text
2026-09-01.csv
2026-09-02.csv
2026-09-03.csv
```

The files remain in S3.

The Glue Data Catalog can store:

```text
Database: analytics
Table: logs

Columns:
timestamp
client_ip
status_code
request
```

Athena uses this metadata to interpret the S3 objects.

---

# 3. Schema-on-Read

Athena is commonly used with a **schema-on-read** approach.

Conceptually:

```text
Raw Data
   |
   v
Amazon S3
   |
   v
Define/Discover Schema
   |
   v
Glue Catalog
   |
   v
Athena
   |
   v
SQL Query
```

The data does not have to be loaded into a traditional relational database before it can be queried.

This is one reason Athena works well with data lakes.

---

# 4. Schema-on-Write vs Schema-on-Read

| Schema-on-Write | Schema-on-Read |
|---|---|
| Schema established before storing/loading data | Schema applied when reading |
| Common in traditional databases | Common in data lakes |
| More rigid | More flexible |
| Data often transformed first | Raw data can remain in S3 |
| Strong structure | Useful for varied datasets |

Athena is particularly useful when organizations want to retain raw data in S3 while providing SQL access to it.

---

# 5. Athena Databases

An Athena database is primarily a logical namespace for tables.

Example:

```text
analytics
 ├── logs
 ├── users
 ├── orders
 ├── cloudtrail_logs
 └── waf_logs
```

Create a database:

```sql
CREATE DATABASE analytics;
```

Use it:

```sql
USE analytics;
```

---

# 6. Creating an External Table

Example:

```sql
CREATE EXTERNAL TABLE logs (
    timestamp STRING,
    client_ip STRING,
    status_code INT,
    request STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
LOCATION 's3://company-data/logs/';
```

The table definition points Athena to the S3 data.

The actual records remain in:

```text
Amazon S3
```

---

# 7. Schema Evolution

Data formats often change over time.

Initial schema:

```text
id
name
email
```

Later:

```text
id
name
email
department
```

This is called:

> Schema evolution

You should understand how your chosen file format, table format, and catalog handle schema changes.

---

# 8. Adding Columns

Example:

```sql
ALTER TABLE users
ADD COLUMNS (
    department STRING
);
```

This changes the table metadata.

Important:

```text
Glue Catalog
     |
     | Metadata
     v
S3 Files
```

Changing the catalog definition does not automatically rewrite existing S3 objects.

Always verify that the physical files are compatible with the new schema.

---

# 9. JSON Data in Athena

Athena can query JSON data stored in S3.

Example:

```json
{
  "id": 101,
  "name": "User1",
  "region": "ap-south-1",
  "status": "ACTIVE"
}
```

A table definition must use an appropriate JSON SerDe and schema.

Example:

```sql
CREATE EXTERNAL TABLE users_json (
    id INT,
    name STRING,
    region STRING,
    status STRING
)
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
LOCATION 's3://company-data/users/';
```

The exact SerDe and table properties should match the source JSON structure.

---

# 10. What is a SerDe?

SerDe means:

> Serializer / Deserializer

It defines how Athena interprets the physical data representation.

Conceptually:

```text
S3 File
   |
   v
SerDe
   |
   v
Athena
   |
   v
SQL Table
```

Different formats and layouts may require different SerDe configurations.

---

# 11. Nested Data

Modern applications frequently generate nested JSON.

Example:

```json
{
  "user": {
    "id": 101,
    "name": "User1"
  },
  "location": {
    "city": "Gurugram",
    "country": "India"
  }
}
```

Athena supports complex data types such as:

```text
STRUCT
ARRAY
MAP
```

---

# 12. STRUCT

A `STRUCT` represents a group of named fields.

Conceptually:

```text
user
 ├── id
 └── name
```

Example schema:

```sql
user_info STRUCT<
    id:INT,
    name:STRING
>
```

Query nested fields:

```sql
SELECT
    user_info.id,
    user_info.name
FROM users;
```

---

# 13. ARRAY

An array contains multiple values.

Example:

```json
{
  "servers": [
    "web01",
    "web02",
    "web03"
  ]
}
```

Conceptually:

```text
servers
 |
 +-- web01
 +-- web02
 +-- web03
```

Athena supports array functions and operations.

---

# 14. MAP

A map stores key-value pairs.

Example:

```text
environment → prod
team        → platform
```

Maps can be useful for flexible attributes.

---

# 15. UNNEST

`UNNEST` can expand array elements into rows.

Example:

```sql
SELECT server
FROM logs
CROSS JOIN UNNEST(servers) AS t(server);
```

Conceptually:

```text
[web01, web02, web03]

        ↓

web01
web02
web03
```

This is useful when analyzing nested application data.

---

# 16. Working With Timestamps

Use suitable timestamp types whenever possible.

Example:

```sql
timestamp TIMESTAMP
```

Query:

```sql
SELECT *
FROM logs
WHERE timestamp >= TIMESTAMP '2026-09-01 00:00:00';
```

Using an appropriate timestamp type can make filtering and time-based analysis easier.

---

# 17. Time Zones

Distributed systems frequently generate timestamps in UTC.

Example:

```text
Application → UTC
AWS Service → UTC
Athena       → Query time handling
Dashboard    → Local presentation
```

A common architecture is to store event timestamps consistently, often in UTC, and convert them for reporting when necessary.

---

# 18. CTAS

CTAS means:

```text
CREATE TABLE AS SELECT
```

It creates a new table from the results of a query.

Example:

```sql
CREATE TABLE optimized_logs
WITH (
    format = 'PARQUET',
    external_location = 's3://company-data/optimized/'
)
AS
SELECT *
FROM raw_logs;
```

CTAS is especially useful for creating optimized analytical datasets.

---

# 19. CTAS for Data-Lake Transformation

A common workflow:

```text
Raw CSV
    |
    v
S3 Raw Zone
    |
    v
Athena CTAS
    |
    v
Parquet
    |
    v
Partitioned Dataset
    |
    v
S3 Curated Zone
```

This can transform raw data into a more efficient query format.

---

# 20. CTAS for Filtering

Suppose you only need successful HTTP requests.

```sql
CREATE TABLE successful_logs
WITH (
    format = 'PARQUET',
    external_location = 's3://company-data/success/'
)
AS
SELECT *
FROM logs
WHERE status_code BETWEEN 200 AND 299;
```

The derived dataset contains only the selected records.

---

# 21. CTAS for Aggregation

Example:

```sql
CREATE TABLE daily_requests
WITH (
    format = 'PARQUET',
    external_location = 's3://company-data/daily/'
)
AS
SELECT
    year,
    month,
    day,
    COUNT(*) AS request_count
FROM logs
GROUP BY
    year,
    month,
    day;
```

This can create a much smaller analytical dataset.

---

# 22. UNLOAD

Athena also supports `UNLOAD` for writing query results to S3.

Example:

```sql
UNLOAD (
    SELECT *
    FROM logs
    WHERE status_code >= 500
)
TO 's3://company-data/export/errors/'
WITH (
    format = 'PARQUET'
);
```

The exact supported syntax and options depend on the Athena engine and current AWS capabilities.

---

# 23. CTAS vs UNLOAD

| CTAS | UNLOAD |
|---|---|
| Creates a new table | Writes query results to S3 |
| Creates table metadata | Primarily produces output files |
| Good for derived datasets | Good for exports |
| Can define table properties | Designed around query-result output |

Mental model:

```text
CTAS
Query
  ↓
Table + Data

UNLOAD
Query
  ↓
Result Files
```

---

# 24. Athena Workgroups

Workgroups allow organizations to separate Athena workloads.

Example:

```text
Athena
 |
 +-- Development
 +-- Testing
 +-- Production
 +-- Finance
 +-- Security
```

Benefits include:

- Workload separation
- Governance
- Cost management
- Query organization
- Configuration control

---

# 25. Workgroup Configuration

Depending on the configuration, workgroups can help manage:

- Query result locations
- Encryption
- Engine-related settings
- Usage controls
- Query execution behavior

Use separate workgroups when different teams or environments require different controls.

---

# 26. Athena Security Model

Production Athena security commonly involves:

```text
IAM
S3
Glue Data Catalog
Lake Formation
KMS
CloudTrail
Workgroups
```

Security is therefore a complete AWS architecture rather than only an Athena setting.

---

# 27. IAM Permissions

An Athena user or role may require access to:

```text
Athena
Glue Data Catalog
S3
KMS
Lake Formation
```

depending on the architecture.

Conceptually:

```text
IAM Role
   |
   +---- Athena
   |
   +---- Glue
   |
   +---- S3
   |
   +---- KMS
```

Follow least-privilege principles.

---

# 28. S3 Permissions

Athena needs appropriate permissions for the S3 data and query-result locations.

Common permissions may include:

```text
s3:GetObject
s3:ListBucket
```

The exact required permissions depend on the operation.

Important:

> Athena does not bypass S3 authorization.

---

# 29. Query Result Location

Athena query results are commonly written to S3.

Example:

```text
s3://company-athena-results/
```

A clean structure might be:

```text
s3://company-data/raw/
s3://company-data/processed/
s3://company-data/curated/
s3://company-athena-results/
```

Keep query results separate from source data where appropriate.

---

# 30. Encryption

Athena query results stored in S3 can be encrypted.

Common S3 server-side encryption choices include:

```text
SSE-S3
SSE-KMS
```

Choose based on organizational security requirements.

---

# 31. SSE-S3

Conceptually:

```text
Athena
  |
  v
S3
  |
  v
AWS-managed S3 encryption
```

This is simple to configure.

---

# 32. SSE-KMS

Conceptually:

```text
Athena
  |
  v
S3
  |
  v
AWS KMS
```

SSE-KMS provides greater control over encryption keys and policies.

You must correctly configure:

- IAM permissions
- KMS key policy
- S3 permissions
- Athena/workgroup settings

---

# 33. KMS AccessDenied Troubleshooting

Possible flow:

```text
Athena Query
     |
     v
S3
     |
     v
KMS
     |
     X
AccessDenied
```

Check:

```text
IAM policy
KMS key policy
S3 bucket policy
Workgroup settings
Cross-account permissions
```

---

# 34. Lake Formation

AWS Lake Formation provides centralized data-lake governance capabilities.

Conceptually:

```text
Users
  |
  v
Lake Formation
  |
  +---- S3
  |
  +---- Glue Catalog
  |
  +---- Athena
```

It can help manage access to data and metadata.

---

# 35. Fine-Grained Access

A data platform may require:

```text
Team A → Full table
Team B → Selected columns
Team C → Filtered data
```

Lake Formation can be part of the architecture used to implement more granular access control.

Always verify the exact supported permissions and features in your AWS environment.

---

# 36. Cross-Account Athena

Enterprise AWS environments may have:

```text
Account A
Data Lake

Account B
Analytics

Account C
Security
```

Cross-account analytics requires careful configuration of:

- IAM
- S3 bucket policies
- Glue Data Catalog permissions
- KMS key policies
- Lake Formation permissions where applicable

---

# 37. Athena Federated Query

Athena can access supported external data sources through federated query capabilities.

Conceptually:

```text
                Athena
                   |
        +----------+----------+
        |          |          |
        v          v          v
       S3       Database    Other Sources
```

This can provide a SQL interface over data that is not already stored in S3.

---

# 38. Federated Query Architecture

A common architecture is:

```text
Athena
  |
  v
Data Source Connector
  |
  v
Lambda
  |
  v
External Data Source
```

The exact architecture varies by connector and data source.

---

# 39. Why Use Federated Queries?

Useful when:

- Data cannot immediately be moved to S3
- You need temporary access to an operational database
- You need SQL analysis across multiple sources
- You want to combine external data with S3 datasets

However, federated queries can introduce additional latency and external-system load.

---

# 40. Federated Query Tradeoffs

Advantages:

```text
Less initial data movement
Central SQL interface
Cross-source analysis
```

Disadvantages:

```text
Additional latency
Connector management
External database load
More permissions
More troubleshooting
```

For repeated large-scale analytics, an S3-based data lake may be more appropriate.

---

# 41. Apache Iceberg

Apache Iceberg is a table format designed for large analytical datasets.

Athena supports querying and working with Iceberg tables.

Iceberg provides capabilities such as:

- Schema evolution
- Partition evolution
- Table snapshots
- Time travel
- Transactional operations for supported workloads
- Better table-level metadata management

---

# 42. Traditional Hive-Style Tables vs Iceberg

Traditional layout:

```text
S3
 |
 +-- year=2026/
 +-- year=2025/
 +-- files
```

Iceberg:

```text
S3
 |
 +-- Data Files
 +-- Metadata
 +-- Snapshots
 +-- Table State
```

Iceberg adds a table abstraction and metadata layer over object storage.

---

# 43. Iceberg Time Travel

One important Iceberg feature is:

> Time travel

Conceptually:

```text
Current Table
     |
     +---- Snapshot 3
     |
     +---- Snapshot 2
     |
     +---- Snapshot 1
```

This can be useful for:

- Auditing
- Debugging
- Historical analysis
- Investigating data changes

The exact syntax depends on the Athena engine and current Iceberg support.

---

# 44. Iceberg Schema Evolution

Iceberg is designed to support schema changes while tracking table metadata.

Example:

```text
Before:

id
name

After:

id
name
department
```

This can make evolving large analytical tables easier than managing raw file layouts manually.

---

# 45. Iceberg Partition Evolution

Iceberg supports partition evolution concepts.

For example:

```text
Initial:
partition by day

Later:
partition by month
```

This is useful when query patterns change as the dataset grows.

---

# 46. When to Consider Iceberg

Consider Iceberg when you need:

- Large analytical tables
- Frequent updates
- Deletes
- Schema evolution
- Transactional data-lake operations
- Time travel
- Advanced table management

For a simple append-only log dataset, partitioned Parquet may be sufficient.

---

# 47. Athena + QuickSight

Athena can act as a query layer for Amazon QuickSight.

Architecture:

```text
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

This is a common serverless analytics architecture.

---

# 48. QuickSight Example

Suppose you have application logs.

You can calculate:

```text
Total Requests
5xx Errors
4xx Errors
Average Response Time
Top URLs
Top Clients
```

Athena provides the SQL analysis layer and QuickSight provides visualization.

---

# 49. Athena + CloudTrail

Security analytics architecture:

```text
CloudTrail
     |
     v
S3
     |
     v
Glue Catalog
     |
     v
Athena
     |
     v
Security Analysis
```

Possible questions:

```text
Who changed a security group?
Who modified an IAM policy?
Who deleted a resource?
Which API calls failed?
```

---

# 50. Athena + VPC Flow Logs

Architecture:

```text
VPC
 |
 v
VPC Flow Logs
 |
 v
S3
 |
 v
Athena
 |
 v
Network Analysis
```

Useful analysis:

```text
Rejected traffic
Top source IPs
Top destination IPs
Top ports
Traffic volume
```

---

# 51. Athena + ALB Logs

Architecture:

```text
ALB
 |
 v
Access Logs
 |
 v
S3
 |
 v
Athena
 |
 v
Application Analytics
```

You can analyze:

```text
5xx errors
Slow requests
Top URLs
Top client IPs
Response codes
Request counts
```

---

# 52. Athena + WAF Logs

WAF logs can be stored in S3 and analyzed using Athena.

Architecture:

```text
AWS WAF
   |
   v
S3
   |
   v
Glue Catalog
   |
   v
Athena
   |
   v
Security Analytics
```

Possible analysis:

```text
Blocked requests
Allowed requests
Top attacking IPs
Rule matches
URI patterns
Countries
```

---

# 53. Athena + CloudWatch

CloudWatch and Athena solve different problems.

CloudWatch:

```text
Metrics
Logs
Alarms
Operational Monitoring
```

Athena:

```text
SQL Analysis
Historical S3 Data
Data Lake Analytics
Ad-hoc Queries
```

Combined architecture:

```text
Application
    |
    +----> CloudWatch
    |
    +----> S3
             |
             v
           Athena
             |
             v
         Analytics
```

---

# 54. Common Error – HIVE_BAD_DATA

Example:

```text
HIVE_BAD_DATA
```

This can happen when the physical data does not match the declared schema.

Example:

```text
Expected:

status_code INT

Actual:

status_code = "UNKNOWN"
```

Possible causes:

- Incorrect data type
- Corrupt records
- Invalid CSV
- Schema mismatch
- Unexpected JSON structure
- Incorrect SerDe

---

# 55. Troubleshooting HIVE_BAD_DATA

Check:

```text
1. Actual S3 file content
2. Athena table schema
3. Data types
4. Delimiters
5. Quotes
6. JSON structure
7. SerDe configuration
```

Start with a small sample of the raw data.

---

# 56. Common Error – AccessDenied

If Athena returns:

```text
AccessDenied
```

check:

```text
IAM
S3 bucket policy
KMS key policy
Lake Formation
Cross-account configuration
```

Troubleshooting flow:

```text
Athena
  |
  X
AccessDenied
  |
  +--> IAM?
  +--> S3?
  +--> KMS?
  +--> Lake Formation?
```

---

# 57. Common Error – Table Not Found

Example:

```text
TABLE_NOT_FOUND
```

Check:

```text
Database
Table name
AWS Region
Data Catalog
Workgroup
```

AWS resources and service configurations are often region-specific.

---

# 58. Common Error – Partition Not Found

A partition may physically exist in S3 while Athena does not know about it.

Example:

```text
S3
 |
 +-- New Partition
 |
 X
Catalog has no partition metadata
```

Possible approaches:

- Register the partition
- Use AWS Glue crawler where appropriate
- Use partition projection
- Use supported partition-management mechanisms

---

# 59. S3 Path Debugging

Suppose Athena expects:

```text
s3://logs/year=2026/month=09/day=02/
```

but the actual structure is:

```text
s3://logs/year=2026/month=09/02/
```

The table definition and physical S3 layout do not match.

Always inspect:

```text
Bucket
Prefix
Partition directories
File names
```

---

# 60. Debugging Athena Queries

Use this workflow:

```text
1. Check SQL
       ↓
2. Check table schema
       ↓
3. Check S3 location
       ↓
4. Inspect sample files
       ↓
5. Check partitions
       ↓
6. Check IAM
       ↓
7. Check KMS
       ↓
8. Check Lake Formation
       ↓
9. Check query statistics
       ↓
10. Check CloudTrail where appropriate
```

---

# 61. CloudTrail for Athena Troubleshooting

CloudTrail records API activity for supported AWS services.

It can help determine:

```text
Who?
When?
What API operation?
Which AWS resource?
```

This is useful when investigating:

- Permission problems
- Configuration changes
- Security events
- Administrative actions

---

# 62. Monitoring Athena

A production Athena environment should monitor:

```text
Query count
Query failures
Data scanned
Execution time
Workgroup usage
Cost
```

Also monitor related services:

```text
S3
Glue
KMS
Lake Formation
CloudTrail
```

as applicable.

---

# 63. Athena Cost Optimization

A key principle is:

> Reduce unnecessary data scanned.

Use:

```text
Partitioning
+
Partition Filtering
+
Parquet / ORC
+
Compression
+
Column Selection
+
File Compaction
```

Example:

```text
10 TB Raw Dataset
      |
      v
Partitioned Parquet
      |
      v
Filter date partition
      |
      v
Select only required columns
      |
      v
Much less data processed
```

Always verify current AWS pricing when estimating production costs.

---

# 64. Production Data-Lake Zones

A common S3 organization is:

```text
company-data/

├── raw/
├── processed/
├── curated/
└── analytics/
```

Example:

```text
raw/
   cloudtrail/
   waf/
   alb/
   vpc-flow-logs/

processed/
   parquet/

curated/
   business-datasets/

analytics/
   reporting-datasets/
```

Athena can query these datasets through the Glue Catalog.

---

# 65. Recommended Production Flow

```text
Source
  |
  v
S3 Raw
  |
  v
Validation
  |
  v
Transformation
  |
  v
Parquet / Iceberg
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
  +----> QuickSight
  |
  +----> Security Analysis
  |
  +----> Reporting
```

---

# 66. Production Security Architecture

```text
                    IAM
                     |
          +----------+----------+
          |                     |
          v                     v
      Athena               Lake Formation
          |                     |
          v                     v
         Glue <------------> Data Catalog
          |
          v
         S3
          |
          v
         KMS
```

Security controls should follow:

```text
Authentication
Authorization
Least Privilege
Encryption
Auditing
```

---

# 67. Production Athena Checklist

```text
[ ] S3 structure is correct
[ ] Partition strategy is defined
[ ] File format is appropriate
[ ] Compression is enabled
[ ] Small-file problem is addressed
[ ] Glue Catalog is configured
[ ] IAM follows least privilege
[ ] Query results are encrypted
[ ] KMS permissions are correct
[ ] Workgroups are configured
[ ] Cost controls are defined
[ ] CloudTrail auditing is enabled
[ ] Lake Formation is configured if required
[ ] QuickSight access is configured if required
[ ] Data retention is defined
```

---

# 68. Interview Question – What is Athena?

Amazon Athena is a serverless interactive query service that allows SQL analysis of data in Amazon S3 and supported external sources.

---

# 69. Interview Question – Does Athena Store Data?

Athena is primarily a query service.

The source data commonly remains in:

```text
Amazon S3
```

Athena reads and processes the data rather than requiring it to be loaded into a traditional database.

---

# 70. Interview Question – What is the Glue Data Catalog?

AWS Glue Data Catalog is a centralized metadata repository containing information such as:

```text
Databases
Tables
Schemas
Partitions
S3 Locations
Table Properties
```

Athena can use this metadata to query data in S3.

---

# 71. Interview Question – What is Schema-on-Read?

Schema-on-read means the structure of the data is applied or interpreted when the data is queried rather than requiring the data to be fully transformed into a traditional database schema before storage.

---

# 72. Interview Question – Why Use Parquet?

Parquet is a columnar format that can:

- Read only required columns
- Compress efficiently
- Reduce data processed
- Improve analytical query performance

---

# 73. Interview Question – What is CTAS?

CTAS stands for:

```text
CREATE TABLE AS SELECT
```

It creates a new table from the result of a SQL query.

---

# 74. Interview Question – CTAS vs UNLOAD?

Mental model:

```text
CTAS
Query
 ↓
Table + Data

UNLOAD
Query
 ↓
Result Files in S3
```

CTAS is useful for derived datasets, while UNLOAD is useful for exporting query results.

---

# 75. Interview Question – What is Partition Projection?

Partition projection allows Athena to derive partition information using configured projection rules rather than requiring every partition to be explicitly registered in the catalog.

---

# 76. Interview Question – Why Avoid SELECT *?

If you only need a few columns, selecting all columns can cause unnecessary data processing.

Prefer:

```sql
SELECT
    user_id,
    status,
    timestamp
FROM logs;
```

instead of:

```sql
SELECT *
FROM logs;
```

---

# 77. Interview Question – What is Federated Query?

Federated Query allows Athena to access supported external data sources through connectors, providing SQL-based analysis without first moving all data into S3.

---

# 78. Interview Question – What is Apache Iceberg?

Apache Iceberg is a table format for large analytical datasets that provides capabilities such as:

```text
Schema Evolution
Partition Evolution
Snapshots
Time Travel
Transactional Operations
```

for supported workloads.

---

# 79. Interview Question – Athena vs RDS

| Athena | RDS |
|---|---|
| Serverless query service | Managed relational database |
| Data commonly in S3 | Data stored in database |
| Analytical workloads | Transactional workloads |
| Ad-hoc analytics | OLTP applications |
| No traditional database server | Database engine configuration required |
| Excellent for data lakes | Excellent for relational applications |

---

# 80. Interview Question – Athena vs Redshift

| Athena | Redshift |
|---|---|
| Serverless query service | Cloud data warehouse |
| Excellent S3 integration | Dedicated warehouse architecture |
| Great for ad-hoc queries | Great for repeated warehouse workloads |
| Minimal infrastructure | More warehouse configuration |
| Pay based on query/data processing model | Warehouse pricing model |

They can also be used together.

---

# 81. Interview Question – Athena vs CloudWatch Logs Insights

| Athena | CloudWatch Logs Insights |
|---|---|
| SQL over S3/external sources | Query CloudWatch Logs |
| Data lake analytics | Operational log analysis |
| Long-term datasets | Near-real-time investigation |
| Broad analytical workflows | CloudWatch-centric workflows |

---

# 82. Senior DevOps Scenario – 10 TB ALB Logs

### Problem

You have:

```text
10 TB ALB logs
```

stored as:

```text
CSV
```

Queries are slow and expensive.

### Solution

Use:

```text
10 TB CSV
    |
    v
S3 Raw
    |
    v
Transformation / CTAS
    |
    v
Parquet
    |
    v
Compression
    |
    v
Partitioning
    |
    v
Glue Catalog
    |
    v
Athena
```

Then:

```text
Avoid SELECT *
+
Filter partitions
+
Select required columns
+
Compact small files
+
Monitor scanned bytes
```

---

# 83. Senior DevOps Scenario – Security Analytics

You need to analyze:

```text
CloudTrail
WAF
VPC Flow Logs
ALB Logs
```

Architecture:

```text
CloudTrail ─┐
WAF       ──┤
VPC Logs  ──┼──> S3 Raw
ALB Logs  ──┘
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
             /                  v        v
       Security   QuickSight
       Analysis   Dashboard
```

Add:

```text
IAM
Lake Formation
KMS
CloudTrail
Workgroups
```

as required by the security and governance design.

---

# 84. Senior DevOps Scenario – Query Is Expensive

Query:

```sql
SELECT *
FROM cloudtrail_logs;
```

Data scanned:

```text
500 GB
```

Optimization:

```text
1. Select only required columns
2. Filter partitions
3. Use Parquet
4. Compress data
5. Check small files
6. Review query statistics
```

Target:

```text
500 GB Raw
   |
   v
Partitioned Parquet
   |
   v
Partition Filter
   |
   v
Required Columns
   |
   v
Much less data processed
```

---

# 85. Production Troubleshooting Decision Tree

```text
Athena Query Failed
        |
        v
Is SQL valid?
   |             |
  No            Yes
   |             |
Fix SQL          v
              Table exists?
                 |
          +------+------+
          |             |
         No            Yes
          |             |
      Check DB          v
                    S3 access?
                       |
                +------+------+
                |             |
               No            Yes
                |             |
             IAM/S3          v
                          Schema valid?
                              |
                       +------+------+
                       |             |
                      No            Yes
                       |             |
                  Fix schema       v
                              Partition?
                                  |
                           +------+------+
                           |             |
                          No            Yes
                           |             |
                       Register/       v
                       Projection   Performance
```

---

# 86. Athena Golden Rules

Remember:

```text
1. Keep raw data in S3.

2. Use Glue Data Catalog for metadata.

3. Partition according to query patterns.

4. Prefer Parquet or ORC for large analytical workloads.

5. Compress data.

6. Avoid small files.

7. Avoid SELECT * when unnecessary.

8. Filter partitions.

9. Monitor data scanned.

10. Use Workgroups for governance.

11. Use IAM and Lake Formation for access control.

12. Encrypt sensitive data and query results.

13. Use CTAS for useful derived datasets.

14. Consider Iceberg for advanced table-management requirements.

15. Use federated queries when external-source access makes sense.
```

---

# 87. Complete Athena Mental Model

```text
                         AMAZON ATHENA
                               |
              +----------------+----------------+
              |                |                |
              v                v                v
             S3           Glue Catalog      External Sources
              |                |                |
              +----------------+----------------+
                               |
                               v
                         SQL Query Engine
                               |
             +-----------------+-----------------+
             |                 |                 |
             v                 v                 v
          CTAS              UNLOAD           Views
             |                 |                 |
             v                 v                 v
        Derived Data       S3 Files        Saved Queries
                               |
                               v
                         QuickSight
```

Security layer:

```text
IAM
 |
 +-- Athena
 +-- S3
 +-- Glue
 +-- KMS
 +-- Lake Formation
```

Observability layer:

```text
CloudTrail
CloudWatch
Athena Query Statistics
S3
```

---

# 88. One-Page Revision

```text
SOURCE DATA
    |
    v
Amazon S3
    |
    +-------------------+
    |                   |
    v                   v
Raw Data           Processed Data
                        |
                        v
                Parquet / Iceberg
                        |
                        v
                   Partitioning
                        |
                        v
                Glue Data Catalog
                        |
                        v
                     Athena
                  /     |                      /      |                      v       v       v
              Views    CTAS   UNLOAD
                        |
                        v
                   QuickSight
```

Optimization:

```text
Partition
   +
Parquet
   +
Compression
   +
Column Selection
   +
Partition Filtering
   +
File Compaction
   =
Better Performance
+
Lower Unnecessary Data Processing
```

---

# 89. Senior DevOps Mental Model

When designing an Athena solution, do not ask only:

> "Can Athena query this data?"

Ask:

```text
Where is the data?

↓

How is it partitioned?

↓

What file format is used?

↓

Is it compressed?

↓

Are there too many small files?

↓

What columns are queried?

↓

Can CTAS optimize the dataset?

↓

Would partition projection help?

↓

Would Iceberg be useful?

↓

How much data does the query process?

↓

Who is allowed to access it?

↓

How are query results encrypted?

↓

How is the workload monitored?

↓

Can QuickSight consume the data?
```

---

# 90. Final Key Takeaways

1. Athena is a **serverless SQL query service** commonly used with Amazon S3.
2. Glue Data Catalog stores metadata used by Athena.
3. Athena commonly follows a **schema-on-read** model.
4. JSON and nested data can be queried using suitable schemas and functions.
5. CTAS is useful for transforming raw data into optimized datasets.
6. UNLOAD is useful for exporting query results to S3.
7. Workgroups help organize and govern Athena workloads.
8. IAM, S3, KMS, and Lake Formation are important security components.
9. Federated queries can access supported external data sources.
10. Iceberg provides advanced table-management capabilities.
11. QuickSight can use Athena for dashboards and analytics.
12. CloudTrail and CloudWatch complement Athena for operational visibility.
13. Parquet, partitioning, compression, and good file layout are important performance techniques.
14. Avoid unnecessary columns and `SELECT *`.
15. Monitor data processed/scanned and query execution.
16. Always validate S3 paths and table metadata when troubleshooting.
17. Design Athena as part of a broader **S3 + Glue + Athena + governance + visualization** architecture.

---

# End of Part 3
