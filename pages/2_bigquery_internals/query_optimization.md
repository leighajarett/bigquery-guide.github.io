---
title: Query Optimization
layout: default
categories: (2) BigQuery Internals
permalink: /internals/query_optimization/
order: 4
description: 
next_page_title: 
next_page_permalink: 
prev_page_title: 
prev_page_permalink: 
---


### Shuffle Quotas
One thing to note is that joins which are very large may not fully fit into memory and will spill to disk, causing performance decrease. Shuffle quotas (e.g. the limit before spilling to disk) increases with the number of flat-rate slots purchased



<a href="https://medium.com/slalom-build/using-bigquery-execution-plans-to-improve-query-performance-af141b0cc33d" class="button">Using Execution Plans to Improve Performance (Blog)</a>