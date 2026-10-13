---
type: "practice"
difficulty: "Deep"
---

# You need to expose a resource whose full representation is 2MB but most callers need three fields. How do you design field selection, projections, and default response shapes so the common case is cheap without maintaining ten endpoints?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Coarse vs Fine-grained Endpoints](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Coarse_vs_Fine-grained_Endpoints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we'll design a REST API that efficiently serves large resources while allowing selective fetching of fields. The key concepts are field selection and response shaping. We need to ensure that the most common case is cheap in terms of bandwidth and computational overhead without overcomplicating our endpoint management.

#### Triaging the Question

The core issue here is optimizing resource retrieval for both efficiency and scalability. Most API users will only require a subset of fields, so we want to minimize the payload size while ensuring full representation capabilities when needed. We must balance this with the cost of maintaining multiple endpoints or complex logic within a single endpoint.

#### First-Principles Solution
Field Selection: Implement a query parameter for field selection.
Response Shaping: Use conditional rendering based on the selected fields.
Default Response Shape: Provide a default response that includes all necessary fields and is optimized for common use cases.
Step-by-Step Solution

Define Query Parameters:

Introduce query parameters to allow clients to specify which fields they need. For example, fields or include.

Conditional Rendering Logic:

Use a library like Jackson in Java or similar tools in other languages for dynamic field inclusion.
If no fields are specified, serve the full representation.

Optimize Common Case:

Ensure that common field selections (e.g., three fields) are handled efficiently by pre-allocating response buffers and only including necessary data.

API Design:

Use a consistent URL structure for both full and partial responses.

#### Example
/api/resource?id=123&fields=name,description,id.
Code Implementation
JAVA
@GetMapping("/resource")
public ResponseEntity<ResourceDto> getResource(@RequestParam(required = false) String fields) {
    Resource resource = repository.findById(id).orElseThrow(() -> new NoSuchElementException());

    // Default full representation if no fields are specified
    ResourceDto defaultResponse = modelMapper.map(resource, ResourceDto.class);

    // Conditional rendering based on query parameters
    if (fields != null && !fields.isEmpty()) {
        String[] selectedFields = fields.split(",");
        List<String> fieldList = Arrays.asList(selectedFields);

        Map<String, Object> reducedResponse = new HashMap<>();

        // Include only necessary fields in the response
        resourceDto.forEach((key, value) -> {
            if (fieldList.contains(key)) {
                reducedResponse.put(key, value);
            }
        });

        defaultResponse.setFields(reducedResponse);
    }

    return ResponseEntity.ok().body(defaultResponse);
}

#### Closing Summary
and Common Mistakes/Follow-Ups

By implementing field selection using query parameters and conditional rendering logic, we can optimize the API for both common and uncommon use cases. The key is to balance simplicity with performance.

#### Common Mistakes

Failing to handle the case where no fields are specified.
Overcomplicating the response shape logic without optimizing for the most frequent cases.
Not considering edge cases like invalid field names or excessive query strings.

#### Follow-Ups
Performance Metrics: How would you measure and optimize performance in a real-world scenario?
Security Considerations: What security measures should be taken when exposing sensitive fields based on client requests?
Scalability: How would this solution scale with increasing numbers of concurrent users or larger resource representations?

**Comparison:**

```
Field	Description
id	Unique identifier for the resource.
name	The name of the resource.
description	A brief description of the resource.
```

**Code:**

```java
// Example Java code for conditional rendering using Jackson
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.util.StdConverter;

@GetMapping("/resource")
public ResponseEntity<ResourceDto> getResource(@RequestParam(required = false) String fields) {
    Resource resource = repository.findById(id).orElseThrow(() -> new NoSuchElementException());

    ObjectMapper mapper = new ObjectMapper();
    ResourceDto defaultResponse = modelMapper.map(resource, ResourceDto.class);

    if (fields != null && !fields.isEmpty()) {
        String[] selectedFields = fields.split(",");
        List<String> fieldList = Arrays.asList(selectedFields);

        Map<String, Object> reducedResponse = new HashMap<>();

        // Include only necessary fields in the response
        resourceDto.forEach((key, value) -> {
            if (fieldList.contains(key)) {
                reducedResponse.put(key, value);
            }
        });

        defaultResponse.setFields(reducedResponse);
    }

    return ResponseEntity.ok().body(defaultResponse);
}
```

**Example input / output:**

```text
Input:
GET /api/resource?id=123&fields=name,description

Output:
{
  "name": "Sample Resource",
  "description": "This is a description of the resource."
}
```

**Data:**

```json
{
  "id": 123,
  "name": "Sample Resource",
  "description": "This is a description of the resource.",
  "additionalField1": "value1",
  "additionalField2": "value2",
  // ... more fields
}
```

**Flow:**

1. Yes
2. No
3. API Request
4. Parse Query Parameters
5. Are fields specified?
6. Conditional Rendering
7. Full Representation
8. Generate Reduced Response
9. Return Response

