---
title: Storage Optimization
layout: default
categories: (2) BigQuery Internals
permalink: /internals/storage_optimization/
order: 3
description: Now that we understand what goes on under the hood, lets take a look at how we can optimize BigQuery storage
next_page_title: Query Optimization
next_page_permalink: /internals/query_optimization/
prev_page_title: Query Execution
prev_page_permalink: /internals/query_execution/
---

## Leveraging Partitions and Clusters

As we discussed in the [storage internals]() section, BigQuery allows you to cluster and partition your data so that data is co-located for optimal performance. So when should you actuallty use partitions and clusters, and how should decide which columns to leverage?

- For both partitions and clusters, you should **consider columns that can be used as filters in a where clause** for a large number of the queries run off of this table. For example, if you are often filtering on the date that an order was placed (say to only look at orders in the past week) or you're often focusing on a specific customer (by filtering on customer ID) these columns are probably good candidates!

- **Partitions are designed for places where there is a low number of distinct values**, generally less than a few thousand. If you over partition your tables, you’ll create a lot of metadata - which means that if you ever need to read in the entire table, it will be inefficient. For example, you might write queries where you're often looking at orders placed in the last hour. To make this query really efficient, you may decide to partition on the hour that the order was placed. However, if you have a few years worth of data this would result in more than 10,000 partitions. Not only is this over the 4,000 partition limit, but if users on your team are also writing queries to go further back in time having that many partitions would add overhead and decrease performance

- **Clusters can be used for columns with higher cardinality**. In BigQuery, you can specify up to four cluster keys - the data will be sorted in the order that the keys are provided, meaning the first key will likely have the largest impact the amount of data scanned. Clustering also helps to improve the performance of aggregations, so you may also want to consider keys that are often used to aggregate against. For example, if you're frequently calculating the total spend for each customer you might want to cluster on customer ID.

## Partitioning Instead of Sharding

In BigQuery, you can date shard a table by having multiple tables with the same schema and the suffix _YYYYMMDD. You can query the tables using a wildcard to conveniently perform unions of these tables: select * from example_dataset.data_sharded_*. Sharding is more of a legacy feature and we recommend partitioning over sharding because it's more intuitive (one table instead of many) and there is reduced query overhead, which results in better performance.

<a href="https://mark-mccracken.medium.com/bigquery-date-sharding-vs-date-partitioning-cee3754f7900" class="button">Date Sharding vs Partitioning (Blog)</a>

## Denormalizing Data

Denormalization is a strategy used to improve the performance on relational datasets. When data is normalized, information is stored in separate logical tables. For example, information about a user might be stored in a separate user table. But with denormalization we colocate data even if it means repeating some information. For example, attributes that might be saved in the user table (name, adress) are instead also incorporate into the fact table (like an orders table). This can improve performance because we don't need to read data from two different tables and join them. Because storage is so cheap, the cost of repeating this information is usually minimal. Additionally, structs and repeated records make it easy to structure this information in BigQuery. 

Although denormalizing data can improve efficiency, it may involve re-shaping many tables which can disrupt workflows so it's easiest to start with other improvements like partitions and clusters. More so, de-normalizing isn't a good fit for data where the dimensional values are often changing (for example, if users are often updating their addresses) because that would require updating multiple rows in BigQuery.

<a href="https://cloud.google.com/architecture/dw2bq/dw-bq-performance-optimization#denormalization" class="button">Denormalization (docs)</a>

## Take Advantage of Long-term Storage

Rather than exporting older data that you may not be regularly querying to another storage option (such as Cloud Storage), take advantage of BigQuery's long-term storage pricing. If you have a table that is not edited for 90 consecutive days, the price of storage for that table automatically drops by 50 percent to $0.01 per GB, per month. This is the same cost as Cloud Storage Nearline.

Each partition of a partitioned table is considered separately for long-term storage pricing. If a partition hasn't been modified in the last 90 days, the data in that partition is considered long term storage and is charged at the discounted price.

<a href="https://cloud.google.com/bigquery/docs/best-practices-storage#take_advantage_of_long-term_storage" class="button">Take Advantage of Long Term Storage (docs)</a>





