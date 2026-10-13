---
type: "practice"
difficulty: "Deep"
---

# What is the difference between REST and gRPC, and when might you use gRPC instead?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ REST Principles](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/REST_Principles.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

REST (Representational State Transfer) and gRPC (Google Remote Procedure Call) are both protocols used to build APIs, but they differ significantly in terms of their underlying architecture, performance, and use cases. In a few sentences, REST is stateless and operates over HTTP/HTTPS using various HTTP methods like GET, POST, PUT, DELETE for interacting with resources. gRPC, on the other hand, relies on a more modern approach leveraging Protocol Buffers (protobuf) for defining services and messages, and it supports multiple protocols including HTTP/2 and TCP.

To properly triage this question, we need to understand that REST is widely adopted due to its simplicity and ease of implementation with web technologies. gRPC, however, offers better performance, scalability, and efficient use of resources by leveraging modern transport mechanisms like bidirectional streaming and advanced data formats. The assumptions here are that the interviewer expects a deep understanding of both protocols, their limitations, and the specific scenarios where one might be preferred over the other.

Step-by-Step Solution

Understand REST:

REST uses HTTP methods for CRUD operations.
Resources are represented as URIs (Uniform Resource Identifiers).
Responses can vary depending on the HTTP status codes.
It is stateless and operates independently of any underlying protocols.

Understand gRPC:

gRPC defines services in a .proto file, which describes message formats and service methods.
Uses Protocol Buffers (protobuf) for efficient data serialization.
Supports various transport layers including HTTP/2 and TCP.
Enables server-to-server and client-to-server bidirectional streaming.

Comparison:

Performance: gRPC is more performant due to its binary serialization format, which reduces the overhead compared to JSON used in REST APIs.
Scalability: gRPC supports advanced features like caching and retries, making it suitable for large-scale applications.
Development Effort: REST requires more manual configuration and handling of states, while gRPC provides a structured approach using .proto files.

When to Use gRPC:

For microservices architectures where performance is critical and you need advanced features like streaming.
In scenarios requiring efficient data transmission, such as IoT applications or real-time communication.
When the application demands more control over error handling and retries.

#### Closing Summary

In summary, while REST has been the go-to choice for many web APIs due to its simplicity and wide adoption, gRPC offers a robust alternative with better performance and scalability. The decision between using REST and gRPC often depends on specific requirements like the need for advanced features, performance optimization, or compatibility with existing systems.

#### Common Mistakes/Follow-Ups
Overlooking the importance of state management in REST APIs.
Not considering the overhead of JSON versus protobuf serialization in gRPC.
Failing to recognize scenarios where bidirectional streaming might be necessary, which is a key feature of gRPC but not present in REST.

**Code:**

```java
// Example .proto file for a simple service
service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply);
}

message HelloRequest {
  string name = 1;
}

message HelloReply {
  string message = 1;
}
```

**Data:**

```json
{
  "name": "Alice"
}
```

**Flow:**

1. Client
2. REST API
3. Database
4. gRPC Service
5. Service Logic
6. Database

