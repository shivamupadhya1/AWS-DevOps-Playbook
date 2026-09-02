# Amazon Athena – Part 1

> AWS DevOps Playbook
>
> Volume 1 – Data, Analytics & Observability
>
> Chapter 43
>
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
