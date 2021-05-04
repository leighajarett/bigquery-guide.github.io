---
title: Tables (incl. Federation & Views)
layout: default
categories: (1) Resource Concepts
permalink: /resource_concepts/tables/
order: 2
description: Here, we’ll go through some common patterns for structuring your BigQuery projects. You can combine these techniques to create a hierarchy that works best for your organization.
next_page_title: Routines & ML Models
next_page_permalink: /resource_concepts/routines/
prev_page_title: Projects Structures
prev_page_permalink: /resource_concepts/project_structures/
---
![image](/assets/images/BQ_Console_Tables.png){: style="float: right;width: 50%; margin-left: 40px; margin-bottom: 40px"}

A BigQuery *table* is a resource that lives inside of a dataset. It contains individual records organized in rows, with each record composed of columns (also called fields) where a specified data type is enforced.

- **Data access** can also be controlled at the table, row and column levels. More details on data governance are discussed later
- **Metadata** such as descriptions and labels can be used for surfacing information to end users and as tags for monitoring
- **Expiration:** When you create a  table in BigQuery you are able to specify an expiration time 
- **Creating & Managing:** You can create and manage a table either directly in the UI, through the API / Client SDKs or in a SQL query using a [DDL statement](https://cloud.google.com/bigquery/docs/reference/standard-sql/data-definition-language). You can see information about tables directly in the UI nested underneath your datasets. 
- **Temporary Tables:** You can also create a temporary table using the [TEMP or TEMPORARY keyword](https://cloud.google.com/bigquery/docs/reference/standard-sql/data-definition-language#temporary_tables), this table can then be referenced for the duration of the script
- **Cached Tables:** BigQuery writes all query results to a table - one either explicitly identified by the user or to a [cached results table](https://cloud.google.com/bigquery/docs/cached-results). Temporary, cached results tables are maintained per-user, per-project. There are no storage costs for temporary tables.

### Schemas and Data Types
When you create a table in BigQuery, you can provide a schema or leverage autodetect. Schemas can also include **column descriptions.** BigQuery supports several different data types which are listed below, including arrays, structs and geographys. You can learn more about working with table schemas here. 

#### Numeric

| Name | Description |
| --- | ------ ----- |
| Integer (INT64) | 64 bit signed integer, no way to represent unsigned integer |
| Floating point (FLOAT64) | Includes -inf, +inf, and NaN |
| Numeric (NUMERIC) | Exact numeric value of precision 38 (total digits) and scale 9 (digits after decimal), offers more precision than floating point |
| BigNumber, alpha (BIGNUMERIC) | Exact numeric value of precision 76+, scale 38, useful for when having a lot of precision is really important (e.g. financial use cases) |
| Bool (BOOL) | TRUE / FALSE, or NULL |

#### String Like

| Name (API representation) | Description |
| --- | ----------- |
| String (STRING) | UTF-8 string values |
| Bytes (BYTES) | raw byte values, often base64 encoded due to transport needs |
| Geography (GEOGRAPHY) | collection of points/lines/polygons on the earth |

#### Temporal 

| Name (API representation) | Description |
| --- | ----------- |
| Date (DATE) | logical calendar date, not timezone aware |
| Time (TIME) | time only representation, not timezone aware |
| Datetime (DATETIME) | date and time (microsecond precision), not timezone aware |
| Timestamp (TIMESTAMP) | date and time (microsecond precision), **timezone aware** |

#### Complex

| Name (API representation) | Description |
| --- | ----------- |
| Array (REPEATED) | list of elements of another type. Elements maintain order, list can be empty, cannot be persisted as  NULL, cannot directly create an array of arrays |
| Struct (RECORD) | representation made up of multiple leaf fields (like a JSON). Structs can contain all other types including other structs. Leaf fields can be anonymous (meaning they don’t have a specified name, but they do have a type) or have a named type plus identifier |

<a href="https://cloud.google.com/bigquery/docs/reference/standard-sql/data-types" class="button">Details on Data Types (docs)</a>

## Managed (Native tables)

Managed tables are tables that are backed by native BigQuery storage, which we’ll dive deeper into in [later modules](#). With managed tables, a user defines basic logical constraints in BigQuery (e.g. schema, lifecycle) and the system manages the details. 

![image](/assets/images/tables.png){: style="float: right;width: 50%; margin-left: 40px; margin-bottom: 0px; margin-top: 20px"}

```
CREATE TABLE `project-id.my_dataset.order_items`
 (
   ORDER_ID INT64 OPTIONS(description="A unique ID for each order, used to join with orders table"),
   ...
 )
 OPTIONS(
   expiration_timestamp=TIMESTAMP "2023-01-01 00:00:00 UTC",
   description="Transactions table that expires in 2023",
   labels=[("org_unit", "development")]
 ) 
 ```

- **Partitions & Clusters:** Managed tables support both partitions and clusters which can drastically improve performance and make queries more cost effective. Partitions and cluster keys must be set when the table is being created. We go into detail on what partitions and clusters are and how to [optimize your data storage in next section](#). 

- [**Time Travel:**](https://cloud.google.com/bigquery/docs/time-travel) BigQuery lets you use time travel to access data that has been changed or deleted. You can access the data from any point within the last seven days and query data that was updated or deleted, restore a table that was deleted, or restore a table that expired. 
    - ```SELECT *
FROM `project-id.my_dataset.my_table`
  FOR SYSTEM_TIME AS OF TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 HOUR);```

<a href="https://cloud.google.com/bigquery/docs/tables-intro" class="button">Working with Tables (docs)</a>

## External (Federated tables)
External tables are backed by storage external to BigQuery. *This can be especially useful for some ETL patterns, or multi-consumer workflows where BQ storage isn’t the source of truth.*

![image](/assets/images/external_tables.png){: style="float: right;width: 50%; margin-left: 40px; margin-bottom: 70px; margin-top: 70px"}

- BigQuery current supports the following external storage systems: Cloud SQL (including partitions), Cloud Storage, Cloud Bigtable and Google Drive 
- You can join external tables onto native tables so long as the underlying data is stored in the same region
- You can create temporary external tables that are just used for single query, this is especially useful for ad-hoc queries 

### Connection
![image](/assets/images/external_tables.png){: style="float: right;width: 50%; margin-left: 40px; margin-bottom: 70px; margin-top: 70px"}

- More analogous to a "federated dataset", it is a link to a remote system (currently only Cloud SQL is supported), but information about the tables are not defined locally on the BigQuery side
- With an [external query](https://cloud.google.com/bigquery/docs/cloud-sql-federated-queries), CloudSQL returns results which are then stored as temporary tables to be used in BigQuery

```
SELECT c.customer_id, c.name, SUM(t.amount) AS total_revenue,
rq.first_order_date
FROM customers AS c
INNER JOIN transaction_fact AS t ON c.customer_id = t.customer_id
LEFT OUTER JOIN EXTERNAL_QUERY(
  'connection_id',
  '''SELECT customer_id, MIN(order_date) AS first_order_date
  FROM orders
  GROUP BY customer_id''') AS rq ON rq.customer_id = c.customer_id
GROUP BY c.customer_id, c.name, rq.first_order_date;
```

External tables and connections are both examples of **federation**, where a system interrogates a remote system via a federation mechanism to access data at runtime
- This is different than *database mirroring* which is generally a faster scan technique that involves replication of data, but may result in stale data
- This is also different than [BigQuery Omni](https://cloud.google.com/blog/products/data-analytics/introducing-bigquery-omni) which allows you to leverage the BigQuery UI and run Google Cloud managed Anthos clusters in other clouds

### Use cases 
Keep in mind that **query performance for external data sources may not be as high as querying data in a native BigQuery table**, and when you query an external data source other than Cloud Storage, the results are not cached. 

<u>Federation in BigQuery is beneficial as it allows your data ecosystems to rely on a single source of truth. For example:</u>
- For ETL operations: grabbing data from an external system, transforming it in BigQuery and loading it into a managed table
- Data that is frequently changed or manually updated in a Google Sheet, for example product hierarchy mappings
- There is a data lake (e.g. Storage bucket) that feeds into other systems like DataProc - however, BigQuery spark connectors make it easier to leverage data stored in BigQuery as a data lake 
- As an intermediary place to run ad hoc queries against external data sources until there is enough justification to migrate the data into BigQuery

<a href="https://cloud.google.com/bigquery/external-data-sources" class="button">Querying External Data Sources (docs)</a>
<a href="https://cloud.google.com/bigquery/docs/working-with-connections" class="button">Working with Connections (docs)</a>
<a href="https://cloud.google.com/bigquery/docs/working-with-connections" class="button">Querying External Data (vid)</a>


## Views
Views are virtual tables that are defined by a SQL query. Views can either be a logical view or a materialized view

![image](/assets/images/views.png){: style="float: right;width: 50%; margin-left: 40px; margin-top: 50px; margin-bottom: 200px"}

- **Logical views** only reference a SQL query
    - When a user queries the view, BigQuery will execute the SQL statement to actually create the view at run time, it will not save the result anywhere
    - Authorized views let you share query results with particular users and groups without giving them access to the underlying tables
    - Logical views are less restrictive in terms of allowed SQL compared with materialized views

```
CREATE  VIEW project-id.my_dataset.my_mv_table AS
    SELECT trans.date, account.name, sum(trans.paid) AS total_paid
    FROM project-id.my_dataset.my_transactions trans
    LEFT JOIN project-id.my_dataset.my_accounts accounts
    ON trans.account_id = accounts.id
    GROUP BY trans.date, account.name
```

- **Materialized views** are precomputed views that periodically cache the results of a query for increased performance and efficiency
    - Materialized views are recomputed in the background when the base table changes,  no user action is required - they are always fresh
    - If a query against the base table can be resolved by querying the materialized view, BigQuery will also reroute for better performance 
    - Materialized views are useful if you have complex, predictable queries that need high processing power or if you need access to real-time aggregations
    - Keep in mind that materialized views use a restricted SQL syntax and a limited set of aggregation functions, they also can only use a single table (no joins)

```
CREATE MATERIALIZED VIEW project-id.my_dataset.my_mv_table AS
    SELECT date, sum(paid) AS total_paid
    FROM project-id.my_dataset.my_base_table
    GROUP BY date
```

<a href="https://cloud.google.com/bigquery/docs/views-intro" class="button">Working with Views (docs)</a>
<a href="https://cloud.google.com/bigquery/docs/share-access-views" class="button">Creating an Authorized View Tutorial</a>
<a href="https://cloud.google.com/bigquery/docs/share-access-views" class="button">Authorized Views (vid)</a>
<a href="https://cloud.google.com/bigquery/docs/materialized-views-intro" class="button">Working with Materialized Views (docs)</a>

