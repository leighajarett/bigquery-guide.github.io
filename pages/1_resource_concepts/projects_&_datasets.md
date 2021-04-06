---
title: Projects & Datasets
layout: default
categories: (1) Resource Concepts
permalink: /resource_concepts/projects_datasets/
order: 0
description: BigQuery, like other Google Cloud resources, is organized hierarchically where the organization node is the root node, and the projects are the children of the organization. For BigQuery specifically, datasets are descendants of projects and contain your data tables, routines and machine learning models. 
next_page_title: Project Structures
next_page_permalink: /resource_concepts/project_structures/
prev_page_title: Background & Introduction
prev_page_permalink: /introduction/background_introduction/

---
## BigQuery Core Resource Model

<img class="center" src="{{site.baseurl}}/assets/images/resource_model.png" width="100%" height="auto">

## Organizations, Folders and Billing Accounts

![image]({{site.baseurl}}/assets/images/GCP_resource_console.png){: style="float: right;width: 50%; margin-left: 40px; margin-bottom: 10px"}

- [**The Organization**](https://cloud.google.com/resource-manager/docs/cloud-platform-resource-hierarchy#organizations) resource is the root node of the GCP resource hierarchy. It represents a company and is closely associated with your organization’s domain by linking to one [Google Workspace](https://gsuite.google.com/?_ga=2.44366563.637813095.1617627706-1731652653.1615906420) or [Cloud Identity account](https://cloud.google.com/identity)..

    - While an Organization is not required to get started using BigQuery, it is recommended. With an Organization resource, projects can be centrally controlled by administrators instead of solely being owned by the employee who created them

- [**Folders**](https://cloud.google.com/resource-manager/docs/cloud-platform-resource-hierarchy#folders) are an additional grouping mechanism on top of Projects. They can be seen as sub-organizations within the Organization. Folders can be used to model different legal entities, departments, and teams within a company. 

    - Folders act as a [policy inheritance](https://cloud.google.com/resource-manager/docs/cloud-platform-resource-hierarchy#inheritance) point - IAM roles granted on a folder are automatically inherited by all Projects and folders included in that folder
    
    - For BigQuery flat-rate customers, [slots](https://cloud.google.com/bigquery/docs/slots) (units of CPU) can be assigned to Organizations, Folders or Projects where they are distributed fairly among projects to handle job workloads

- [**A Billing Account**](https://cloud.google.com/billing/docs/concepts) is required to use BigQuery, unless you are using the BigQuery sandbox. Many times, different teams will want to be billed individually for consuming resources in Google Cloud. Therefore, each billing group will have its own billing account, which results in a single invoice and is tied to a Google Payments profile. 

<a href="https://cloud.google.com/resource-manager/docs/cloud-platform-resource-hierarchy" class="button">Google Cloud Resource Hierarchy Docs</a>
<a href="https://cloud.google.com/docs/enterprise/best-practices-for-enterprise-organizations#define-hierarchy" class="button">Best Practices for Google Cloud Resource Hierarchy</a>

## Projects
![image]({{site.baseurl}}/assets/images/BQ_Console_Projects.png){: style="float: right;width: 50%; margin-left: 40px; margin-bottom: 10px"} 

A [Project](https://cloud.google.com/billing/docs/concepts#projects) is required to use BigQuery, and forms the basis for creating, enabling, and using all Google Cloud services, including managing APIs, enabling billing, adding and removing collaborators, and managing permissions.

- **Storage & Compute:** A project is used both for storing data and for running jobs (e.g. querying). And because storage and compute are separate, these don’t need to be the same project. You can store your data in one project and query it from another, this includes combining data stored in multiple projects in a single query

- **Billing:** A project can have only one billing account, the project will be billed for data stored in the project as well as jobs run in the project

- **Concurrency Limits:** If you are using an on-demand pricing model, you are limited to 100 concurrent queries per project, [learn more about other per-project limits here](https://cloud.google.com/bigquery/quotas)

- **Slots:** [Reservations](https://cloud.google.com/bigquery/docs/reservations-intro), or reserved slots, can be assigned to specific Projects (or more broadly at the Organization and Folder levels) so that workloads running there will have designated amounts of CPUs to leverage

- **Access controls:** You can grant permissions for accessing data at the project level 

<button href="https://cloud.google.com/resource-manager/docs/creating-managing-projects">Creating and Managing Projects</button>
<button href="https://cloud.google.com/resource-manager/docs/access-control-proj">Access Controls for Projects</button>

## Datasets
![image]({{site.baseurl}}/assets/images/BQ_Console_Datasets.png){: style="float: right;width: 50%; margin-left: 40px; margin-bottom: 10px"} 

[Datasets](https://cloud.google.com/bigquery/docs/datasets-intro) are top-level containers, within a Project, that are used to organize and control access to your tables and views. A table or view must belong to a dataset, so you *need to create at least one dataset before loading data* into BigQuery.

- **Location:** Your data will be stored in the [geographic location](https://cloud.google.com/bigquery/docs/locations) that you chose at the dataset's creation time. After a dataset has been created, the location can't be changed. One important consideration is that you will not be able to query across multiple locations.

    - Many users chose to store their data in a multi-region location like *us*, however some chose to set a specific region that is close to on-premise databases or ETL jobs

- **Metadata management:** Datasets can have descriptions and [labels]() (key:value pairs) which are helpful to making data accessible and monitoring usage. Descriptions and labels can be viewed in the [console, through the API and in the information schema](https://cloud.google.com/bigquery/docs/dataset-metadata). 

- **Size limits:** Datasets do not have fixed size limits. However, high resource cardinality can cause latency (e.g. millions of tables). 

- **Access controls:** You can control access at the dataset level


<button href="https://cloud.google.com/bigquery/docs/datasets">Creating Datasets</button>
<button href="https://cloud.google.com/bigquery/docs/managing-datasets">Managing Datasets</button>
<button href="https://cloud.google.com/bigquery/docs/dataset-access-controls ">Access Controls for Datasets</button>
