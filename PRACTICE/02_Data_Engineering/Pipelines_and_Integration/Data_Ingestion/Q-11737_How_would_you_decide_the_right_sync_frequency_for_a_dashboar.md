---
type: "practice"
difficulty: "Deep"
---

# How would you decide the right sync frequency for a dashboard that needs near-real-time data?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Incremental Sync and Connector Migration](<../../../../02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Incremental_Sync_and_Connector_Migration.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide the right sync frequency for a dashboard that requires near-real-time data, we need to strike a balance between data freshness and system performance. The key is understanding the nature of the data and the business requirements. This involves evaluating the latency tolerance, the volume and velocity of data, and the processing capabilities of both the source and target systems.

First, let's define what "near-real-time" means in context. Typically, it ranges from seconds to a few minutes. For example, if your dashboard needs to reflect changes within 1 minute of occurrence, that sets our target latency. Next, we need to consider the data sources and their characteristics, such as whether they are high-frequency or low-frequency streams.

Now, let's reason step-by-step:

Evaluate Data Source Characteristics:

High-frequency data (e.g., sensor readings) might require more frequent syncs.
Low-frequency data (e.g., periodic reports) can be synced less often without significantly impacting dashboard accuracy.

Assess Business Requirements and SLAs:

The business might have specific time windows during which the most critical data needs to be refreshed. For example, if a financial report is required every 5 minutes, we need to ensure that the sync frequency aligns with this requirement.

#### Cons
ider the impact on downstream systems or other services that depend on the freshness of the data.

#### Cons
ider System Performance and Scalability:

Frequent syncs can increase load on both source and target systems, potentially causing performance issues.
Evaluate the capacity of your infrastructure to handle more frequent updates without degradation in service quality.

Pilot and Iterate:

Start with a conservative frequency and monitor the dashboard's accuracy and system performance.
Gradually adjust the sync frequency based on observed behavior, ensuring that any changes are carefully monitored for their impact.

Use Monitoring and Alerts:

Implement real-time monitoring to catch issues early and ensure data integrity.
Set up alerts for critical conditions such as data latency breaches or system performance degradation.

To illustrate this process, let's consider a scenario where we have a high-frequency stock price feed that needs to be updated every 5 seconds. Here is how you might approach the decision:

PYTHON
# Example code snippet to simulate sync frequency tuning
import time

def update_dashboard(data_feed):
    # Simulate updating the dashboard with new data
    print("Updating dashboard with new data")

def monitor_performance(target_latency):
    current_time = time.time()
    next_update_time = current_time + target_latency

    while True:
        if time.time() >= next_update_time:
            update_dashboard(data_feed)
            next_update_time += target_latency

# Example high-frequency feed
data_feed = {"stock_price": 100.5, "timestamp": time.time()}
target_latency = 5  # seconds

monitor_performance(target_latency)

This example demonstrates a simple loop that updates the dashboard every target_latency seconds.

Finally, it's crucial to document your decisions and monitor the system closely to ensure that the chosen sync frequency meets both business needs and technical constraints.

**Comparison:**

```
Time	Stock Price
14:30:00Z	100.5
14:30:05Z	100.6
```

**Code:**

```python
# Example code snippet to simulate sync frequency tuning
import time

def update_dashboard(data_feed):
    # Simulate updating the dashboard with new data
    print("Updating dashboard with new data")

def monitor_performance(target_latency):
    current_time = time.time()
    next_update_time = current_time + target_latency

    while True:
        if time.time() >= next_update_time:
            update_dashboard(data_feed)
            next_update_time += target_latency

# Example high-frequency feed
data_feed = {"stock_price": 100.5, "timestamp": time.time()}
target_latency = 5  # seconds

monitor_performance(target_latency)
```

**Example input / output:**

```

Initial input:

JSON
Copy
{
    "high_frequency_feed_example": {
        "stock_price": 100.5,
        "timestamp": "2023-10-05T14:30:00Z"
    }
}

Output after processing (every 5 seconds):

JSON
Copy
{
    "dashboard_update_timestamp": "2023-10-05T14:30:05Z",
    "stock_price": 100.6
}
```

**Data:**

```json
{
    "high_frequency_feed_example": {
        "stock_price": 100.5,
        "timestamp": "2023-10-05T14:30:00Z"
    }
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is data high-frequency?
5. Set sync frequency to every 5seconds
6. Set sync frequency based onbusiness requirements
7. Monitor system performanceand adjust if needed
8. Document decisions andmonitor continuously
9. End

