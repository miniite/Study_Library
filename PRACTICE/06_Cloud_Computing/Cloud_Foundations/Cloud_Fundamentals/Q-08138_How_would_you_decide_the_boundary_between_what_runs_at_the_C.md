---
type: "practice"
difficulty: "Deep"
---

# How would you decide the boundary between what runs at the CDN edge (edge functions) versus in a central region, given latency, data locality, and debugging complexity?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cold Starts and Regional Differences](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Cold_Starts_and_Regional_Differences.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide where to run code—whether it should be hosted on the CDN edge or within a central region—we must consider key factors such as latency, data locality, and debugging complexity. By understanding these aspects from first principles, we can make an informed decision that aligns with business goals and technical constraints.

#### Triage
What's Being Probed

The core of this question revolves around understanding the trade-offs between running code at the edge (CDN) versus in a central region. The interviewer is probing your ability to evaluate performance metrics, data access patterns, and operational complexities. We need to make assumptions about the nature of the application, the network architecture, and typical usage scenarios.

#### First-Principles Solution
Latency: Edge functions are ideal for reducing latency by processing requests closer to end-users. This is critical for real-time applications like streaming services or interactive web applications.
Data Locality: If data-intensive operations need access to large datasets, running them centrally ensures consistent and high-performance data access. Conversely, simple transformations or metadata operations can be offloaded to the edge without significant performance loss.
Debugging Complexity: Centralized code management and debugging tools typically offer better support for complex applications and larger development teams.

Let's break down these factors in detail:

Latency
Edge Functions: Place functions that require minimal interaction with backend services or databases at the edge. For instance, image resizing or simple filtering operations can be handled by edge nodes.
Central Region: Optimize central regions for tasks that are data-intensive or require complex processing. This could include machine learning models, detailed database queries, or batch processing jobs.
Data Locality
Edge Functions: If the function can operate on small or static datasets, it's fine to place them at the edge. For example, applying a fixed set of rules to incoming data.
Central Region: Use central regions for applications that require access to large and dynamic datasets, such as real-time analytics or serving personalized content.
Debugging Complexity
Edge Functions: These can be simpler to debug because they often run in more controlled environments. However, network issues may complicate diagnostics.
Central Region: Centralized systems provide better tools for tracing and debugging, but increased complexity due to larger codebases and multiple dependencies needs to be managed.

#### Conclusion
& Common Mistakes

Deciding between edge functions and central regions involves a nuanced understanding of the application's requirements. A common mistake is overcomplicating decisions by trying to push all non-trivial logic to the edge, which can introduce unnecessary complexity and overhead. Conversely, placing too much logic in the central region might lead to increased latency.

Common Follow-Ups
What if we have hybrid scenarios?
How do we handle stateful operations at the edge?
What about security considerations?

**Comparison:**

| Function Type | Characteristics                    | Example Use Case                           |
|---------------|------------------------------------|-------------------------------------------|
| Edge          | Low latency, simple operations      | Image resizing, filtering                  |
| Central       | High performance data access        | Machine learning models, complex queries   |

**Code:**

```python
# Example of an edge function for image resizing (Python)
def resize_image(image_url, target_size):
    # Fetch the image from CDN
    img = fetch_image_from_cdn(image_url)

    # Resize the image locally or using a library
    resized_img = img.resize(target_size)

    return resized_img

# Example of a central function for complex processing (Python)
def process_user_data(user_id, data):
    user_db = get_user_data_from_db(user_id)
    processed_data = apply_complex_algorithms(user_db, data)

    # Store the result back in the database
    store_processed_data_in_db(processed_data)
```

**Example input / output:**

```text
# Example Input-Output for Edge Function (Resize Image)
Input:
{
  "image_url": "https://cdn.example.com/image.jpg",
  "target_size": [300, 300]
}

Output:
{
  "resized_image_url": "https://edge.example.com/resized-image.jpg"
}
```

**Data:**

```json
{
  "edgeFunction": {
    "name": "resize_image",
    "params": ["image_url", "target_size"],
    "description": "Resizes an image to a specified target size."
  },
  "centralFunction": {
    "name": "process_user_data",
    "params": ["user_id", "data"],
    "description": "Processes user data using complex algorithms and stores the result in the database."
  }
}
```

**Flow:**

1. No
2. Yes
3. User Request
4. Is Data Intensive?
5. Edge Function
6. Central Region
7. Example: Image Resizing
8. Example: Machine LearningModel

