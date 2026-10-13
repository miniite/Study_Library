---
type: "practice"
difficulty: "Intermediate"
---

# How would you use Python to measure the round-trip latency of an HTTP health check and store it as a metric sample?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ OpenTelemetry Instrumentation](<../../../../TOOLS/Python/Python_Language/OpenTelemetry_Instrumentation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To measure the round-trip latency of an HTTP health check using Python, we need to send an HTTP request to a specified endpoint, capture the time taken for the response, and then store this value as a metric sample. This task involves understanding network requests, timing operations, and data storage in Python.

#### Triage

The core question is: "How do you measure the round-trip latency of an HTTP health check using Python?" The interviewer is probing your knowledge of making HTTP requests, handling time measurements, and storing the result as a metric. Key assumptions include having access to the target URL and a suitable method for storing metrics.

#### First-Principles Solution
Send the HTTP Request: Use requests library to send an HTTP GET request.
Measure Time: Record the start time before sending the request, and record the end time once the response is received.
Calculate Latency: Subtract the start time from the end time to get the latency.
Store Metric Sample: Use a suitable backend (e.g., InfluxDB for Prometheus metrics) to store this value.

#### Implementation

We will use requests for making HTTP requests and time for timing operations.

PYTHON
import requests
import time

# Define the URL of the health check endpoint
url = "http://example.com/health"

def measure_latency(url):
    # Start time before sending the request
    start_time = time.time()

    try:
        response = requests.get(url)
        if response.status_code == 200:  # Assuming a healthy status code is 200
            end_time = time.time()
            latency = (end_time - start_time) * 1000  # Latency in milliseconds
            print(f"Latency: {latency} ms")
            return latency
        else:
            print("Health check failed with status code:", response.status_code)
    except requests.RequestException as e:
        print("Request failed:", e)

# Example usage
measure_latency(url)
Storing the Metric Sample

For storing the metric, we can use a library like prometheus_client to export the data in Prometheus format and push it to an InfluxDB instance.

PYTHON
from prometheus_client import start_http_server, Gauge
import time

# Initialize a gauge for latency metrics
latency_gauge = Gauge('http_healthcheck_latency', 'Round-trip latency of HTTP health check')

def store_metric(latency):
    # Store the metric in Prometheus format
    latency_gauge.set(latency)

# Example usage with storing
measure_latency(url)
store_metric(measure_latency(url))
Full Solution

Combining both parts, we get:

PYTHON
import requests
import time
from prometheus_client import start_http_server, Gauge

# Define the URL of the health check endpoint
url = "http://example.com/health"

def measure_and_store_latency(url):
    # Start time before sending the request
    start_time = time.time()

    try:
        response = requests.get(url)
        if response.status_code == 200:  # Assuming a healthy status code is 200
            end_time = time.time()
            latency = (end_time - start_time) * 1000  # Latency in milliseconds

            # Print and store the metric
            print(f"Latency: {latency} ms")
            return latency
        else:
            print("Health check failed with status code:", response.status_code)
    except requests.RequestException as e:
        print("Request failed:", e)

# Initialize a gauge for latency metrics
start_http_server(8000)  # Prometheus server starts on port 8000
latency_gauge = Gauge('http_healthcheck_latency', 'Round-trip latency of HTTP health check')

def store_metric(latency):
    # Store the metric in Prometheus format
    latency_gauge.set(latency)

# Example usage with storing
measure_and_store_latency(url)
store_metric(measure_and_store_latency(url))

#### Closing Summary
and Common Mistakes/Follow-Ups
Ensure you handle exceptions properly, especially network-related errors.
Use meaningful variable names to make the code more readable.

#### Cons
ider adding logging or error handling for production use.
If running in a containerized environment, consider using Docker Compose or Kubernetes for service discovery.

#### Common mistakes
include not handling HTTP status codes correctly, not accounting for potential network issues, and improper metric storage. Follow-ups might involve discussing alternative methods (e.g., async request handling with aiohttp) or how to scale this solution in a production environment.

**Comparison:**

```
HTTP Status Code	Meaning
200	OK
404	Not Found
500	Internal Server Error
503	Service Unavailable
```

**Code:**

```

import requests import time

Define the URL of the health check endpoint

url = "http://example.com/health"

def measure_latency(url): # Start time before sending the request start_time = time.time()

CODE
Copy
try:
    response = requests.get(url)
    if response.status_code == 200:  # Assuming a healthy status code is 200
        end_time = time.time()
        latency = (end_time - start_time) * 1000  # Latency in milliseconds
        print(f"Latency: {latency} ms")
        return latency
    else:
        print("Health check failed with status code:", response.status_code)
except requests.RequestException as e:
    print("Request failed:", e)
Example usage

measure_latency(url)

CODE
Copy

```python
from prometheus_client import start_http_server, Gauge

# Initialize a gauge for latency metrics
start_http_server(8000)  # Prometheus server starts on port 8000
latency_gauge = Gauge('http_healthcheck_latency', 'Round-trip latency of HTTP health check')

def store_metric(latency):
    # Store the metric in Prometheus format
    latency_gauge.set(latency)
PYTHON
Copy
import requests
import time
from prometheus_client import start_http_server, Gauge

# Define the URL of the health check endpoint
url = "http://example.com/health"

def measure_and_store_latency(url):
    # Start time before sending the request
    start_time = time.time()

    try:
        response = requests.get(url)
        if response.status_code == 200:  # Assuming a healthy status code is 200
            end_time = time.time()
            latency = (end_time - start_time) * 1000  # Latency in milliseconds

            # Print and store the metric
            print(f"Latency: {latency} ms")
            return latency
        else:
            print("Health check failed with status code:", response.status_code)
    except requests.RequestException as e:
        print("Request failed:", e)

# Example usage with storing
measure_and_store_latency(url)
store_metric(measure_and_store_latency(url))
```

**Example input / output:**

```text
Input:
- URL: "http://example.com/health"

Output:
Latency: 150.3 ms

Store Metric Sample:
Metric Name: http_healthcheck_latency
Value: 150.3
Timestamp: <current timestamp>
```

**Data:**

```yaml
url: http://example.com/health
status_code: 200
latency: 150.3 ms
```

**Flow:**

1. Yes
2. No
3. Send HTTP GET request
4. Measure start time
5. Receive response
6. Measure end time
7. Response status == 200?
8. Calculate latency
9. Print error message and exit
10. Store metric sample

