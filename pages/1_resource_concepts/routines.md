---
title: Routines (UDFs & Procedures) & ML Models
layout: default
categories: (1) Resource Concepts
permalink: /resource_concepts/routines/
order: 3
description: Routines and Machine Learning models are resources within datasets. Routines allow you to reuse functions and procedures for handlng data in a unique way. Models allow you to make predictions using built in machine learning functionality.
next_page_title: Project Structures
next_page_permalink: /resource_concepts/project_structures/
prev_page_title: Background & Introduction
prev_page_permalink: /introduction/background_introduction/
---

In BigQuery, a routine is either a user defined function (UDF) or a procedure. Routines are a resource belonging to a dataset, you can grant users the BigQuery Viewer role on a routine so that they can leverage them within their own queries. 

### User Defined Functions (UDF)
A UDF is a function that is created using either SQL or Javascript, it takes arguments as input and returns a single value as an output. 

![image](/assets/images/UDFs.png){: style="float: right;width: 50%; margin-left: 40px; margin-bottom: 10px"}


- UDFs allow you to handle data in a unique way - for example, pulling out useful parameterms from a URL or cleansing strings

- UDFs can either be persistent or temporary:
    - You can reuse persistent UDFs across multiple queries and share them to be used across your team
    ```
    CREATE OR REPLACE FUNCTION
    my_dataset.cleanse_string_test (text STRING)
    RETURNS STRING
    AS (REGEXP_REPLACE(LOWER(TRIM(text)), '[^a-zA-Z0-9 ]+', ''));
    ```
    
    - Temporary UDFs are only valid for a single query
    ```
    CREATE TEMP FUNCTION cleanse_string (text STRING)
        RETURNS STRING
        AS (REGEXP_REPLACE(LOWER(TRIM(text)), '[^a-zA-Z0-9 ]+', ''));
    SELECT text
        , cleanse_string(text) AS clean_text
    FROM strings;
    ```

- For Javascript functions, you can’t import modules but you can reference library files stored in your Google Cloud Storage bucket

- An authorized UDF is a UDF that is authorized to access a particular dataset: the UDF can query tables in the dataset, even if the user who calls the UDF does not have access to those tables, meaning you can share results with particular users or groups without giving those users or groups access to the underlying tables

<a href="https://cloud.google.com/bigquery/docs/reference/standard-sql/user-defined-functions?utm_source=youtube&utm_medium=unpaidsoc&utm_campaign=CDR_ali_analytics_c3dtglwrycs_BigQuerySpotlight_111220&utm_content=description" class="button">UDFs (docs)</a>
<a href="https://hoffa.medium.com/new-in-bigquery-persistent-udfs-c9ea4100fd83" class="button">Examples of UDFs</a>
<a href="https://www.youtube.com/watch?v=c3dtgLWRycs" class="button">UDFs (vid)</a>


### [Stored] Procedures 
Stored procedures are blocks of SQL statements that can be called from other queries. Unlike UDFs, stored procedures can return multiple values or no values - which means you can run them to create or modify tables.

![image](/assets/images/procedures.png){: style="float: right;width: 50%; margin-left: 40px; margin-bottom: 10px"}


- Procedures contain statement_lists (statements separated by a semicolon), meaning you can run multiple statements - or multiple queries - within one procedure

- In BigQuery, procedures give you scripting capabilities, allowing you to control the execution flow with if and while statements

- Exceptions can be handled with a begin...exception block 

- With [EXECUTE_IMMEDIATE](https://cloud.google.com/bigquery/docs/reference/standard-sql/scripting#execute_immediate) you can also use procedures to dynamically create and run SQL queries, for example - if we want to dynamically query a table where each date value is its own column [as described here](https://towardsdatascience.com/how-to-use-dynamic-sql-in-bigquery-8c04dcc0f0de)


    ```
    CREATE OR REPLACE PROCEDURE testing.sum_sales(startDate STRING, endDate STRING)
    BEGIN
    CREATE OR REPLACE TABLE testing.sales_result
    AS (SELECT 
            date(t.transaction_timestamp) as date, 
            sum(li.sale_price) as total_sales
        FROM `leigha-bq-dev.retail.transaction_detail` as t
        LEFT JOIN UNNEST(t.line_items) as li
        WHERE transaction_timestamp >= TIMESTAMP(startDate) AND transaction_timestamp <= TIMESTAMP(endDate) 
        GROUP BY 1);
    END;

    CALL testing.sum_sales('2020-08-01', '2020-01-20');
    ```


<a href="https://cloud.google.com/bigquery/docs/reference/standard-sql/data-definition-language#create_procedure" class="button">Create Prcoedures (docs)</a>
<a href="https://www.google.com/search?q=bugquery+procedures&oq=bugquery+procedures&aqs=chrome..69i57j0i13l2j0i22i30l2j69i60l3.1913j0j4&sourceid=chrome&ie=UTF-8" class="button">Scripting (docs)</a>
<a href="https://hoffa.medium.com/easy-pivot-in-bigquery-one-step-5a1f13c6c710" class="button">Ex. Procedure that Pivots Table</a>



### UDF vs. Procedure
- Procedures can return no values or multiple values, whereas functions must always return a single value

- Procedures can have input and output parameters, functions can have only input parameters

- Procedures allow for [DDL (Data Definition Language)](https://cloud.google.com/bigquery/docs/reference/standard-sql/data-definition-language) & [DML (Data Manipulation Language) statements](https://cloud.google.com/bigquery/docs/reference/standard-sql/data-manipulation-language)

- Functions can be called from procedure whereas procedures cannot be called from function

- UDFs can be used in the SQL statements anywhere in the WHERE/HAVING/SELECT section, procedures cannot

### Creating an Organization Wide Function & Procedure Library
- One of the main benefits of routines is that you can create a function or procedure once, store it in BigQuery, and call it any number of times across your organization - ensuring single-source-of-truth business logic  and reusability across teams

- Many BigQuery users chose to create a designated dataset to act as a library for persisent UDFs and procedures

- This approach ensures that everyone in the business is applying consistent logic to their analyses

- You can grant all analysts the BigQuery Viewer role on this dataset so that they can leverage the UDFs in any queries they write in their own projects.

- You can also add descriptions so that everyone understands exactly how the routine is functioning 


### Machine Learning Models

A model is a resource that exists inside of a dataset, it represents a machine learning model that you create within BigQuery and is trained usng the query you provide when declaring the model definition. 

```
CREATE MODEL
  `mydataset.mymodel`
OPTIONS
  ( MODEL_TYPE='KMEANS',
    NUM_CLUSTERS=4 ) AS
SELECT
  *
FROM `mydataset.mytable`
```

- Once you have created your model you can leverage it to make predictions with predict functions like ML.PREDICT, ML.FORECAST and ML.RECOMMEND

- You can also use evaluation functions like ML.EVALUATE, ML.ROC_CURVE, ML.CONFUSION_MATRIX and ML. to understand how the model is performing 

- Model and feature inspection functions allow you to understand the details of your model - for example, gathering information about centroids with ML.CENTROIDS for a clustering model


<a href="https://cloud.google.com/bigquery-ml/docs/reference" class="button">BigQuery ML Reference (docs)</a>
<a href="https://www.qwiklabs.com/focuses/2157?parent=catalog" class="button">Getting Started with BQML (tutorial)</a>




