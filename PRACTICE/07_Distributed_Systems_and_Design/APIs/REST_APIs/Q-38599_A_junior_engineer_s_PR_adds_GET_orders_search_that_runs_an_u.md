---
type: "practice"
difficulty: "Deep"
---

# A junior engineer's PR adds GET /orders/search that runs an unbounded query. What would you flag in review about pagination, injection, timeouts, indexing, and abuse, and how do you teach the pattern?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Consistent Pagination](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Consistent_Pagination.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When reviewing a junior engineer's PR that introduces a new GET /orders/search endpoint without proper pagination, I would focus on several critical aspects: pagination to prevent excessive load and memory usage, SQL injection to ensure data security, timeout settings to manage server performance, indexing for efficient query execution, and abuse prevention to protect against misuse. By addressing these issues, we can create a robust and scalable API.

To start with the core solution, the GET /orders/search endpoint should implement pagination effectively to limit the number of results returned in each request. We need to ensure that the search queries are parameterized properly to avoid SQL injection. Timeout settings will help manage server load by limiting how long each query can run. Indexing on relevant fields will improve query performance, and implementing rate limiting or other abuse prevention mechanisms is crucial.

Firstly, pagination should be implemented using standard query parameters such as page and pageSize. For example:

PLAINTEXT
GET /orders/search?page=1&pageSize=10

This approach ensures that the server does not have to process an excessive number of records in one go. Additionally, we can add a default limit on the pageSize parameter, for instance, setting it to 50 or 100 by default.

Next, regarding SQL injection, the query parameters should be treated as inputs and used with placeholders in prepared statements. For example:

SQL
SELECT * FROM orders WHERE status = :status AND customer_id = :customer_id;

This pattern ensures that user input is sanitized and prevents SQL injection attacks.

Timeout settings are essential to prevent server overload. We can set a reasonable timeout, for instance, 5 seconds or 10 seconds, depending on the complexity of the query. This can be configured in the database connection pool or directly within the application code:

JAVA
connection.setQueryTimeout(5);

Properly setting timeouts helps manage resources and prevent hanging requests.

Indexing should be applied to fields that are frequently used in search queries, such as status, customer_id, etc. This can significantly reduce query execution time. For example, creating an index on the status column:

SQL
CREATE INDEX idx_orders_status ON orders (status);

This optimization is crucial for large datasets and can drastically improve performance.

Lastly, abuse prevention measures are necessary to prevent malicious or excessive use of the API. Implementing rate limiting, such as allowing a maximum of 100 requests per minute, can help control abuse:

JSON
{
  "rate_limit": {
    "limit": 100,
    "interval": "1m"
  }
}

These measures ensure that the API remains secure and reliable. By reviewing these aspects, we can teach junior engineers about best practices in designing robust REST APIs.

**Comparison:**

```
Parameter	Description
page	Page number (default is 1)
pageSize	Number of results per page (default is 50)
```

**Code:**

```

// Example of using pagination with query parameters @GetMapping("/orders/search") public ResponseEntity<List<Order>> searchOrders( @RequestParam(value = "page", defaultValue = "1") int page, @RequestParam(value = "pageSize", defaultValue = "50") int pageSize) { PageRequest pageRequest = PageRequest.of(page - 1, pageSize); return new ResponseEntity<>(orderService.searchOrders(pageRequest), HttpStatus.OK); }

CODE
Copy

```java
// Example of using prepared statements to prevent SQL injection
@GetMapping("/orders/search")
public ResponseEntity<List<Order>> searchOrders(
    @RequestParam("status") String status,
    @RequestParam("customer_id") Long customerId) {
    try (Connection conn = dataSource.getConnection()) {
        PreparedStatement ps = conn.prepareStatement("SELECT * FROM orders WHERE status = ? AND customer_id = ?");
        ps.setString(1, status);
        ps.setLong(2, customerId);
        ResultSet rs = ps.executeQuery();
        List<Order> orders = new ArrayList<>();
        while (rs.next()) {
            orders.add(mapOrder(rs));
        }
        return new ResponseEntity<>(orders, HttpStatus.OK);
    } catch (SQLException e) {
        throw new RuntimeException(e);
    }
}
JAVA
Copy
// Example of setting query timeout in JDBC connection pool configuration
datasource.setQueryTimeout(10); // 10 seconds
```

**Example input / output:**

```

Sample input and output:

PLAINTEXT
Copy
Input:
GET /orders/search?page=1&pageSize=50&status=pending&customer_id=123

Output:
HTTP/1.1 200 OK
Content-Type: application/json

[
  {
    "id": 1,
    "customerId": 123,
    "status": "pending",
    "createdAt": "2023-09-15T14:00:00Z"
  },
  {
    "id": 2,
    "customerId": 123,
    "status": "pending",
    "createdAt": "2023-09-16T14:00:00Z"
  }
]
```

**Data:**

```json
{
  "rate_limit": {
    "limit": 100,
    "interval": "1m"
  }
}
```

**Flow:**

1. No
2. Yes
3. GET /orders/search
4. Is pagination set?
5. Log error and return empty list
6. Parse query parameters
7. Prepare SQL statement withplaceholders
8. Execute query
9. Map results to Order objects
10. Return 200 OK response withorder list

