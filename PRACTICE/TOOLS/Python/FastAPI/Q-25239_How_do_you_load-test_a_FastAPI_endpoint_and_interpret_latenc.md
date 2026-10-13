---
type: "practice"
difficulty: "Deep"
---

# How do you load-test a FastAPI endpoint and interpret latency percentiles like p95 and p99?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Real-time and Batch Inference APIs](<../../../../TOOLS/Python/FastAPI/Real-time_and_Batch_Inference_APIs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To effectively load-test a FastAPI endpoint and understand its performance metrics such as p95 and p99, we need to follow a structured approach that involves setting up the test environment, choosing appropriate tools, executing tests, and analyzing results. This process ensures not only the identification of bottlenecks but also helps in making informed decisions about scaling and optimizing the application.

Firstly, we need to understand what load testing is: it simulates real-world usage scenarios to measure system performance under anticipated workloads. In this case, FastAPI endpoints are critical for backend services handling various types of requests. We will use tools like locust or Apache JMeter to simulate users and send traffic to the endpoint.

#### Assumptions

The application is running on a local development environment.
Network conditions between the testing machine and the application are stable.
We have access to the necessary credentials and API keys for authentication, if required.

Let's break down the process:

Setup Testing Environment:

Install locust or Apache JMeter. For this example, we'll use Locust.
SH
pip install locust

Define Test Users (Locust):

Define a simple user behavior to represent typical user interactions with the FastAPI endpoint.
PYTHON
from locust import HttpUser, between, task

class QuickstartUser(HttpUser):
    wait_time = between(1, 5)

    @task
    def index_page(self):
        self.client.get("/api/endpoint")

Run Load Test:

Start the Locust test runner with a specified number of users and hatches per second.
SH
locust --host=http://localhost:8000 --web-host=127.0.0.1 --web-port=8089 -f locustfile.py

Interpret Results:

Navigate to the Locust web interface (e.g., http://127.0.0.1:8089) and observe the test results.
Look at the latency percentiles, specifically p95 and p99.

Latency Percentiles:

p95 Latency: 95% of all requests are handled within this time frame. This is a good measure to ensure that most users experience acceptable performance.
p99 Latency: 99% of all requests are handled within this time frame. This is more stringent and ensures that only the smallest percentage of requests are delayed.
Analyze Performance:
If p95 or p99 latency exceeds acceptable thresholds, investigate potential causes such as database queries, network issues, or bottlenecks in your application code.

#### Cons
ider scaling strategies like adding more resources to the server, improving database query performance, or implementing caching mechanisms.

#### Closing Summary

Load testing with FastAPI involves setting up a testing environment, defining user behaviors, running tests, and analyzing latency percentiles to ensure optimal performance.
Understanding p95 and p99 helps in identifying critical issues that impact end-user experience.

#### Common Mistakes/Follow-Ups

Failing to set realistic load conditions can lead to inaccurate test results.
Not closely monitoring the application under stress may result in unnoticed issues.
Overlooking the importance of understanding the business context when setting performance thresholds.

