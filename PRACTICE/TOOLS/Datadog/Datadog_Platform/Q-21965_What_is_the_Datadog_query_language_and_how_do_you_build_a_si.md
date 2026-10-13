---
type: "practice"
difficulty: "Intermediate"
---

# What is the Datadog query language and how do you build a simple metric query with it?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Metrics vs Logs Monitoring](<../../../../TOOLS/Datadog/Datadog_Platform/Metrics_vs_Logs_Monitoring.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this answer, I'll cover the basics of the Datadog Query Language (DQL) and demonstrate how to construct a simple metric query using DQL.

TL;DR:

Datadog Query Language (DQL) is used for querying time series data in Datadog. A simple metric query involves selecting specific metrics and applying filters or transformations to get desired results. We'll go through the core concepts and build an example step-by-step.

#### Triaging the Question

The question asks you to explain DQL and create a simple metric query using it. The interviewer is probing your understanding of basic time series data querying, which is crucial for AIOps roles as they often involve monitoring and analyzing system performance metrics.

#### Assumptions
You are familiar with time series data.
You understand basic SQL concepts (though DQL has its own syntax).

#### First-Principles Solution
Step 1: Understanding the Basics of DQL

DQL allows you to query metrics from Datadog’s database, which stores time-series data. The core components are:

Metrics: These represent specific types of monitoring data.
Aggregations: Functions that operate on multiple points in a time series.
Filters and conditions: Used to refine the data you want to query.
Step 2: Basic DQL Syntax

A simple metric query typically involves the following steps:

Selecting metrics using series().
Applying conditions with if or filter.
Aggregating results if necessary with functions like avg, max, etc.
Optionally, formatting the output.
Step 3: Building a Simple Metric Query

Let’s build a query to find the average response time of requests over the past hour for a specific service named "my_service".

DQL
average by host (response_time{service:"my_service"}) over 1h

Here's a breakdown:

response_time: This is the metric name.
{service:"my_service"}: A filter to limit the query results to the specified service.
average by host: Aggregates the data by the "host" dimension and calculates the average response time.
over 1h: Specifies the time range for aggregation.

#### Closing Summary

We have covered the basic structure of a DQL query, explained each component step-by-step, and provided an example. Understanding these concepts is fundamental for using Datadog effectively in AIOps roles.

#### Common Mistakes/Follow-Ups
Common Mistake: Forgetting to specify the time range or aggregation function.

#### Follow-Up
Discussing more advanced query scenarios such as rate calculations, histogram operations, and combining metrics with events.

**Code:**

```
DQL
Copy
average by host (response_time{service:"my_service"}) over 1h
```

