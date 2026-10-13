---
type: "practice"
difficulty: "Expert"
---

# What is persisted queries in GraphQL and how does it enable CDN-level caching of otherwise-POST queries?

**Difficulty:** Expert · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Stale While Revalidate](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Stale_While_Revalidate.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Persisted queries in GraphQL are a feature that allows clients to reuse the exact same query with different variables, which can be precompiled on the server side and cached at the application level. This enables more efficient and faster retrieval of data by leveraging CDN-level caching for otherwise-POST queries. Essentially, it transforms dynamic queries into static ones, making them cacheable.

#### Triage

The question is asking about the concept of persisted queries in GraphQL and how they facilitate CDN-level caching for what would typically be POST requests. The core idea here involves understanding why these queries need to be cached, the implications of precompiling and persisting them, and the benefits in terms of performance and scalability.

#### First-Principles Solution
Understanding Persistent Queries: A persisted query is a GraphQL query that gets stored as a string on the server side after it has been executed for the first time with a set of variables. Subsequent requests can reuse this exact same query, but with different variable values.
Precompilation and Caching at Application Level: By precompiling queries, we avoid parsing them each time they are received from the client. This results in faster execution times on the server side.
CDN-Level Caching for Performance: When a persisted query is reused with different variables, it can be cached at the CDN (Content Delivery Network) level. CDNs typically cache HTTP requests, and since GraphQL queries can now be represented as simple strings, they too can be cached.
Detailed Explanation

Why Cache Queries?

Frequent requests to a server for the same data with slightly different parameters are common in web applications.
Caching these queries reduces the load on the backend services, leading to better performance and reduced latency.

Precompilation Process:

The client sends its query along with variable values to the server.
The server compiles the query into a string representation (persisted query) and caches it.
Subsequent requests for the same query structure but different variables can use this cached persisted query.

#### Example
Workflow
Initial Request:
GRAPHQL
{
  users(id: "123") {
    name
    email
  }
}
Server Processing:
Server compiles the query and stores it as a string.
Response is sent back to the client.
Subsequent Requests with Different Variables:
GRAPHQL
{
  users(id: "456") {
    name
    email
  }
}
Server Handling:
Server looks up the persisted query by matching its structure.
Replaces variables and executes the query.
Benefits of Persisted Queries for CDNs
Reduced Backend Load: Since queries are precompiled, there’s less computation required on each request.
Faster Response Times: Queries can be served directly from the CDN, reducing latency.
Scalability: With fewer backend requests, the system can handle more concurrent users efficiently.

#### Common Mistakes
and Follow-Ups
Mistake: Not Understanding Variable Reuse:
A common pitfall is assuming that each request for a similar query needs to be fully recompiled.

#### Follow-Up
How does this affect the security and privacy of data?
Mistake: Ignoring Query Structure Changes:
If the structure of a query changes, it might not get cached correctly if persisted queries are not properly handled.

#### Follow-Up
What happens when the schema evolves over time?

#### Closing Summary

Persisted queries in GraphQL enable efficient caching by converting dynamic queries into static strings. This approach reduces backend load and improves response times, making them ideal for CDNs where data can be cached at multiple layers of the network. The key lies in understanding how to handle query variables effectively while ensuring that schema changes are managed properly.

**Comparison:**

```
Query Text	Variables	Cached ID
user(id)	{ id: "123" }	MD5(user(id) + { id: "123" })
user(id)	{ id: "456" }	MD5(user(id) + { id: "456" })
```

**Code:**

```javascript
// Example code snippet demonstrating how persisted queries might be handled on the server side
const persistedQueries = new Map();

app.post('/graphql', (req, res) => {
  const queryText = req.body.query;
  const variables = req.body.variables;

  // Check if the exact query already exists in the cache
  let cachedQuery = persistedQueries.get(queryText);

  if (!cachedQuery) {
    // Compile and store the query with its unique ID (could be MD5 hash of queryText + variables)
    const uniqueID = hash(`${queryText}+${JSON.stringify(variables)}`);
    cachedQuery = { id: uniqueID, query: queryText };
    persistedQueries.set(queryText, cachedQuery);
  }

  // Execute the query with the given variables
  executeGraphQL(cachedQuery.query, variables).then(result => {
    res.json(result);
  });
});
```

**Example input / output:**

```

Initial Request:

Input:
GRAPHQL
Copy
{
  users(id: "123") {
    name
    email
  }
}
Output:
JSON
Copy
{
  "data": {
    "users": [
      { "name": "John Doe", "email": "john@example.com" }
    ]
  }
}

Subsequent Request with Different Variables:

Input:
GRAPHQL
Copy
{
  users(id: "456") {
    name
    email
  }
}
Output:
JSON
Copy
{
  "data": {
    "users": [
      { "name": "Jane Doe", "email": "jane@example.com" }
    ]
  }
}
```

**Data:**

```json
{
  "query": "{ users(id: \"123\") { name email } }",
  "variables": {}
}
```

**Flow:**

1. No
2. Yes
3. Client sends query
4. Is the exact query cached?
5. Server compiles & caches
6. Server sends precompiledquery to cache
7. Server fetches and executesfrom cache
8. Client gets response quickly

