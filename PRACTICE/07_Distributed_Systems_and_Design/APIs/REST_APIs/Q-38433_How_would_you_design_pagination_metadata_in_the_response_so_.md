---
type: "practice"
difficulty: "Intermediate"
---

# How would you design pagination metadata in the response so a client knows the total count and next page?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Filtering Sorting and Pagination](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Filtering_Sorting_and_Pagination.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address how to include pagination metadata in a REST API response, let's first summarize the core solution and key idea. We'll need to return relevant metadata along with the requested data to inform the client about the total count of items and whether there is a next page. This involves fetching the necessary details from your database or cache while also providing the client with an understanding of how pagination works.

The question is probing our knowledge of RESTful API design, specifically around efficient data retrieval and response formatting. We need to make sure that the metadata we return accurately reflects the current state of the resource collection without overly taxing server resources. Here are some key assumptions:

The backend uses a database or an in-memory store.
The client makes requests using standard HTTP methods like GET.
The API is designed for paginated data retrieval, such as user profiles, posts, comments, etc.

Now let's break down the process from first principles:

Understand the Client’s Request: Typically, clients request a specific page of data with a query parameter specifying the page number and optionally the page size.
Fetch Data from Storage: Retrieve the requested subset of data along with the relevant total count.

#### Cons
truct Response Metadata: Include metadata such as the current page, page size, total items, and next page URL or token if applicable.
Return Data and Metadata: Format the response to include both the data and the metadata in a structured manner.

Here’s a step-by-step solution:

Step 1: Understand the Client's Request

The client sends a request like this:

HTTP
GET /posts?page=2&pageSize=5

This would fetch page 2 of posts with 5 items per page.

Step 2: Fetch Data from Storage

Fetch the data based on the provided page and pageSize. For example, if you are fetching user profiles:

SQL
SELECT * FROM users LIMIT (page - 1) * pageSize, pageSize;
Step 3: Construct Response Metadata

Calculate the total count of items in the database. Then construct the metadata as follows:

Current Page: The page number requested by the client.
Page Size: The size of each page.
Total Items: The total number of items available, which might be fetched from a COUNT query or cached.
Next Page URL: If there is a next page, construct its URL.
Step 4: Return Data and Metadata

Return the data along with metadata in a structured response. Here’s an example JSON structure:

JSON
{
    "data": [
        {"id": 101, "name": "User A"},
        {"id": 102, "name": "User B"}
    ],
    "metadata": {
        "currentPage": 2,
        "pageSize": 5,
        "totalItems": 100,
        "nextPageUrl": "/users?page=3&pageSize=5"
    }
}

#### Closing Summary

By including pagination metadata in the response, we ensure that clients can navigate through data efficiently and understand the current state of their resource collection. This involves fetching both the requested subset of data and relevant counts to inform the client about the next page.

#### Common Mistakes/Follow-Ups
Overfetching: Always consider the performance implications of fetching total count, especially for large datasets.
Cache Invalidation: Ensure that your caching strategy does not invalidate the count unnecessarily.
Client-Side Paging: Discuss how clients might handle paging and whether server-side or client-side pagination is more appropriate.

**Comparison:**

```

N/A

N/A

Example request:

HTTP
Copy
GET /users?page=1&pageSize=5

Example response:

JSON
Copy
{
    "data": [
        {"id": 101, "name": "User A"},
        {"id": 102, "name": "User B"}
    ],
    "metadata": {
        "currentPage": 1,
        "pageSize": 5,
        "totalItems": 100,
        "nextPageUrl": "/users?page=2&pageSize=5"
    }
}
```

**Code:**

```python
# Example Python code to fetch user profiles with metadata
def get_user_profiles(page, page_size):
    # Fetch data from database
    query = "SELECT * FROM users LIMIT %s OFFSET %s"
    params = (page_size, (page - 1) * page_size)
    result = execute_query(query, params)

    # Calculate total items count
    total_items_query = "SELECT COUNT(*) FROM users"
    total_items_result = execute_query(total_items_query)
    total_items = total_items_result.fetchone()[0]

    # Construct metadata and response
    next_page_url = f"/users?page={page + 1}&pageSize={page_size}" if page * page_size < total_items else None
    response_data = {
        "data": result,
        "metadata": {
            "currentPage": page,
            "pageSize": page_size,
            "totalItems": total_items,
            "nextPageUrl": next_page_url
        }
    }

    return response_data

N/A
```

