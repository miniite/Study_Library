---
type: "practice"
difficulty: "Deep"
---

# What is the difference between a CDN 'push' model and a 'pull' model for populating edge caches?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Web and Browser Caching](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Web_and_Browser_Caching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In caching, particularly in content delivery networks (CDNs), the push and pull models refer to how data is populated into edge caches from the origin server. The core difference lies in who initiates the cache update process—either the origin server pushing the content down to the edges or the client pulling it up. Understanding these models helps in choosing the right strategy based on network conditions, load balancing needs, and latency requirements.

The push model involves the origin server proactively sending content to edge locations. This can be useful for frequently updated content, as it ensures that cache entries are always fresh and avoids stale data issues. However, this model may not scale well with a vast number of clients due to high initial bandwidth usage.

In contrast, the pull model relies on client requests driving cache updates. Each client request triggers a fetch from the origin server, which then populates the local edge cache. This approach is more scalable in terms of initial load but can introduce latency and may not keep all caches fresh unless the system uses mechanisms like timestamps or etags to detect changes.

**Comparison:**

```
Model	Initiation	Scalability	Freshness Management
Push	Origin	Moderate	Automatic
Pull	Client	High	Depends on mechanisms
```

**Code:**

```text
# Example code snippet for setting up a pull-based caching strategy
def handle_request(request):
    # Check if cache contains recent content
    if is_cache_valid(request.url, request.timestamp):
        return serve_from_cache()
    else:
        # Fetch from origin and update cache
        response = fetch_from_origin(request.url)
        update_cache(response.content, request.url)
        return serve_from_cache()

def fetch_from_origin(url):
    # Simulate network roundtrip to origin server
    time.sleep(0.5)  # Simulated latency
    return get_content_from_server(url)

def is_cache_valid(url, timestamp):
    # Check if cached content has expired based on last modified timestamp
    latest_modified_time = get_latest_modified_time_from_db(url)
    return datetime.now() - timestamp <= timedelta(seconds=latest_modified_time)

def update_cache(content, url):
    # Store content in local cache database
    store_content_in_local_db(url, content)
```

**Example input / output:**

```

Input:

PYTHON
Copy
request = {
    'url': 'https://example.com/image.jpg',
    'timestamp': datetime(2023, 10, 1, 14, 0)
}

Output:

If cache is valid (less than 2 seconds old):

JSON
Copy
{"content": "<image content>"}

If cache is not valid (more than 2 seconds old):

JSON
Copy
Fetching from origin...
{"content": "Updated image content", "latency": 0.5s}
```

**Data:**

```json
{
  "request": {
    "url": "https://example.com/image.jpg",
    "timestamp": "2023-10-01T14:00:00Z"
  },
  "cache": {
    "url": "https://example.com/image.jpg",
    "content": "<image content>",
    "last_modified_time": "2023-10-01T13:59:58Z"
  }
}
```

**Flow:**

1. Yes
2. No
3. Client Request
4. Cache Valid?
5. Serve From Cache
6. Fetch from Origin
7. Update Cache

