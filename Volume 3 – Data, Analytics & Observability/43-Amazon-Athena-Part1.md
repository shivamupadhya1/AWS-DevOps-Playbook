# Amazon Athena – Part 1

> AWS DevOps Playbook  
> Volume 1 – Data, Analytics & Observability  
> Chapter 43  
> Amazon Athena – Part 1 (Introduction, Serverless SQL, S3, Databases, Tables & Architecture)

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Understand Amazon Athena
- Understand serverless SQL analytics
- Understand how Athena works with Amazon S3
- Understand databases and tables in Athena
- Understand the AWS Glue Data Catalog
- Query CSV, JSON and Parquet data
- Understand Athena architecture
- Understand Athena pricing basics
- Identify real-world DevOps use cases
- Answer basic Athena interview questions

---

# 1. What is Amazon Athena?

Amazon Athena is a **serverless interactive query service** that allows you to analyze data stored in Amazon S3 using standard SQL.

The important point is:

> Athena does not require you to create or manage database servers.

Architecture:

```text
Amazon S3
   ↓
Amazon Athena
   ↓
SQL Query
   ↓
Query Results
2. Why Do We Need Athena?

Suppose you have several TB of logs stored in S3.

S3

├── Application Logs
├── CloudTrail Logs
├── ALB Logs
├── CloudFront Logs
└── VPC Flow Logs

You want to answer:

Which IP generated the most requests?

Traditionally, you might need:

S3
 ↓
ETL
 ↓
Database
 ↓
Query

With Athena:

S3
 ↓
Athena
 ↓
SQL
 ↓
Result

No database server is required.

3. Athena is Serverless

Athena is a serverless service.

You don't manage:

EC2
Database servers
Operating systems
Database patches
Database scaling

AWS manages the underlying infrastructure.

You mainly focus on:

Data
 ↓
SQL
 ↓
Results
4. Athena + S3

Athena is commonly used with Amazon S3.

Example:

S3 Bucket

s3://company-data/

├── logs/
├── sales/
├── customers/
└── reports/

Athena can query the data directly from S3.

5. Important Concept

Athena does not normally move your data into a traditional database before querying it.

Instead:

SQL Query
    ↓
Athena
    ↓
Read Data from S3
    ↓
Process Data
    ↓
Return Results

This is one of the key concepts to remember.

6. Athena Architecture

Basic architecture:

                   User
                    |
                    v
               SQL Query
                    |
                    v
              Amazon Athena
                    |
          +---------+---------+
          |                   |
          v                   v
   Glue Data Catalog        S3 Data
          |                   |
          |                   |
          +---------+---------+
                    |
                    v
                Query Engine
                    |
                    v
                 Results
                    |
                    v
              S3 / Console
7. AWS Glue Data Catalog

Athena needs metadata describing the data.

This metadata can be stored in the AWS Glue Data Catalog.

The catalog contains information such as:

Database name
Table name
Column names
Data types
S3 location
Partition information

Think of it as:

Actual Data

↓

S3

Metadata

↓

Glue Data Catalog
8. Data vs Metadata

This distinction is very important.

Data

Actual records:

John,India,25
Rahul,India,30
Amit,USA,35

Stored in:

Amazon S3
Metadata

Information about the data:

Column 1 → name → string
Column 2 → country → string
Column 3 → age → integer

Stored in:

Glue Data Catalog
9. Athena Database

An Athena database is a logical container for tables.

Example:

Database: production_logs

    |
    +-- alb_logs
    |
    +-- cloudtrail_logs
    |
    +-- vpc_flow_logs

The database does not mean that the data itself is stored inside Athena.

10. Athena Table

A table defines how Athena should interpret data stored in S3.

Example:

Table:

alb_logs

Schema:

request_time   timestamp
client_ip      string
request_method string
uri            string
status_code    integer

Actual data remains in S3.

11. External Table Concept

Athena commonly works with external tables.

Example:

Athena Table
     |
     | points to
     v
S3 Location

The table describes the structure of the data but does not necessarily contain the actual data.

12. Example S3 Data

Suppose S3 contains:

s3://my-company-data/users/users.csv

File:

101,Shivam,India
102,Rahul,India
103,Amit,USA

We can create an Athena table representing this data.

13. Creating a Database

Example:

CREATE DATABASE company_data;

Now:

company_data

is available as a logical database.

14. Creating a Table

Example:

CREATE EXTERNAL TABLE users (
    id INT,
    name STRING,
    country STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
LOCATION 's3://my-company-data/users/';

Important:

LOCATION

tells Athena where the actual data is stored.

15. Querying the Table

Example:

SELECT *
FROM users;

Athena reads the data from the configured S3 location.

16. Filtering Data

Example:

SELECT *
FROM users
WHERE country = 'India';

Result:

101  Shivam  India
102  Rahul   India
17. Selecting Specific Columns

Instead of:

SELECT *
FROM users;

Use:

SELECT name, country
FROM users;

This returns only the required columns.

18. Counting Records

Example:

SELECT COUNT(*)
FROM users;

Result:

3
19. Aggregation

Example:

SELECT country, COUNT(*)
FROM users
GROUP BY country;

Result:

India   2
USA     1
20. Supported Data Formats

Athena can work with many common formats.

Examples:

CSV
JSON
Apache Parquet
Apache ORC
Avro
Text formats
21. CSV

Example:

101,Shivam,India
102,Rahul,India
103,Amit,USA

CSV is easy to understand but may not be the most efficient format for large-scale analytics.

22. JSON

Example:

{
  "id": 101,
  "name": "Shivam",
  "country": "India"
}

Athena can query JSON data using appropriate table definitions.

23. Parquet

Parquet is a columnar storage format.

Example:

id | name | country | age

Instead of reading every column for a query, Athena can read only the required columns.

Example:

SELECT name
FROM users;

With columnar formats such as Parquet, Athena can avoid reading unnecessary columns.

24. Why Parquet is Important

Suppose your dataset has:

100 columns

But your query needs:

2 columns

With a columnar format:

Read only required columns

This can significantly reduce the amount of data scanned.

25. Athena Pricing Concept

Athena pricing is primarily based on the amount of data scanned by queries.

Conceptually:

More Data Scanned
       ↓
Higher Cost

Therefore:

Optimize Queries
       ↓
Reduce Data Scanned
       ↓
Reduce Cost

Always check current AWS pricing for the exact rates applicable to your region and configuration.

26. Why SELECT * Can Be Expensive

Suppose:

1 TB Dataset

You run:

SELECT *
FROM logs;

You may scan a large amount of data.

Instead:

SELECT client_ip, status_code
FROM logs;

can reduce the amount of data that needs to be read, depending on the storage format and table design.

27. Athena Query Result Location

Athena stores query results in S3.

Example:

s3://athena-results/

Architecture:

SQL Query
    ↓
Athena
    ↓
Query Execution
    ↓
Results
    ↓
S3
28. Athena Workgroup

A Workgroup is used to organize and control Athena queries.

Workgroups can help with:

Query organization
Access control
Cost controls
Query settings
Result locations

Example:

Athena

├── Dev Workgroup
├── Test Workgroup
└── Production Workgroup
29. Athena Use Cases

Athena is commonly used for:

Log Analysis
CloudTrail
    ↓
S3
    ↓
Athena
Security Analysis
VPC Flow Logs
    ↓
S3
    ↓
Athena
Application Analysis
ALB Logs
    ↓
S3
    ↓
Athena
Business Analytics
Sales Data
    ↓
S3
    ↓
Athena
30. DevOps Use Case – CloudTrail

Suppose you want to find all EC2 instances launched recently.

Architecture:

CloudTrail
     ↓
S3
     ↓
Athena
     ↓
SQL Query

Example:

SELECT eventtime,
       eventname,
       useridentity
FROM cloudtrail_logs
WHERE eventname = 'RunInstances';
31. DevOps Use Case – ALB Logs

ALB access logs:

ALB
 ↓
S3
 ↓
Athena

You can analyze:

HTTP status codes
Client IPs
URLs
Request counts
Response times
Errors
32. DevOps Use Case – VPC Flow Logs

Architecture:

VPC Flow Logs
      ↓
S3
      ↓
Athena
      ↓
SQL

You can investigate:

Source IP
Destination IP
Port
Protocol
Accept / Reject
33. Athena vs RDS
Athena	RDS
Serverless	Managed database
Data commonly in S3	Data stored in database
Pay primarily for query data scanned	Pay for DB instances/storage
Great for analytics	Great for transactional workloads
SQL	SQL
No DB server management	AWS manages DB infrastructure
34. Athena vs Redshift
Athena	Redshift
Serverless query service	Data warehouse
Data usually in S3	Data stored in warehouse
Excellent for ad-hoc queries	Excellent for repeated analytics
No cluster management	Cluster/serverless warehouse management
Pay per data scanned / configured capacity	Pricing depends on selected Redshift architecture
35. Athena vs CloudWatch Logs Insights
Athena	CloudWatch Logs Insights
Query data in S3	Query CloudWatch log groups
Long-term data lake analytics	Operational log analysis
SQL	Logs Insights query language
Good for large historical datasets	Good for near-real-time troubleshooting
36. Athena + QuickSight

Athena and QuickSight work very well together.

Architecture:

S3
 ↓
Athena
 ↓
QuickSight
 ↓
Dashboard

Example:

Application Logs

↓

Athena

↓

Daily Error Count

↓

QuickSight Dashboard

We'll study this integration in detail in the QuickSight chapter.

37. Athena Security

Athena security commonly involves:

IAM
S3 bucket policies
Glue Data Catalog permissions
KMS encryption
Workgroups

Architecture:

IAM
 ↓
Athena
 ↓
Glue Catalog
 ↓
S3
38. S3 Permissions

Athena needs appropriate permissions to access the underlying data.

For example:

Athena

↓

S3:GetObject

↓

S3 Data

Access should follow least privilege.

39. Encryption

S3 data can be encrypted using:

SSE-S3
SSE-KMS

Query results can also be encrypted.

This becomes particularly important for sensitive logs and business data.

40. Best Practices – Part 1

✔ Store analytics data in Amazon S3.

✔ Use Glue Data Catalog for metadata.

✔ Prefer Parquet or ORC for large datasets.

✔ Avoid unnecessary SELECT *.

✔ Use Workgroups to organize queries.

✔ Restrict S3 access using IAM.

✔ Encrypt sensitive data.

✔ Separate raw and processed data.

41. Common Mistakes

❌ Treating Athena as a traditional database.

❌ Assuming Athena stores the actual data.

❌ Scanning huge datasets unnecessarily.

❌ Using CSV for every large-scale workload.

❌ Giving excessive S3 permissions.

❌ Ignoring query-result storage.

42. Interview Questions
Question 1

What is Amazon Athena?

Answer

Amazon Athena is a serverless interactive query service that allows you to analyze data stored in Amazon S3 using SQL.

Question 2

Does Athena require a database server?

Answer

No.

Athena is serverless, so you don't provision or manage database servers.

Question 3

Where is the actual data stored when using Athena?

Answer

The data is commonly stored in Amazon S3. Athena queries the data directly from its S3 location.

Question 4

What is the AWS Glue Data Catalog?

Answer

It is a centralized metadata repository containing information such as table schemas, column definitions, data types, S3 locations, and partitions.

Question 5

Does Athena store data?

Answer

Athena is primarily a query service. The data being queried commonly remains in S3, while Athena maintains query-related metadata and produces query results.

Question 6

What is an external table?

Answer

An external table describes data stored outside the query service, commonly in Amazon S3. The table definition tells Athena how to interpret that data.

Question 7

Why is Parquet preferred for analytics?

Answer

Parquet is a columnar format that allows analytics engines to read only required columns, often reducing data scanned and improving query performance and cost efficiency.

Question 8

How does Athena pricing generally work?

Answer

For SQL queries using the standard pricing model, charges are primarily based on the amount of data scanned. Other Athena features can have their own pricing models.

Question 9

Where are Athena query results stored?

Answer

Athena query results are commonly stored in an S3 location configured for the workgroup or query execution.

Question 10

Can Athena query CloudTrail logs?

Answer

Yes.

CloudTrail logs stored in S3 can be queried using Athena after an appropriate table/schema is configured.

43. Hands-on Labs
Lab 1 – Create an S3 Dataset

Create:

s3://my-athena-lab/

Upload a CSV file:

users.csv

Example:

1,Shivam,India
2,Rahul,India
3,Amit,USA
Lab 2 – Create Athena Database
CREATE DATABASE athena_lab;
Lab 3 – Create Table
CREATE EXTERNAL TABLE users (
    id INT,
    name STRING,
    country STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
LOCATION 's3://my-athena-lab/';
Lab 4 – Query Data
SELECT *
FROM users;
Lab 5 – Filter Data
SELECT name
FROM users
WHERE country = 'India';
Lab 6 – Count Users
SELECT COUNT(*)
FROM users;
44. One-Page Revision
Amazon Athena

        ↓

Serverless SQL

        ↓

Amazon S3
        ↓
Actual Data

        +

Glue Data Catalog
        ↓
Metadata

        ↓

Athena Query Engine

        ↓

SQL Query

        ↓

Results

        ↓

Amazon S3

Remember:

Athena ≠ Traditional Database

Athena = Serverless Query Service
Think Like a Senior DevOps Engineer

When you see large amounts of data sitting in S3, don't immediately think:

S3
 ↓
EC2
 ↓
Database
 ↓
Query

First ask:

Can Athena query this data directly?

For many log-analysis and ad-hoc analytics workloads, the architecture can be:

Application / AWS Service
          ↓
        Logs
          ↓
     Amazon S3
          ↓
       Athena
          ↓
         SQL
          ↓
       Results

And when visualization is required:

S3
 ↓
Athena
 ↓
QuickSight
 ↓
Dashboard

The key mental model is:

S3 stores the data, Glue Data Catalog describes the data, Athena queries the data, and QuickSight visualizes the results.
