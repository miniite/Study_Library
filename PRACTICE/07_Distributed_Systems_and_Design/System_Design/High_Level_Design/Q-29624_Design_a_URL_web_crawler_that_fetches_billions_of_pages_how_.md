---
type: "practice"
difficulty: "Deep"
---

# Design a URL/web crawler that fetches billions of pages — how do you handle politeness, dedup of URLs, distributed frontier management, and re-crawl scheduling?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Search Backend Design](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Search_Backend_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a high-level architecture for a web crawler that handles fetching billions of pages efficiently, we need to consider several key aspects: politeness (ensuring we respect site limits), deduplication of URLs, distributed frontier management, and re-crawl scheduling. This problem is complex but can be tackled methodically by breaking it down into manageable components.

Politeness involves adhering to the robots.txt file rules of a website to avoid overwhelming servers with too many requests. Deduplication ensures we don't process the same URL multiple times, saving both time and resources. Distributed frontier management helps scale our crawler across multiple machines while maintaining consistency. Finally, re-crawl scheduling optimizes when to revisit pages based on their freshness.

**Comparison:**

```
Component	Description
Politeness	Checks if the crawler can access a URL based on robots.txt.
Deduplication	Ensures no duplicate URLs are fetched.
Frontier	Manages which pages to fetch next and ensures consistency across multiple crawlers.
Scheduling	Decides when to re-crawl a page, balancing freshness and resource usage.
```

**Code:**

```python
# Example code snippet for basic politeness check using requests
import requests

def is_allowed(url):
    try:
        response = requests.get(f"{url}/robots.txt")
        if "Disallow:" in response.text:
            return not any(d in url for d in response.text.split("\n") if d.strip())
        return True
    except Exception as e:
        print(f"Error checking politeness: {e}")
        return False
```

**Example input / output:**

```

Given the input:

URL: https://example.com/

The output would be a boolean indicating whether crawling is allowed according to robots.txt:

PYTHON
Copy
>>> is_allowed("https://example.com/")
False

This check respects site limits and avoids potential issues.
```

**Data:**

```json
{
  "url": "https://example.com/",
  "robots_txt_content": "User-agent: *\nDisallow: /admin/"
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is URL allowed?
5. Deduplication Check
6. Fetch Page
7. Scheduling Decision
8. Store in Frontier Queue
9. Log Violation
10. End

