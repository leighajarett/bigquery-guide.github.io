---
title: Workload Management 
layout: default
categories: (1) Resource Concepts
permalink: /resource_concepts/workload_management/
order: 6
description: Now that you understand the reservation model, lets talk through how you can make decisions on workload management for your own organization
next_page_title: 
next_page_permalink: 
prev_page_title: Jobs & Reservation Model
prev_page_permalink:  permalink: /resource_concepts/jobs_reservations/
---

## Choosing a Billing Model

As we discussed in the introduction, BigQuery supports two billing options - On-Demand vs Flat-Rate. It's important to point out that you can switch between these two at any point in time. You can also mix-and-match! The main benefit of flat-rate pricing is that you have price predictability and can increase capacity. 

*You might want to use on-demand pricing when:*
- You value efficiency, you only want to pay for what you use
- You have a project where there is often ad-hoc querying, but you're not super worried about analysts going crazy with queries or you decide to use [custom cost controls](https://cloud.google.com/bigquery/docs/custom-quotas)
- You have a project where predictable workloads run (for example, an ETL job every evening) that don't necessarily need an increased capacity

*You might want to use flat-rate pricing when:*
- You value predictable pricing 
- You have a project where there are often experimental queries and you want to make sure that no one accidentally rings up a huge bill
- You have a project where predictable workloads run (for example, the same dashboards are loaded each morning) and you want to ensure that they meet SLAs or run as quickly as possible with increased capacity

<a href="https://cloud.google.com/blog/products/data-analytics/choosing-bigquery-pricing" class="button">Chooding BQ Pricing (Blog)</a>

## Leveraging a Central Admin Project

Many times organizationst that use BigQuery will create a central admin project that is used to purchase commitments. Those slots can be assigned to any folder or project as discussed in the previous section. However, the commitments themselves will be billed to the admin project. Since slots can be used across the organization, we find that having a central admin project simplifies billing. In the monitoring section we'll go into details on how you can determine who's using what reservations to ensure that different teams' budgets cover their BigQuery compute utilization.

<a href="https://cloud.google.com/bigquery/docs/reservations-workload-management#admin-project" class="button">Reservation Management, Admin Project (Docs)</a>

## Estimating the Number of Slots to Purchase

When getting started with flat-rate pricing, customers often wonder how many slots they should purchase. For brand new BigQuery customers we would recommend starting with the minimum of 500 slots to get predictable pricing. As you begin to use BigQuery more regularly you can monitor the bytes scanned in queries and the slot utilization across workflows to compare pricing against on-demand and determine the optimal sizing for reservations. 

There are a few different ways to monitor the bytes scanned and slots that are being consumed by different workloads. We would recommend either using the [Resource Admin charts](https://cloud.google.com/bigquery/docs/admin-resource-charts), or writing your own queries against the [Information Schema](https://cloud.google.com/bigquery/docs/information-schema-intro) (more details on this are covered in the monitoring section).

### Benchmarking Performance with Flex Slots

When looking at the number of slots consumed, you might try to narrow down the queries that result in the highest spikes. For example, maybe you have a computationally intensive query that is used for reporting each week. You can use flex slots, by simply purchasing the number of slots you want to test for just a few minutes, to experimentally run your query with different amounts of slots available. By benchmarking the time it takes to run the query with different slots, you can optimize the size of the reservation for both price and performance. 

<a href="https://cloud.google.com/bigquery/docs/reservations-workload-management#estimate-slots" class="button">Reservation Management, Estimating Slots (Docs)</a>


## Programmatically Manage Slots
![image](/assets/images/workload_management.png){: style="float: right;width: 50%; margin-left: 40px; margin-bottom: 10px"}

Using the reservation API you can

- Create and delete reservations
- Assign reservations to different projects
- Move slots between reservations

This can be especially helpful if you have predictable workloads that you want to accomodate with slots. For example, lets imagine you have an important data science workload running in project_d at 3am

1. We create a reservation at 3 a.m.
2. Move 1000 slots to the reservation
3. Assign reservation to project_d
4. Delete the reservation at 6 a.m.

Project_d was guaranteed 1000 slots from 3 a.m. to 6 a.m.

<a href="https://cloud.google.com/bigquery/docs/reservations-workload-management#managing_your_workloads_and_departments_using_reservations" class="button">Reservation Management, Workloads & Departments</a>





