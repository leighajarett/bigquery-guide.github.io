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

## One common storage project, several different compute projects
With this structure it is common for a centralized data platform team to create pipelines in the storage project to actually ingest data from different sources into BigQuery. Each department or team would then have their own compute projects where they can query the data, save results or create aggregate views. 

![image]({{site.baseurl}}/assets/images/central_storage.png){: style="float: right;width: 50%; margin-left: 40px; margin-bottom: 10px"}

**How it works:** 
- Central data engineering team would be  granted permission to ingest and edit date in the storage project
- Department analysts would be granted viewer access to datasets in the storage project
- Labels on inidivdual datasets with team information can be helpful for monitoring usage
- Department analysts would also be granted data editor and job users access to their department's compute project
- Each compute project would be connected to the team's billing account

**This is especially useful for when:** 
- Each business unit wants to be billed individually for their queries
- There is a centralized platform or data engineering team that ingests data into BigQuery across business units
- Different business units access their data in their own tools or directly in the console 
- You need to avoid too many concurrent queries running in the same project (e.g. On Demand pricing)


## Several different storage projects, one common data warehouse project
With this option, data for each department is ingested into separate projects. Analysts are able to query across these datasets or create aggregate views in a central data warehouse project, which can also easily be connected to a business intelligence tool.

![image]({{site.baseurl}}/assets/images/central_warehouse.png){: style="float: right;width: 50%; margin-left: 40px; margin-bottom: 10px"}

**How it works:** 
- Data engineers who are responsible for ingesting specific data sources are granted edit access to storage projects
- Analysts are granted data viewer access at the project level, for example an HR analyst might be granted view access to the entire HR storage project
- Service accounts that are used to connect BigQuery to external business intelligence tools can be also be granted data viewer access to specific projects that contain datasets to be used in visualizations
- Analysts and Service Account are granted job user and editor permissions in the common data warehouse project 

**This is especially useful for when:** 
- Easier to manage raw data access at the project / department level
- Central analytics team would rather have a single project for compute
- Users are accessing data from a centralized business intelligence tool
- Slots can be assigned to the data warehouse project to handle all queries from analysts and external tools

