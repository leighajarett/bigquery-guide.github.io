---
title: Query Optimization
layout: default
categories: (2) BigQuery Internals
permalink: /internals/query_optimization/
order: 4
description: Now that we have some pointers on optizing storage, lets walk through how we can also optimize queries
next_page_title: 
next_page_permalink: 
prev_page_title: Storage Optimization
prev_page_permalink: /internals/storage_optimization/
---

## General Optimization Tips

### Limit Columns and Rows
- One of the most straightforward optimizations you can make on queries is to make sure you don’t use `select *` Since BigQuery leverages columnar storage, reading in lots of columns can be expensive. Instead, use only the columns you need in the query. 

- Including filters (`WHERE` or `HAVING`) is also helpful as it saves you IO time in any shuffles or joins performed on the filtered rows and CPU time for computations on those rows.

<a href="https://cloud.google.com/bigquery/docs/best-practices-performance-input#control_projection_-_avoid_select
" class="button">Using Approximate Aggregation Functions (docs)</a>

### Optimizing WHERE
BigQuery assumes that the user has provided the best order of expressions in the WHERE clause, and does not attempt to reorder expressions. Expressions in your WHERE clauses should be ordered with the most selective expression first. 

### Optimizing ORDER BY
Writing results for a query with an ORDER BY clause can result in *Resources Exceeded* errors. Because the final sorting must be done on a single worker, if you are attempting to order a very large result set, the final sorting can overwhelm the slot that is processing the data. Instead, include LIMIT clause so the workers can eliminate data points earlier in the query execution.

### Materialize Your Data
Sometimes you might find yourseulf reference a Common Table Expression (defined with a WITH) clause, in multiple places:

```
WITH a AS (
  SELECT ...
),
b AS (
  SELECT ... FROM a ...
),
c as (
  SELECT ... FROM a ...
)
SELECT 
  b.dim1, c.dim2
FROM
  b, c;
```
However, at runtime the contents of the subquery will be inlined every place the alias is referenced. This can lead to the same query being executed multiple times. Instead, you should refactor it to remove multiple references or refactor the  query to use a TEMP table.

```
CREATE TEMP TABLE a AS
SELECT …;

WITH b AS (
  SELECT ... FROM a ...
),
c as (
  SELECT ... FROM a ...
)
SELECT 
  b.dim1, c.dim2
FROM
  b, c;
```

### Late aggregation
Aggregate as late and as seldom as possible, because aggregation is very costly. 

Instead of this:
```
SELECT
  t1.dim1,
  SUM(t1.m1)
  SUM(t2.m2)
FROM (SELECT
    dim1, 
    sum(metric1) m1
  FROM `dataset.table1` GROUP BY 1) t1
JOIN (SELECT
    dim1, 
    sum(metric2) m2
  FROM `dataset.table2` GROUP BY 1) t2
ON t1.dim1 = t2.dim1
GROUP BY 1;
```
Do this:
```
SELECT
  t1.dim1,
  SUM(t1.m1)
  SUM(t2.m2)
FROM (select
    dim1, 
    metric1 m1
  FROM `dataset.table1`) t1
JOIN (SELECT
    dim1, 
    metric2 m2
  FROM `dataset.table2`) t2
ON t1.dim1 = t2.dim1
GROUP BY 1;
```

**But,** if a table can be reduced drastically by aggregation in preparation for being joined, then aggregate it early - and make sure to aggregate the tables to the same level (i.e., one row for every join key value).

## Optimizing Joins:

### Push filters down
It's important to filter data as early as possible, especially before JOINs and aggregation. These are the most expensive parts of a query, so filtering data significantly improves performance. For example, if the condition in the WHERE clause can be evaluated on left or right of the JOIN independently, rewrite the following:

```
 SELECT FROM (...) JOIN (...) ON ... WHERE ...
```
to
```
 SELECT FROM (... WHERE ...) JOIN (…) ON ...
```

This can be especially important when working with skewed data. In the previous section we taked about how BigQuery performs joins. All data with the same join key will wind up in the same slot. But if your data is heavily skewed you could end up with some workers being overloaded and some barely having any data. You may have a sense that this is happening when you see that the maximum execution time in the query plan is much higher than the average.

![img](/assets/images/skewed_join.png){: style="max-width:70%; float:center;" }

### Place the largest table first
The standard SQL query optimizer can sometimes determine which table should be on which side of the join, but it is still recommended to order your joined tables appropriately. The best practice is to manually place the largest table first, followed by the smallest, and then by decreasing size.

## Optimizing Functions

### Use Approximation Functions if Possible
If the SQL aggregation function you're using has an equivalent approximation function, the approximation function will yield faster query performance. For example, instead of using `COUNT(DISTINCT)`, use `APPROX_COUNT_DISTINCT()`. Approximate functions produce a result which is generally within 1% of the exact number. 

<a href="https://cloud.google.com/bigquery/docs/best-practices-performance-compute#use_approximate_aggregation_functions" class="button">Using Approximate Aggregation Functions (docs)</a>

### Simplify String Operations
REGEXP_CONTAINS offers more functionality, but also has slower execution time. Try using LIKE when the full power of regex is not needed (e.g. wildcard matching). 

### Use ARRAY_AGG over ROW_NUMBER
Using a ROW_NUMBER function can be a good way to return only the latest records that exist in a table, for example `row_number() over( partition by id order by created_at desc)`. However, as data size grows, you may end up with *Resources Exceeded* errors because the query has too many elements to ORDER BY in a single partition. A standard SQL trick to get around this is to use ARRAY_AGG -  `array_agg( t order by t.created_at desc limit 1)[offset(0)]`

