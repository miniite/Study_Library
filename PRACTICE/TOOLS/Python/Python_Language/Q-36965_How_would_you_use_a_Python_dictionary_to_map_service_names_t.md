---
type: "practice"
difficulty: "Basic"
---

# How would you use a Python dictionary to map service names to their health-check URLs, and iterate over it to check each one?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Parsing JSON Safely](<../../../../TOOLS/Python/Python_Language/Parsing_JSON_Safely.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this question, we'll start by creating a Python dictionary that maps service names to their corresponding health-check URLs. Then, we will write a function to iterate through the dictionary and perform some basic checks on each URL.

The main goal here is to understand how dictionaries in Python can be used for mapping key-value pairs and iterating over them. This concept is fundamental for managing configurations or settings in various applications.

**Code:**

```python
# Define a sample dictionary mapping service names to health-check URLs
services = {
    "service1": "http://example.com/service1/health",
    "service2": "https://api.example.com/healthcheck",
    "service3": "http://localhost/service3/status"
}

def check_health(service_name, url):
    # Placeholder function for checking the health of a service
    print(f"Checking {service_name} at {url}")

# Iterate over the dictionary and perform checks on each URL
for service_name, url in services.items():
    check_health(service_name, url)
```

