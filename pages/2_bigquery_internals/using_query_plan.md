<!-- ---
title: Using Query Plans
layout: default
categories: (2) BigQuery Internals
permalink: /internals/query_plans/
order: 3
description: Now that you understand how queries are executed in BigQuery, lets go deeper into how you can use the query plan
next_page_title: 
next_page_permalink: 
prev_page_title: 
prev_page_permalink: 
--- -->


In the previous section, we went through the different query execution concepts. Now, let's go even deeper into the query plan and discuss how to use it to make informed decisions. 

In BigQuery, execution plans are divided into stages corresponding to the leaf nodes (stage 0) and mixer nodes (stages 1 to n). Each stage has 4 components, as shown in Figure 2 below.

The most important phase in any stage of a BigQuery execution plan is the compute phase, where the actual processing takes place, such as evaluating SQL functions or expressions. As the name implies, optimizer engine is waiting in the wait phase. It is waiting for either the slots to become available or for a previous stage to start writing results that it can begin consuming. In the read phase, the slot is reading data either from Colossus (in the case of leaf nodes) or from a previous stage (in the case of mixer nodes). The final stage is the write phase, where data is written, either to the next stage, or the master node, which is the output returned to the developer.

Timings in an execution plan are not absolute, rather they are relative to the timings of other phases. Since there are multiple parallel workers processing the data, the execution plan provides both the average and maximum times across all workers. A well-tuned query typically spends most of its time in the compute phase, and an average compute time close to max compute time indicates an even distribution of data coming out of the previous stage.

https://cloud.google.com/bigquery/query-plan-explanation

https://medium.com/slalom-build/using-bigquery-execution-plans-to-improve-query-performance-af141b0cc33d
