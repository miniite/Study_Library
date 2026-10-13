---
type: "practice"
difficulty: "Intermediate"
---

# How would you parallelize independent extraction tasks in Python (for example fetching many API endpoints) to speed up a pipeline, and what should you watch out for?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Large Data Migrations in Python](<../../../../TOOLS/Python/Python_Language/Large_Data_Migrations_in_Python.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To parallelize independent extraction tasks in Python, such as fetching data from multiple API endpoints, we can use libraries like concurrent.futures or multiprocessing. This approach allows us to leverage multi-core processors and speed up our data pipeline. The key is understanding the nuances of these libraries and being mindful of potential issues.

The question essentially asks how to efficiently distribute tasks among threads or processes in Python, ensuring that we don't overwhelm the system with too many requests at once, while also handling any errors gracefully. We'll break down this problem step-by-step from first principles.

**Comparison:**

```
Endpoint	Response Time (s)	Error
https://api.example.com/data1	0.532	None
https://api.example.com/data2	0.498	None
https://api.example.com/data3	0.567	None
```

**Code:**

```python
import concurrent.futures
import requests

# Sample function to fetch data from an API endpoint
def fetch_data(url):
    try:
        response = requests.get(url)
        response.raise_for_status()
        return response.json()
    except requests.RequestException as e:
        print(f"Error fetching {url}: {e}")
        return None
```

**Example input / output:**

```text
Given the following API endpoints:
- `https://api.example.com/data1`
- `https://api.example.com/data2`
- `https://api.example.com/data3`

The expected output would be a list of JSON responses from these endpoints, possibly with error messages for any that failed to fetch.
```

**Data:**

```json
{
  "api_endpoints": [
    "https://api.example.com/data1",
    "https://api.example.com/data2",
    "https://api.example.com/data3"
  ]
}
```

**Flow:**

1. Yes
2. No
3. Define fetch_data function
4. (Get API endpoints)
5. Is endpoint valid?
6. (Fetch data using requests.get)
7. (Handle response, return JSONor log error)
8. (Log and skip invalid URL)

