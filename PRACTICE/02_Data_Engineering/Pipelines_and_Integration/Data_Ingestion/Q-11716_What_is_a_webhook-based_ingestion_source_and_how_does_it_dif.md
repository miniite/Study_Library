---
type: "practice"
difficulty: "Intermediate"
---

# What is a webhook-based ingestion source and how does it differ from polling?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ CDC vs Full Reload](<../../../../02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/CDC_vs_Full_Reload.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: A webhook-based data ingestion source sends real-time notifications when new data is available, whereas polling involves periodically checking a server or API for updates. The key difference lies in the timing and efficiency of data retrieval.

Webhooks are a method of communication where an application can send a notification to another application via HTTP POST request whenever certain events occur, such as data being updated or inserted into a database. This is particularly useful in scenarios requiring real-time data processing without constant polling.

In contrast, polling involves periodically sending requests to an API or server to check for new data. While straightforward and easy to implement, it can be inefficient and resource-intensive if the data doesn't change frequently.

To summarize, webhooks are more efficient for real-time data ingestion by reacting directly to events, while polling is a simple yet less efficient approach that continually checks for updates.

#### Triage
The question asks us to explain what a webhook-based ingestion source is and how it differs from polling. Specifically, the interviewer might be looking for an understanding of:

The concept of real-time data retrieval using webhooks.
The mechanics of webhook notifications.
How polling works in comparison.

#### Assumptions
include that you have a basic understanding of HTTP requests and APIs.

Step-by-First Principles Solution:

Webhook Concept:

Webhooks are event-driven mechanisms where one application sends an HTTP request to another application when specific events occur, such as new data being added to a database.
They allow the receiving application (sink) to be notified and act on those events without needing to periodically check for updates.

Webhook Mechanics:

A webhook is set up by configuring an endpoint in the source system (e.g., a database, API). This endpoint is essentially a URL that can receive HTTP POST requests.
When data changes or certain conditions are met, the source system sends a POST request to the configured webhook URL with relevant data.
The sink application listens for these POST requests and processes them accordingly.

Polling Mechanism:

Polling involves the source periodically sending GET or POST requests to an API or server endpoint to check for new data.
This approach is more like a "constant tapping" on the door, where you keep checking until something changes.

Efficiency and Use Cases:

Webhooks are ideal for scenarios requiring real-time processing of events as they happen. For example, in e-commerce platforms where order status updates need to be immediately reflected elsewhere.
Polling is useful when you cannot control the source system or when setting up webhooks is not feasible due to technical constraints.

#### Example

Imagine a customer management system that uses Airbyte for data ingestion. If the system adds new customers, it can send a webhook notification to your ETL pipeline.
The ETL pipeline will receive this POST request and process the new customer details immediately, ensuring real-time updates.

Comparison and Trade-offs:

Webhooks reduce unnecessary network calls by only sending data when needed, which is more efficient.
Polling might be simpler to implement but can lead to wasted resources if you poll too frequently or if no changes are made between requests.

#### Conclusion

For real-time data ingestion, webhooks provide a much more efficient and responsive mechanism compared to polling.
However, for environments where the source system cannot support webhooks, traditional polling might be necessary despite its drawbacks.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

Webhooks are an excellent choice when you need real-time data updates and can configure them in your systems.
Polling is simpler but less efficient; it’s useful as a fallback or for scenarios where real-time updates aren’t critical.

#### Common mistakes
include underestimating the network overhead of frequent polling, neglecting to handle webhook validation securely, and not properly setting up error handling.

#### Follow-up
questions might explore:

How you would set up webhooks in practice?
What security measures are necessary when using webhooks?
How do you ensure reliable data processing with both methods?

**Comparison:**

```
Method	Description
Webhook	Event-driven communication where the source sends data directly to the target application.
Polling	Periodic checking of a server or API for updates, which can be inefficient if no changes occur frequently.
```

**Example input / output:**

```text
Input:
POST /webhook HTTP/1.1
Host: localhost:5000
Content-Type: application/json

{
  "customer_id": 42,
  "name": "John Doe",
  "email": "john.doe@example.com"
}

Output:
Received webhook: {'customer_id': 42, 'name': 'John Doe', 'email': 'john.doe@example.com'}
```

**Flow:**

1. Webhook Ingestion
2. Is there an event?
3. Send HTTP POST to webhookURL
4. No event - continueprocessing
5. Process data in sinkapplication
6. Polling Ingestion
7. Periodically send request toAPI/endpoint
8. Check for new data
9. Is there new data?
10. No - retry after timeout
11. Yes - process data in sinkapplication

