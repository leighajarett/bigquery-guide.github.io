---
title: Query Execution
layout: default
categories: (2) BigQuery Internals
permalink: /internals/query_execution/
order: 2
description: Armed with our new understanding of the life of a query, we can dive deeper into query execution. 
next_page_title: 
next_page_permalink: 
prev_page_title: 
prev_page_permalink: 
---
Earlier in this section, we talked about Colossus - Googles's distributed filesystem which stores BigQuery data. To actually compute data, BigQuery uses Dremel. Workers, or slots, are used to extract data from storage (we'll call these leaf nodes) and perform aggregations (we'll call these mixers). Data that needs to be shared between slots is stored in the shuffle. Movement of data is facilitated by Jupiter, the 1 petabit/sec network through which storage and compute talk.

## Remote Memory Shuffle
![image](/assets/images/memory_shuffle.png){: style="float: right;width: 50%; margin-left: 40px; margin-bottom: 40px"}

In-memory BigQuery shuffle **stores intermediate data produced from various stages of query processing** in a set of nodes that are dedicated to hosting remote memory. 

This lets you query large datasets and get fast responses. Each executed query is broken up into stages which are then processed by workers / slots and written back out to shuffle. We’ll see this later on with the query plan, which details each of these stages.

In BigQuery, each shuffled row can be consumed by BigQuery workers as soon as it's created by the producers. This makes it possible to execute distributed operations in a pipeline. If a worker were to have an issue during query processing, another worker would simply pick up where the previous one left off by reading from the previous shuffle. This provides resilience to failures within workers themselves. When a query is complete, the results are written out to persistent storage and returned to the user. This also enables us to serve up cached results the next time that query executes.

## Simple Query: Scan, Filter, Aggregate
Now that we understand the architecture that allows for query execution, let’s take a look at a relatively simple query. 
Lets say we want to count the number of citibike trips that began at stations with Broadway in the name.

```
SELECT COUNT(*)
FROM `bigquery-public-data.new_york.citibike_trips`
WHERE start_station_name LIKE "%Broadway%"
```

![image](/assets/images/simple_execution_1.png){: style="float: right; max-width: 30%; margin-left: 40px; margin-bottom: 40px"}

###  High level stages of execution:

From this diagram, we can see at a high level how BigQuery breaks down execution into different stages:  

1. In the first stage, a set of leaf nodes access the distributed storage to read the table, filter down the records, and generate partial counts

2. These workers then send their counts to another stage, which sums them all together and returns the final result to the query controller.

### Now lets go into a bit more detail:

![image](/assets/images/simple_execution_2.png){: style="float: right;width: 50%; margin-left: 40px; margin-bottom: 40px"}

1. Here, we see that the input table is made up of a set of input files.  For each input file, we need a worker to process the file, read and filter the rows.  Each worker then writes one record into shuffle that contains the partial count for that input file.

2. The second stage here reads from those shuffle records as its input, and sums them together.  It then writes the output file into a single file, which becomes accessible as the result of the query.

**You can see that the workers don't communicate directly with one another at all; they simply communicate through reading and writing data.**

![image](/assets/images/simple_execution_console.png){: style="float: right;width: 50%; margin-left: 40px; margin-bottom: 40px"}

In the console, we can view the query plan to get even deeper into the details of the query execution. In BigQuery, execution plans are divided into the different stages of execution, with stage 0 where the leaf nodes do work and stage 1+  where the mixers do work.

There are also several different phases in each stage. The compute phase is where the actual processing takes place, such as evaluating SQL functions or expressions. In the wait phase, the engine is waiting for either the slots to become available or for a previous stage to start writing results that it can begin consuming. In the read phase, the slot is reading data either from Colossus (in the case of leaf nodes) or from shuffle (in the case of mixer nodes). The final stage is the write phase, where data is written, either to the next stage, or shuffle, which is the output returned to the developer.

A well-tuned query typically spends most of its time in the compute phase, and an average compute time close to max compute time indicates an even distribution of data coming out of the previous stage.

![image](/assets/images/simple_execution_qp.png){: style="float: center;width: 70%; margin-left: 40px; margin-bottom: 40px"}

In the query plan for our example query, we can see the stages as reflected in the diagram above. 


## Combining Data: Understanding JOINs

One of the powerful abilities of SQL is the ability to combine data to understand relationships and correlate information from different sources.  Much of the JOIN syntax is about expressing how that data should be combined, and how to handle  mismatched data.

But BigQuery needs to figure out how to actually execute it. Because BigQuery came about to help solve the large data processing needs of Google itself - large scale joins were one of the first concerns for the query engine. 

### Hash-Based Joins

When joining two tables on a common key, BigQuery favors a technique called the hash-based join.  With this technique, we can process a table using multiple workers, rather than moving data through a coordinating node.  

![image](/assets/images/hash_joins.png){: style="float: center;width: 60%; margin-left: 40px; margin-bottom: 40px"}


So what actually is hashing? When we hash values, we're converting the input value into a number that falls in a known range. There are many properties of hash functions that we care about for hash joins, but two of the most important are that our function is deterministic (meaning that the same input always yields the same output value) and uniform (our output values are evenly spread throughout the allowed range of values).

With an appropriate hashing function, we can use the output to bucket values.  For example, if our hash function yields an output floating point value between 0 and 1, we can bucket by dividing that key range into N parts, where N is the number of buckets we want. Grouping data based on this hash value means our buckets should have roughly the same number of discrete values, but even more importantly, all duplicate values should end up in the same bucket.

Now that you understand what hashing does, let's talk through joining. To perform the hash join, we're going to split up our work into three stages.

#### Stage 1: Co-locate Data from the First Table

In BigQuery, data for a table is typically split into multiple columnar files, but within those files there's no sorting guarantee that ensures that the columns that represent the join key are sorted and colocated. So, what we do is apply our hashing function to the join key, and based on the buckets we desire we can write rows into different shuffle partitions. It's easiest to think of shuffle partitions as memory-based files for this exercise, though they have advanced features beyond that.

In the diagram, we have three columnar files in the first table, and we've used our hashing technique to split the data into four buckets (color coded).  Once the first stage is complete, the rows of the first table are effectively split into four "file-like" partitions in shuffle, with duplicates colocated.

#### Stage 2: Co-locate Data from the Second Table

This is effectively the same work as the first stage, but we're processing the other table we'll be joining data against.  The important thing to note here is that we need to use the same hashing function and bucket, as we're aligning data.  In the diagram above, the second table has four input files (and thus four worker), and the data is written into a second set of shuffle partitions.


#### Stage 3: Consume the Aligned data and Perform the Koin

After the first two stages are completed, we've aligned the data in the two tables using a common hash function and bucketing strategy. What this means is that we have a set of paired shuffle partitions that correspond to the same hash range, which means that instead of scanning potentially large sets of data, we can execute the join in pieces because each worker is only provided the relevant data for doing its subset of the join.

Now, you can also get a better sense for how important having a good hashing function may be:  if the output values are poorly distributed, we have problems because we're much more likely to have a single worker that's slower and forced to do the majority of the work. Similarly, if we picked our number of buckets poorly, we may have split the work too finely or too coarsely. Fortunately, these are not insurmountable problems, as we can leverage dynamic planning to fix this: we simply insert query stages to adjust the shuffle partitions.

### Broadcast Joins

![image](/assets/images/broadcast_joins.png){: style="float: right;width: 50%; margin-left: 40px; margin-bottom: 40px"}

Hash-based joins are an incredibly powerful technique for joining lots of data, but your data isn't always large enough to warrant it.  For cases where one of the tables is small, we can avoid all the alignment work altogether.

Broadcast joins work in cases where one table is small in size.  In these instances, it's easiest to replicate the small table into shuffle for faster access, and then simply provide a reference to that data for each worker that's responsible for processing the other table's input files.

<a href="https://cloud.google.com/bigquery/query-plan-explanation" class="button">Query Plan Explanation (Docs)</a>
