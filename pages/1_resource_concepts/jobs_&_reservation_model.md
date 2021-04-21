---
title: Jobs & Reservation Model
layout: default
categories: (1) Resource Concepts
permalink: /resource_concepts/jobs_reservations/
order: 5
description: BigQuery leverages jobs to perform actions within your data warehouse
next_page_title: 
next_page_permalink: 
prev_page_title: Machine Learning Models
prev_page_permalink: /resource_concepts/ml_models/
---

## Jobs
A job is an action that BigQuery runs on your behalf. It is a resource within a project. The data you are copying, querying or exporting may be stored in a different BigQuery project than where the job itself lives. 

![image](/assets/images/jobs.png){: style="float: right;width: 50%; margin-left: 40px; margin-bottom: 10px"}

- There are several different job types:

    - **Load:** A load job ingests data from a POST request, Google Cloud Storage or other sources to create a managed table

    - **Query:** Invoke the query engine to execute a SQL query.  This includes SELECT statements, DML, DDL, and scripts (as well as procedure calls).
    
    - **Copy:** Direct data copy.  Moves committed data from one (or more) source tables to a destination table.

    - **Export:** Write the contents of a table out to Cloud Storage using the specified format and options.


- Other actions like listing resources or getting metadata about resources are *not* managed by a job

- When you use the **console or the bq command-line** tool to load, query, copy or export data - a job resource is automatically created, scheduled and run

- When you use the **API and client libraries**, you programmatically create jobs, which BigQuery will then schedule and run for you

- Because jobs have the potential to take a long time to complete, BigQuery executes them <u>asynchronously</u> - meaning each job is run independently, and we can execute multiple jobs at a time without needing to finish the current job to move on to the next one

- Using the Job ID, you can share a link so that other BigQuery users can view information about the job in the console- [details here](https://stackoverflow.com/questions/67104653/is-it-possible-to-link-to-a-job-in-the-bigquery-console/67108721#67108721), alternatively you can use the [share query button in the console](https://cloud.google.com/bigquery/docs/saving-sharing-queries) to get a unique URL for sharing just the query

- A Job is tied to both a user (who ran the job, can be a [service account](https://cloud.google.com/iam/docs/service-accounts)) and a [location](https://cloud.google.com/bigquery/docs/locations#specifying_your_location) which is where the job is run
    
    - BigQuery determines the location to run the job based on the datasets referenced in the request. You can also explicity set the location for your job, but it must be in the same location as any datasets that are being used


### Job Status [State]
Jobs are guaranteed to make forward progress to a Done state. You can poll the job for state as it progresses - either through the API or by checking the status in the query history panel or job history panel. 

![image](/assets/images/job_states.png){: style="width: 70%"}

For example, using the Python client library we can list the Job IDs, states and types for the last 10 jobs that were run in our project. 

```
client = bigquery.Client(project="my-project")

for job in client.list_jobs(max_results=10):  # API request(s)
    print(job.job_id, job.state, job.job_type)
```
<a href="https://cloud.google.com/bigquery/docs/jobs-overview" class="button">Running & Managing Jobs (docs)</a>

## Reservation Model

![image](/assets/images/reservation_model.png){: style="float: right;width: 50%; margin-left: 40px; margin-bottom: 10px"}

There are several different resources related to the reservation model, each one is within a project and a specific **location**:

- [**Capacity Commitment**](https://cloud.google.com/bigquery/docs/reservations-intro#commitments): a purchase of BigQuery *slots*, made by flat rate customers (pricing models covered in the [intro]())

    - [**Slot**](https://cloud.google.com/bigquery/docs/slots): BigQuery unit of computational capacity, on-demand customers get access to 2,000 slots per project

    - BigQuery offers several commitment plans to choose from:

        - **Monthly commitment**: minimum 30-day commitment
        
        - **Annual commitment**: you purchas 365-day commitment which can be auto-renewed
        
        - **Flex slots**: you purchase a 60-second commitment. Useful for testing how your workloads perform with flat-rate billing, before purchasing a longer-term commitment

- [**Reservation**](https://cloud.google.com/bigquery/docs/reservations-intro#reservations): Bucket of slots, can be allocated in different ways that make sense for your organization (for example a *prod* reservation for production )

- [**Assignment**](https://cloud.google.com/bigquery/docs/reservations-intro#assignments): To use slots, you assign a reservations to projects, folders or organizations

    - Each level in the resource hierarchy inherits the assignment from the level above it, unless you override it, and slots are [distributed fairly among projects and jobs](https://cloud.google.com/bigquery/docs/reservations-intro#slot_scheduling)

    - When you create an assignment you specify the type of job that the assignment will be used for: either **QUERY** for all query jobs, **PIPELINE** for extract, copy and load jobs, or **ML_EXTERNAL** for BQML queries that use external machine learning resources

        - By default, load and export jobs are free and use a shared pool of slots, but BigQuery makes no guarantees on available slots. When jobs are assigned to a PIPELINE reservation, they lose access to the free pool. So make sure that jobs have enough capacity otherwise, performance could  be worse than using the free pool.

    - Assignments will only be used for jobs that are run in the same location

<a href="https://cloud.google.com/bigquery/pricing#flat_rate_pricing" class="button">Flat Rate Pricing (docs)</a>
<a href="https://cloud.google.com/bigquery/docs/reservations-intro" class="button">Reservations for Workload Management (docs)</a>

## BI Engine Reservation

BigQuery BI Engine is an in-memory analysis service that intellignetly caches's data stored in BigQuery for fast and fresh dashboards

- To leverage BI engine you create a BI engine reservation

    - While standard BigQuery reservations manage compute resources, BI Engine reservations manage memory resource - they are measured in GB

    - BI Engine reservations also belong to a project and are specific to the location that it was created in


<a href="https://cloud.google.com/bi-engine/docs/introduction" class="button">What is BI Engine? (docs)</a>
<a href="https://cloud.google.com/bi-engine/docs/reserving-capacity" class="button">Reserving BI Engine Capacity (docs)</a>
