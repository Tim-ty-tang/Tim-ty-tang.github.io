---
title: "OLAP"
date: "2024-09-09"
summary: "on enterprise OLAPs"
description: "on enterprise OLAPs"
toc: false
readTime: true
---

## On enterprise OLAPs

Large-scale cloud OLAP has increasingly converged toward the lakehouse paradigm.

In this context:

- Internal tables refer to data loaded directly into these systems and use their proprietary format.
- External tables refer to data in customer-managed storage, often utilizing formats like Iceberg Delta Lake.

### BigQuery

BigQuery operates with a disaggregated architecture, where the Dremel query engine prosses data from Google’s Colossus file system.

- Table Format: Besides its proprietary format (Capacitor), Google recently introduced BigLake, allowing users to manage data in object storage and query multi-format tables. Iceberg support was added in 2022, and Hudi and Delta Lake followed in 2023.
- Metadata Management: They manage both internal and external table metadata using their dedicated serverless metadata system

### Databricks

Databricks emerged as a lakehouse solution, managing data in object storage through its SparkSQL/Photon engine and Delta Lake format.

- Table Format: They expanded with Delta UniForm in 2023, supporting compatibility for Iceberg and Hudi. In June 2024, they acquired Tabular, a company founded by Iceberg's creator.
- Metadata Management: Unity Catalog was introduced and open-sourced in 2024.

### Snowflake

Like BigQuery, Snowflake separates computing (via virtual warehouses) and storage. Data must also be loaded into the system to utilize its proprietary format.

- Table Format: It added Iceberg support in 2022 and Delta Lake via the external table later. However, I haven’t found any documentation on Hudi support yet.
- Metadata Management: Internal table metadata is managed via FoundationDB, but it’s unclear to me whether external tables use the same system. In 2024, they also open-sourced the Polaris Catalog.

### Redshift

AWS Redshift initially used a shared-nothing architecture. In 2019, the introduction of RA3 nodes with Redshift Managed Storage allowed separate computing and storage.

- Table Format: Iceberg support came in 2023, with Hudi and Delta Lake accessible through Redshift Spectrum.
- Metadata Management: In the paper in 2022, they shared that they also manage transaction metadata in the Redshift Manage Storage. For open table formats, Redshift utilizes AWS Glue Data Catalog and Lake Formation.

Conclusion: Iceberg seems to be the top priority across the systems mentioned, even with Databricks. Systems that separate computing and storage have an advantage when adopting the lakehouse paradigm.
