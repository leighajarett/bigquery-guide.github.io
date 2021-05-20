---
title: API Landscape
layout: default
categories: (3) Working with BigQuery
permalink: /working/api_landscape/
order: 0
description: There are many different APIs associated with BigQuery and each one serves a different purpose
next_page_title: 
next_page_permalink: 
prev_page_title: 
prev_page_permalink: 
---

## Working with these APIs

Web UI, CLI, ...

## BigQuery (v2) API
The BigQuery API is the coe of BigQuery and largely represents the resources and interactoions we discussed in the previous modules. 

1. **Storage Resources**
    - Datasets: Collections of other resources (tables, routines, models, etc).
    - Tables: Data with schema.  Types include managed, external, as well as logical and materialized views.  Also exposes functionality for streaming records into tables.
    - Routines: Encapsulates user defined functions (UDFs) and stored procedures.

2. **Execution Resources**
    - Jobs: Generalized work task.  Running a query, loading data, exporting a CSV, etc.

3. **Machine Learning**
    - ML Models:Access to trained models.  Can use queries to train and predict.


## Storage API
The Storage API allows you to use BigQuery like a Data Warehouse and a Data Lake. It's **real-time** so that you don’t have to wait for your data, it’s **fast** so that you don’t need to reduce or sample your data, and it’s **efficient** so that you should only read the data you want.

1. **Read Client:** Exposes a data-stream suitable for reading large volumes of data. It also provides features for parallelizing reads, partial projections, filtering data, and precise control over snapshot time.

2. **Write Client (alpha):** This is the succesor to the streaming mechanism found in the BigQuery v2 API. It supports more advanced write patterns such as exactly one semantics. 

The Storage API was used to build a series of Hadoop and Spark connectors so that you can run your Hadoop and Spark workloads directly on your data in BigQuery, meaning you no longer need a data lake alongside your warehouse. You can also **build your own connectors** using the Store API.

## BigQuery Reservation API
The Reservation API is used to programmatically purchase capacity commitments, create standard reservations and assignments or create BI engine reservations. 

## BigQuery Connections API
The BigQuery Connections API is used to create a connection to Cloud SQL. This enables BigQuery users to issue live, federated, queries against Cloud SQL. It also supports BigQuery Omni to define multi-cloud data sources and structures.

<!-- https://cloud.google.com/bigquery/docs/reference/bigqueryconnection/rest -->

## Data Transfer Service
BigQuery data transfer service allows you to automate work to ingest data from known data sources, with standard scheduling. DTS offers support for migration workloads, such as specialized integrations that help with synchronizing changes from a legacy, on premise warehouse.

## Data Catalog
Data Catalog provides infrastructure for indexing and searching resources / metadata, for both BigQuery and other data analytics products. Additionally, it exposesfunctionality related to data governance, such as defining policy tags for extending access to sensitive data columns.

# Data QnA 
Building an analytics chatbot with Data QnA


On the next few pages we'll walk through some common user journeys where these APIs may come into play. 


<!-- CUJs

Loading data into BigQuery 
- Dataflow is incredibly powerful for batch and stream processing. Today, you can run Dataflow jobs on top of BigQuery data using the Storage API, enriching it with data from PubSub, Spanner, or any other data source. ??
https://docs.google.com/presentation/d/1PW8W4V0kPeMKvGgiKE8vqw2PlpoX6Mh7DYOj853-dqY/edit#slide=id.g61817d4183_7_878

Exporting data from BigQuery

Programmatically managing execution resources (purchasing slots or refining assignments based on query workloads)
- Programmatically purchasing flex slots and canceling them 
- Adding or removing projects to assignment  
https://github.com/Fourcast/bq_flex_slots/blob/master/README.md

Metadata management (find seth notes) -->



