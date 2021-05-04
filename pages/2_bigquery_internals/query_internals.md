---
title: Life of a query
layout: default
categories: (2) BigQuery Internals
permalink: /internals/query/
order: 1
description: Here, we’ll give an overview 
next_page_title: 
next_page_permalink: 
prev_page_title: 
prev_page_permalink: 
---

![image](/assets/images/life_of_query.png){: style="width: 70%"}

### API Request Management

- BigQuery supports an asynchronous API for executing queries: callers can insert a query request, and then poll it until complete.  BigQuery supports a REST-based protocol for this, which accepts queries encoded via JSON

- In order to proceed, there’s some level of API processing that must occur. For example, authenticating and authorizing the request, and building and tracking associated request metadata such as the SQL statement, cloud project, and/or query parameters

- Remember from our [previous section on jobs]() that the BigQuery API ensures a query makes forward progress to a terminal DONE state


### Decoding the query text: Lexing and Parsing
Lexing and parsing is a common task for programming languages, and SQL is no different.  

- **Lexing** refers to the process of scanning an array of bytes (the raw SQL statement) and converting that into a series of tokens
- **Parsing** is the process of consuming those tokens to build up a syntactical tree representation of the query that can be validated and understood by BigQuery’s software 

For more insight into this, see the [open source ZetaSQL project](https://github.com/google/zetasql), which provides reference implementations.

### Referencing resources: Catalog Resolution
SQL commonly contains references to entities retained by the BigQuery system, such as tables, views, stored procedures and functions.  In order for BigQuery to process these references, it must resolve them into something more comprehensible to build a query plan.  

### Building a blueprint: Query Planning
BigQuery begins to build a query plan as a more fully-formed picture of the request is exposed via parsing and resolution. Many techniques exist to refactor and improve a query plan to make it faster and more efficient.  Algebraization, for example, converts the parse tree into a form that makes it possible to refactor and simplify subqueries. 

Another aspect of this planning is adapting it to run as a set of distributed execution tasks.  BigQuery leverages large pools of persistent, stateless query computation nodes. So it needs to coordinate how different stages of the query plan share data through reading and writing from storage, and how to stage temporary data within the shuffle system.

### Doing the work: Query Execution
Query execution is the process of working through the query stages in the execution graph.  A query stage may have a single unit of work, or it may be represented by many thousands of units of work, for example reading the data in a large table, which itself may be composed of many separate columnar input files.

### Query management: Scheduling and Dynamic Planning
In addition to the workers that perform the work of the query plan itself, additional workers monitor and direct the overall progress. 

- **Scheduling** is concerned with how aggressively work is queued, executed and completed
- BigQuery also has **dynamic planning capabilities**: A query plan often contains ambiguities, and as a query progresses it may need to make adjustment to ensure success  For example, repartitioning data as it flows through the system, which helps ensure that data is properly balanced and sized for subsequent stages

### Finishing up: Finalizing Results
When a query completes, it often yields results or changes to entities within the system. Finalizing results includes the work to commit these pending changes into the storage layer. The metadata around the query is updated to note the work is done, or the error stream is attached to indicate where things went wrong.


