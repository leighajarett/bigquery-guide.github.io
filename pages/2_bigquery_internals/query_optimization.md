---
title: Query Optimization
layout: default
categories: (2) BigQuery Internals
permalink: /internals/query_optimization/
order: 4
description: Now that we have some pointers on optizing storage, lets walk through how we can also optimize queries
next_page_title: 
next_page_permalink: 
prev_page_title: 
prev_page_permalink: 
---

## Pruning or Pre-filtering Data


### Limit Columns and Rows
One of the most straightforward optimizations you can make on queries is to make sure you don’t use `select *`. Since BigQuery leverages columnar storage, reading in lots of columns can be expensive. Instead, use only the columns you need in the query. 

Including filters (where or having) is also helpful as it saves you IO time in any shuffles or joins performed on the filtered rows and CPU time for computations on those rows.

https://cloud.google.com/bigquery/docs/best-practices-performance-input#control_projection_-_avoid_select


### Materialize Your Data

Sometimes you might find yourseulf reference a Common Table Expression (defined with a WITH) clause, in multiple places:

```
with a as (
  select ...
),
b as (
  select ... from a ...
),
c as (
  select ... from a ...
)
select 
  b.dim1, c.dim2
from
  b, c;
```

However, at runtime the contents of the subquery will be inlined every place the alias is referenced. This can lead to the same query being executed multiple times. Instead, you should refactor it to remove multiple references or refactor the  query to use a TEMP table.

```
create temp table a as
select …;

with b as (
  select ... from a ...
),
c as (
  select ... from a ...
)
select 
  b.dim1, c.dim2
from
  b, c;
```

### Push filters down
Finally, it's important to filter data as early as possible, especially before JOINs and aggregation. These are the most expensive parts of a query, so filtering data significantly improves performance. For example, if the condition in the WHERE clause can be evaluated on left or right of the JOIN independently, rewrite the following:

```
 SELECT FROM (...) JOIN (...) ON ... WHERE ...
```
 to
```
 SELECT FROM (... WHERE ...) JOIN (…) ON ...
```

## Late aggregation

Aggregate as late and as seldom as possible, because aggregation is very costly. BUT if a table can be reduced drastically by aggregation in preparation for being joined, then aggregate it early. With JOINS, this will only work if the two tables are already being aggregated to the same level (i.e., if there is only one row for every join key value).

Instead of this:

```
select
  t1.dim1,
  sum(t1.m1)
  sum(t2.m2)
from (select
    dim1, 
    sum(metric1) m1
  from `dataset.table1` group by 1) t1
join (select
    dim1, 
    sum(metric2) m2
  from `dataset.table2` group by 1) t2
on t1.dim1 = t2.dim1
group by 1;
```

Do this:

```
select
  t1.dim1,
  sum(t1.m1)
  sum(t2.m2)
from (select
    dim1, 
    metric1 m1
  from `dataset.table1`) t1
join (select
    dim1, 
    metric2 m2
  from `dataset.table2`) t2
on t1.dim1 = t2.dim1
group by 1;
```

## Optimizing Joins:
- placing the largest table first
- skewed join

## Optimizing WHERE
- order matters, use the filter that will eliminate the most data

## Optimizing ORDER BY
- Including Limit with Order By

## Optimizing CPU tasks
For long periods of time spent on CPU tasks consider approx functions https://cloud.google.com/bigquery/docs/reference/standard-sql/approximate_aggregate_functions, filtering early and optimize your UDF usage with a persistent UDF or avoid using all together (see Best Practices for functions section at end)


## Simplify String Operations
(prefer like to regex)



