---
type: "practice"
difficulty: "Expert"
---

# How would you design pagination that stays consistent when new records are being inserted between page requests?

**Difficulty:** Expert · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Consistent Pagination](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Consistent_Pagination.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Designing a pagination mechanism in REST APIs that remains consistent even when new records are inserted requires a careful approach to handling token-based paging, typically using cursors or tokens. This solution ensures that pagination results remain predictable and reliable.

#### Triaging the Question

The question is asking about how to implement pagination in a way that maintains consistency across multiple page requests, especially when new records are being added dynamically. The interviewer is likely probing your understanding of token-based paging mechanisms and your ability to handle edge cases where data changes between requests.

#### Assumptions
and Context
We have an API endpoint for listing resources.
New records can be inserted at any time during the application's operation, leading to potential mismatches in pagination tokens or cursors used by clients.
The goal is to ensure that a client can reliably retrieve the same set of results even after new data has been added.

#### Solution

The key to maintaining consistency in such scenarios is using token-based paging. Specifically, we'll use offset-based pagination with time-stamped cursors or server-side tokens.

Step-by-Step Design

Identify the Current Offset: When a client requests a page of data, record the current offset (position) and any necessary metadata (like timestamps).

Token Generation: Generate a unique token that encapsulates this state information. This token can be generated based on the last seen ID or timestamp.

Inserting New Records: When new records are added, update the database without changing existing offsets.

Client Behavior:

On each request, the client sends the previously received cursor/token.
The server checks if this token is valid and then retrieves the data from that offset.

Handling Invalid Tokens: If a token becomes invalid due to new records being inserted, return an error or handle it gracefully by prompting the client to fetch newer data with a new token.

Code Example
PYTHON
# Simplified example in Python

class PaginationService:
    def __init__(self):
        self.records = []
        self.last_token = None

    def insert_record(self, record):
        self.records.append(record)
        # Generate the next token based on the last record's ID or timestamp
        self.last_token = str(len(self.records) - 1)

    def get_records_by_token(self, token):
        try:
            index = int(token)
            if index < len(self.records):
                return (self.records[index], self.last_token)
            else:
                # Token is invalid; return error or a new token
                raise ValueError("Invalid token")
        except ValueError as e:
            print(e)
            return None, "new-token"
Handling Edge Cases and Common Mistakes
Race Conditions: Ensure that the token generation and validation processes are thread-safe if your application is multi-threaded.
Client State Management: Clients should always manage tokens correctly to avoid hitting invalid states or needing to fetch data multiple times.
Error Handling: Provide clear error messages when a token becomes invalid, guiding clients on how to handle such situations.

#### Closing Summary

By implementing token-based paging, we can ensure that pagination results remain consistent even when new records are inserted between page requests. This approach requires careful management of tokens and handling of edge cases where tokens become invalid.

#### Common Mistakes/Follow-Ups
Using Simple Offset Without Tokens: Using simple offset-based pagination without considering the possibility of token invalidation.
Handling Large Datasets: Ensuring that the token generation and validation mechanisms scale well with large datasets.
Client-Server Synchronization: Ensuring both client and server are in sync regarding token management, especially when dealing with distributed systems.

**Comparison:**

```
Client Request	Server Response
GET /api/records?page=1&token=0	Results: [record-1, record-2], next_token: 0
POST /api/records {"id": 3, "data": "record-3"}	Inserted new record with id: 3
GET /api/records?page=1&token=0	Results: [record-1, record-2], next_token: 2
```

**Code:**

```python
# Python code example for pagination service
```

**Example input / output:**

```text
Request:
GET /api/records?page=1&token=0

Response:
{
  "results": [
    {"id": 1, "data": "record-1"},
    {"id": 2, "data": "record-2"}
  ],
  "next_token": "0"
}

Insert New Record:
POST /api/records
{"id": 3, "data": "record-3"}

Request with Updated Token:
GET /api/records?page=1&token=0

Response:
{
  "results": [
    {"id": 1, "data": "record-1"},
    {"id": 2, "data": "record-2"}
  ],
  "next_token": "2"
}
```

**Data:**

```json
{
  "records": [
    {"id": 1, "data": "record-1"},
    {"id": 2, "data": "record-2"}
  ],
  "last_token": "0"
}
```

**Flow:**

1. Yes
2. No
3. Client sends request
4. Server receives request
5. Is token valid?
6. D[Server returns results]
7. Server generates new token
8. Client requests with new token

