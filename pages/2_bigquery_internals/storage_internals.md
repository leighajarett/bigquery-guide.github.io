---
title: Storage Internals
layout: default
categories: (2) BigQuery Internals
permalink: /internals/storage/
order: 0
description: Here, we’ll go walk through what happens under the hood for BigQuery storage
next_page_title: Life of a Query
next_page_permalink: /internals/query
prev_page_title: Workload Management
prev_page_permalink: /resource_concepts/workload_management/
---

BigQuery offers fully managed storage, meaning you don't have to provision servers. Sizing is done automatically and you only pay for what you use. 


## Columnar Storage
![image](/assets/images/column_storage.png){: style="float: right;width: 50%; margin-left: 40px; margin-bottom: 10px"}

Traditional relational databases, like Postgres and MySQL, store data row-by-row - **record-oriented storage**. This makes them good at transactional updates and OLTP (Online Transaction Processing) use cases because they only need to open up a single row to read or write data. However, if you want to perform an aggregation like a sum of an entire column, you would need to read the entire table into memory.

BigQuery, uses **columnar storage** where each column is stored in a separate file block. This makes BigQuery an ideal solution for OLAP (Online Analytical Processing) use cases. When you want to perform aggregations you only need to read the column that you are aggregating over. 


## Optimized Storage Format
![image](/assets/images/capacitor.png){: style="float: right;width: 50%; margin-left: 40px; margin-bottom: 10px"}

Internally, BigQuery stores data in a proprietary columnar format called **Capacitor**

- Each column in the table is stored in a separate file block and all the columns are stored in a single capacitor file, which is **compressed and encrypted on disk** 

- Capacitor builds an approximation model that takes in relevant factors like the type of data (e.g. a really long string vs an integer) and usage of the data (e.g. some columns are more likely to be used as filters, for example in WHERE clauses) in order to **reshuffle rows and encode columns**

- While every column is being encoded, BigQuery also collect various **statistics about the data** — which are persisted and used later during query execution

- BigQuery will also use these access patterns to determine an **ideal number of shards**, which is discussed in more detail on the query processing page


## Encryption and Managed Durability
![image](/assets/images/colossus.png){: style="float: right;width: 50%; margin-left: 40px; margin-bottom: 30px"}

The actual persistence layer is provided by Google’s distributed file system, Colossus, where data is automatically compressed, encrypted, replicated, and distributed. 

- There are many levels of defense against unauthorized access in Google Cloud Platform, one of them being that **100% of data is encrypted at rest**

- Colossus ensures **durability** using erasure encoding to store redundant chunks of data on multiple physical disks

- Immediately upon writing data to Colossus, BigQuery starts the **geo-replication** process, mirroring all the data into different data centers around the specified region

This is all accomplished without impacting the compute power available for your queries. Plus encoding, encryption and replication are included in the price of BigQuery storage - no hidden costs.

<a href="https://cloud.google.com/blog/products/bigquery/inside-capacitor-bigquerys-next-generation-columnar-storage-format" class="button">Inside Capacitor (blog)</a>
<a href="https://medium.com/google-cloud/bigquery-explained-storage-overview-70cac32251fa" class="button">BQ Storage Explained (blog)</a>

## Paritions and Clusters
BigQuery takes several steps to optimize storage behind the scenes. But when you create a new table, you also have some levers to pull that can optimize how data is stored based on your needs, namely *Partitioning and Clustering.* 

### Partitions
A partitioned table is a special table that is divided into segments, called partitions. BigQuery leverages partitioning to minimize the amount of data that workers read from disk. **Queries that express filters on the partitioning column can dramatically reduce the overall data scanned, which can yield improved performance and reduced query cost for on-demand queries.** New data written to a partitioned table is automatically delivered to the appropriate partition.  

BigQuery supports following ways to create partitioned tables:
![image](/assets/images/partitions.png){: style="float: right;width: 50%; margin-left: 20px; margin-bottom: 80px; margin-top: 80px"}

- [**Ingestion time partitioned tables:**](https://cloud.google.com/bigquery/docs/creating-partitioned-tables)
    - Daily partitions reflecting  the time the data was ingested into BigQuery
    - BigQuery adds two pseudo columns to ingestion-time partitioned tables — a _PARTITIONTIME pseudo column containing a date-based timestamp for data and a _PARTITIONDATE pseudo column contains a date representation
    - You can [create an empty table with ingestion time partioning in the console or through the API](https://cloud.google.com/bigquery/docs/creating-partitioned-tables#console)

- [**DATE/TIMESTAMP column partitioned tables:**](https://cloud.google.com/bigquery/docs/creating-column-partitions)
    - BigQuery routes data to the appropriate partition based on the date value (expressed in UTC) in the partitioning column
    - You can create partitions with granularity starting from hourly partitioning
    ```
    CREATE TABLE
        mydataset.newtable (transaction_id INT64, customer_id INT64, transaction_ts TIMESTAMP)
    PARTITION BY
        TIMESTAMP_TRUNC(transaction_ts, HOUR)
    ```

- [**INTEGER range partitioned tables:**](https://cloud.google.com/bigquery/docs/creating-integer-range-partitions)
    - Partitioned based on an integer column with start, end, and interval values (e.g. start at 0, end at 100, interval of 10)
    ```
    CREATE TABLE
        mydataset.newtable (transaction_id INT64, customer_id INT64, transaction_ts TIMESTAMP)
    PARTITION BY
        RANGE_BUCKET(customer_id, GENERATE_ARRAY(0, 100, 10))
    ```

<a href="https://cloud.google.com/bigquery/docs/partitioned-tables" class="button">Working with Partitioned Tables (docs)</a>

### Clusters
![image](/assets/images/clusters.png){: style="float: right;width: 50%; margin-left: 20px; margin-bottom: 10px"}

When a table is clustered in BigQuery, the data is automatically sorted based on the contents of one or more columns (that you specify). Usually high cardinality and non-temporal columns are preferred for clustering.

The order of clustered columns determines the sort order of the data. When new data is added to a table or a specific partition, BigQuery performs **free, automatic re-clustering** in the background.

 ```
CREATE TABLE
    mydataset.newtable (transaction_id INT64, customer_id INT64, transaction_ts TIMESTAMP)
ClUSTER BY
    [ customer_id, transaction_id ] 
```

Clustering can improve the performance of certain types of queries:
- Queries containing where clauses with a clustered column: BigQuery uses the sorted blocks to eliminate scans of unnecessary data
- Queries that aggregate data based on values in a clustered column: performance is improved because the sorted blocks collocate rows with similar values

BigQuery supports clustering over both partitioned and non-partitioned tables.


<a href="https://cloud.google.com/bigquery/docs/clustered-tables" class="button">Working with Clustered Tables (docs)</a>
<a href="https://codelabs.developers.google.com/codelabs/gcp-bq-partitioning-and-clustering#0" class="button">Partitioning & Clustering (Tutorial)</a>


For details on how you should make decisions for partitions and clusters, check out the [storage optimization](/internals/storage_optimization/) section!


