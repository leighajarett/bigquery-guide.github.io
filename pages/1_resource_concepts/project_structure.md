---
title: Project Structures
layout: default
categories: (1) Resource Concepts
permalink: /resource_concepts/project_structures/
order: 1
description: Here, we’ll go through some common patterns for structuring your BigQuery projects. You can combine these techniques to create a hierarchy that works best for your organization.
next_page_title: Tables
next_page_permalink: /resource_concepts/tables/
prev_page_title: Projects & Datasets
prev_page_permalink: /resource_concepts/projects_datasets/
---

## One data lake to deparment data marts
With this structure, there is a common project that stores raw data in BigQuery, also referred to as a Data Lake. It's common for a centralized data platform team to create pipeline that actually ingest data from different sources into BigQuery within this project. Each department or team would then have their own datamart projects where they can query the data, save results and create aggregate views. 

![image]({{site.baseurl}}/assets/images/central_storage.png){: style="float: right;width: 50%; margin-left: 40px; margin-bottom: 10px"}

**How it works:** 
- Central data engineering team is granted permission to ingest and edit date in the storage project
- Department analysts are granted [BigQuery Data Viewer role](https://cloud.google.com/bigquery/docs/access-control#bigquery) for specific datasets in the storage project
- Department analysts are also granted [BigQuery Data Editor role](https://cloud.google.com/bigquery/docs/access-control#bigquery) and [BigQuery Job User role](https://cloud.google.com/bigquery/docs/access-control#bigquery) for their department's compute project
- Each compute project would be connected to the team's billing account 

**This is especially useful for when:** 
- Each business unit wants to be billed individually for their queries
- There is a centralized platform or data engineering team that ingests data into BigQuery across business units
- Different business units access their data in their own tools or directly in the console 
- You need to avoid too many concurrent queries running in the same project (due to [per-project quotas](https://cloud.google.com/bigquery/quotas#:~:text=You%20can%20run%20up%20to,See%20Cloud%20SQL%20federated%20queries.&text=You%20may%20specify%20limits%20on,query%20by%20setting%20custom%20quotas.&text=Destination%20tables%20in%20a%20query,updates%20per%20table%20per%20day.))


## Department data lakes, one common data warehouse project
With this option, data for each department is ingested into separate projects - essentially giving each department their own data lakr. Analysts are then able to query these datasets or create aggregate views in a central data warehouse project, which can also easily be connected to a business intelligence tool.

![image]({{site.baseurl}}/assets/images/central_warehouse.png){: style="float: right;width: 50%; margin-left: 40px; margin-bottom: 10px"}

**How it works:** 
- Data engineers who are responsible for ingesting specific data sources are granted BQ Data Editor and BQ Job User roles in their designated project
- Analysts are granted BQ Data Viewer role to underlying data at the project level, for example an HR analyst might be granted data viewer access to the entire HR storage project
- Service accounts that are used to connect BigQuery to external business intelligence tools can be also be granted data viewer access to specific projects that contain datasets to be used in visualizations
- Analysts and Service Account are then granted BQ Job User and BQ Data Editor roles in the common data warehouse project 

**This is especially useful for when:** 
- It's easier to manage raw data access at the project / department level
- Central analytics team would rather have a single project for compute
- Users are accessing data from a centralized business intelligence tool
- Slots can be assigned to the data warehouse project to handle all queries from analysts and external tools

## Department data lakes and department data marts
Here, we combine the previous approachaes and create a data lake or storage project for each department. Additionally, each department might have their own datamart project where analysts can run queries.

![image]({{site.baseurl}}/assets/images/department_projects.png){: style="float: right;width: 50%; margin-left: 40px; margin-bottom: 10px"}

**How it works:** 
- Department data engineers are granted BQ Data Editor and BQ Job User roles for their department's data lake project
- Department data analysts are granted BQ Viewer roles for their department's data lake project
- Department data analysts are granted BQ Data Editor and BQ Job User roles for their department's data mart project
- Authorized views are leveraged to give data analysts access to data in projects where they themselves don't have access


**This is especially useful for when:** 
- Each business unit wants to be billed individually both for data storage and compute
- Different business units access their data in their own tools or directly in the console 
- You need to avoid too many concurrent queries running in the same project
- It's easier to manage raw data access at the project / department level
