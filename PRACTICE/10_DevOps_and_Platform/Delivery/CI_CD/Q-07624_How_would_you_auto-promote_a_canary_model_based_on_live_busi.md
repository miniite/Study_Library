---
type: "practice"
difficulty: "Expert"
---

# How would you auto-promote a canary model based on live business metrics (CTR, conversion) rather than just error rate, and what's the risk of gating on business metrics?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Canary and Progressive Delivery](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Rollback_Strategy/Canary_and_Progressive_Delivery.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To auto-promote a canary model based on live business metrics such as click-through rate (CTR) and conversion rate, we need to set up an automated system that continuously monitors these metrics during the canary deployment phase. The key idea is to define thresholds for CTR and conversion rates that indicate better performance compared to the baseline. If the new model consistently outperforms the old one in terms of these business metrics over a period, it should be auto-promoted.

#### Triaging the Question

The question revolves around automating the process of promoting a machine learning model based on live business metrics like CTR and conversion rate rather than just relying on error rates. The interviewer is probing your understanding of:

Continuous Monitoring: How to set up real-time monitoring for key performance indicators (KPIs).
Performance Metrics: Understanding the nuances between different types of metrics, such as error rate vs. business metrics.
Thresholds and Decision Logic: Defining criteria under which a model should be auto-promoted.
Risks and Trade-offs: Recognizing potential risks associated with gating on these metrics.
Step-by-Step Solution

Setup Continuous Integration/Continuous Deployment (CI/CD) Pipeline:

Implement an automated pipeline where the canary model is deployed in a staging environment or a shadow deployment setup.
Use tools like Kubernetes, Jenkins, GitLab CI/CD, or AWS CodePipeline to manage the deployment and monitoring.

Define Metrics for Monitoring:

Click-Through Rate (CTR): Measure how often users click on a specific element in your application.
Conversion Rate: Track the percentage of users who perform a desired action after clicking through an ad or link.

Collect Real-Time Data:

Use logging and analytics tools like Google Analytics, Mixpanel, or custom-built data pipelines to collect CTR and conversion rate data from the live traffic.
Ensure real-time data collection by integrating with monitoring services such as Prometheus, Grafana, or Datadog.

Set Thresholds for Business Metrics:

Determine acceptable thresholds for CTR and conversion rates based on historical data.
For example, if the current model has a baseline CTR of 2% and a conversion rate of 1%, set new thresholds to be triggered at, say, CTR > 2.5% or Conversion Rate > 1.2%.

Implement Automated Monitoring:

Write scripts or use pre-built monitoring tools to automatically check the metrics against defined thresholds.
For instance, you could write a script that periodically checks these metrics and triggers an alert if they exceed the set thresholds.

Auto-Promotion Logic:

If the canary model consistently performs better than the baseline over a predefined period (e.g., 30 minutes or 1 hour), trigger the auto-promotion process.
This could involve rolling out the canary model to more users while gradually reducing traffic from the previous version.

Risk Assessment:

Bias and Fairness: Ensure that the new model does not introduce any biased behavior, which might harm certain user segments.
User Experience: CTR and conversion rate improvements should translate into a better user experience without causing frustration or confusion.
Operational Risk: Monitor for potential operational issues such as increased latency or resource usage.

#### Closing Summary

In summary, auto-promoting a canary model based on business metrics requires setting up a robust CI/CD pipeline that continuously monitors key performance indicators. Thresholds are defined to trigger the promotion process if the new model outperforms the baseline. However, this approach comes with risks such as potential biases and user experience issues, which need careful consideration.

#### Common Mistakes
Ignoring Error Rate: Over-reliance on business metrics while neglecting error rates can lead to unstable models.
Insufficient Monitoring: Not having a comprehensive monitoring system in place can result in missed opportunities or unnoticed issues.
Over-Automaticity: Automated decisions without human oversight might introduce unforeseen biases or errors.
Self-Check

This solution is both correct and complete, addressing all aspects of the question and potential risks.

