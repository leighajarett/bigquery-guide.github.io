---
title: Loading Data into BigQuery
layout: default
categories: (3) Working with BigQuery
permalink: /working/loading_data/
order: 1
description: Understand common user journeys and best practices for loading data into BigQuery
next_page_title: 
next_page_permalink: 
prev_page_title: 
prev_page_permalink: 
---


## Data Sources:
- From Google services like Cloud Storage or Stackdriver
- From non-Google services like S3 or Teradata
- From a readable data source (like your local machine)
- By inserting individual records using streaming inserts
- Using DML statements to perform bulk inserts
- Using a BigQuery I/O transform in a Cloud Dataflow pipeline to write data to BigQuery


## Load Jobs
Big takeaway: Batch load jobs are free and do not consume query capacity; you can load all the data you want per daily basis if it’s a batch load. 
Loading has its own dedicated BQ slots and won’t interfere with your regular query resources. Furthermore, customers may purchase loading slot reservations to fully ensure that query capacity and load capacity are separated since public bigquery slots have no specific guarantee.
Alternatively, you also have the option of doing streaming ingest for data, but that does get billed. 
ACID semantics: in BQ if you’re doing multiple load jobs to the same table and have conflicting inserts, which one will be materialized?
BQ Eliminates the need for manual locking. Therefore, if a job completes, you will know that the job transacted; if there is an error, you’ll know to retry. 

When you’re loading data, the file types that you use do affect the performance of loading 

Faster
Avro (compressed data blocks)
Avro (uncompressed)
Parquet / ORC (compressed: data blocks file footer, stripes) 
Parquet / ORC (uncompressed) 
CSV
JSON
CSV (compressed)
JSON (compressed)
Slower


## Best Practice
Prefer ELT into BigQuery over ETL where possible
Leverage federated queries to GCS to load and transform data in a single step
Load data into raw and staging tables before publishing to reporting tables
Use Dataflow or Data Fusion for streaming pipelines and to speed up large complex batch jobs
Get started streaming using Google-provided Dataflow Templates and modify the open source pipeline for more complex needs


### Batch:
- Loading with DDL
- Alternative to loading with DDL
- Loading with DML
- Alternatives to loading with DML
- Storage API via dataflow?

### Streaming:
- streaming inserts via dataflow?
