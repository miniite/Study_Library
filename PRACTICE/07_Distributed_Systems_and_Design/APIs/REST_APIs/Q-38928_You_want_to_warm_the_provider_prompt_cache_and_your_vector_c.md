---
type: "practice"
difficulty: "Deep"
---

# You want to warm the provider prompt cache and your vector cache before a known traffic spike (e.g. a product launch). How would you design a prefetch/warm REST job and measure whether it actually improved p99 during the spike?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Load Shedding Responses](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Load_Shedding_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To warm the provider prompt cache and vector cache before a known traffic spike, we need to design a prefetch job that runs a series of predefined tasks to populate these caches efficiently. This job will ensure that when the actual spike happens, the system can handle requests more quickly with minimal latency. The key steps include defining the jobs, setting up monitoring, and analyzing the impact on performance metrics.

The main goal is to measure whether the prefetch job improves p99 response time during the traffic spike. This involves understanding the current baseline performance, simulating the expected load, and comparing post-warm-up performance with historical data or controlled experiments.

**Comparison:**

```
Warm Job Metrics	Baseline	Post-Warm-Up
p99 Response Time (ms)	850	720
```

**Code:**

```python
# Example Python code snippet for a simple warm job scheduler

import requests
from datetime import datetime

def fetch_data_from_provider(provider_url):
    response = requests.get(provider_url)
    return response.json()

def store_in_cache(cache, data):
    # Simulate cache storage logic (e.g., using Redis set commands)
    pass

def run_warm_job(provider_urls, vector_cache):
    start_time = datetime.now()
    for provider_url in provider_urls:
        prompt_data = fetch_data_from_provider(provider_url)
        store_in_cache(vector_cache, prompt_data)
    end_time = datetime.now()
    return (start_time, end_time)

provider_urls = ["http://example-provider1.com/api/data",
                 "http://example-provider2.com/api/data"]

warm_job_start_end_times = run_warm_job(provider_urls, vector_cache)
print(f"Warm job completed in {warm_job_start_end_times[1] - warm_job_start_end_times[0]} seconds")
```

**Example input / output:**

```text
# Example dry-run input and expected output

provider_urls = ["http://example-provider1.com/api/data", "http://example-provider2.com/api/data"]
vector_cache = {
    "type": "redis",
    "host": "localhost",
    "port": 6379,
    "db_index": 0
}

# Expected warm job completion time: around 5 seconds if each request takes about 1 second and there's no network delay

warm_job_start_end_times = run_warm_job(provider_urls, vector_cache)
print(f"Warm job completed in {warm_job_start_end_times[1] - warm_job_start_end_times[0]} seconds")
```

**Data:**

```json
{
  "provider_urls": ["http://example-provider1.com/api/data", "http://example-provider2.com/api/data"],
  "vector_cache": {
    "type": "redis",
    "host": "localhost",
    "port": 6379,
    "db_index": 0
  }
}
```

**Flow:**

1. Not Warmed
2. Start Warm Job
3. Cache Status?
4. Run Prefetch Jobs
5. Warm Up Complete
6. Monitor Performance
7. Mitigate Issues if Any
8. Compare Baseline vsPost-Warm-Up Metrics

