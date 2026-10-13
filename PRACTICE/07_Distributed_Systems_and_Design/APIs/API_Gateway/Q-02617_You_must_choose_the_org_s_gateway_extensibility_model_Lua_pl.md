---
type: "practice"
difficulty: "Expert"
---

# You must choose the org's gateway extensibility model: Lua plugins, WASM, external gRPC filters, or sidecar processors; adjudicate the long-term trade-offs of safety, performance, and developer ergonomics, and explain the migration path if you picked wrong.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ API Gateway Strategy at Scale](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Routing_and_BFF_Patterns/API_Gateway_Strategy_at_Scale.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

As a Backend Architect/Principal, choosing the org's gateway extensibility model is a critical decision that will impact the long-term maintainability, performance, and developer ergonomics of our APIs. We need to consider multiple factors such as safety, performance, and ease of development. In this case, we are evaluating Lua plugins, WASM (WebAssembly), external gRPC filters, and sidecar processors.

To begin, let’s triage the question: The primary goal is to assess which extensibility model best fits our needs in terms of safety, performance, and developer ergonomics while also considering a migration path if we make an incorrect choice. Our assumptions include:

We are working with an established API gateway that supports these extension mechanisms.
Developer ergonomics encompass ease of writing, testing, and maintaining extensions.

Now, let’s explore each option:

Lua Plugins: Lua is a lightweight, embeddable scripting language known for its simplicity and flexibility. Lua plugins can be easy to develop due to their straightforward syntax and extensive libraries. However, they may lack the performance optimizations found in more modern languages like Rust or WebAssembly.

WASM (WebAssembly): WASM is gaining popularity due to its ability to run compiled code from various languages with close-to-native performance. It offers a higher level of safety compared to low-level languages and better cross-platform compatibility. The downside is the initial setup overhead for compilation, which can be complex.

External gRPC Filters: gRPC filters allow us to define custom logic in separate services that communicate via gRPC. This approach provides excellent isolation between the core gateway code and extensions. However, it introduces network latency due to inter-service communication and requires additional infrastructure management.

Sidecar Processors: Sidecar processors run alongside the main API gateway process as a separate microservice. They can provide significant performance benefits by offloading computationally intensive tasks but come with increased complexity in deployment and maintenance.

Let’s evaluate these options based on safety, performance, and developer ergonomics:

Safety:

Lua: Good for simple scripts but may introduce vulnerabilities if not carefully managed.
WASM: High safety due to sandboxing and limited access to the host environment.
gRPC Filters: Moderate risk; network calls can be exploited if not properly secured.
Sidecar Processors: Low risk as they run in isolation, but inter-service communication needs proper security.

Performance:

Lua: Best for lightweight scripting tasks but may become a bottleneck for heavy processing.
WASM: Excellent performance suitable for CPU-bound and memory-intensive tasks.
gRPC Filters: Network latency can impact performance.
Sidecar Processors: High performance due to offloading, but initial setup and management overhead.

Developer Ergonomics:

Lua: Easy to learn and use for simple tasks but can become complex for large-scale applications.
WASM: Requires a compiler and development environment knowledge; might be challenging for some developers.
gRPC Filters: Complex setup with significant learning curve due to service communication and protocol handling.
Sidecar Processors: Similar to gRPC filters in complexity, requiring robust deployment practices.

Given these considerations, WASM seems the most balanced choice. It offers near-native performance, high safety, and a relatively straightforward developer experience compared to other options. If chosen incorrectly, we can always revisit our decision by migrating to Lua plugins or gRPC filters as needed.

#### Closing Summary
After thorough analysis, WASM emerges as the preferred extensibility model due to its combination of high performance, safety, and ease of development. However, a clear migration path should be planned in case this choice turns out not to be optimal.

**Comparison:**

```
Model	Safety	Performance	Developer Ergonomics
Lua Plugins	Moderate	Good	Easy
WASM	High	Excellent	Medium
gRPC Filters	Moderate	Good	Hard
Sidecar Proc.	Low	High	Hard
```

**Code:**

```

-- Example Lua plugin code function handle_request(request) -- Perform custom logic here end

CODE
Copy

```wasm
// Example WASM module
extern "C" {
  fn handle_request(input_data) -> output_data;
}

export fn main() {
  // Handle incoming requests and perform custom logic
}
```

**Example input / output:**

```text
Input:
- Request with method GET and path /api/v1/resource

Output:
- Response with status 200 and body {"message": "Hello, World!"}

Dry-run request for Lua plugin:

Request: GET /api/v1/resource
Response: 200 {"message": "Hello, World!"}
```

**Data:**

```json
{
  "exampleRequest": {
    "id": 1,
    "method": "GET",
    "path": "/api/v1/resource"
  },
  "expectedResponse": {
    "status": 200,
    "body": "{\"message\": \"Hello, World!\"}"
  }
}
```

**Flow:**

1. Yes
2. No
3. Choose Extensibility Model
4. WASM
5. Correct Choice?
6. Implement WASM Filters
7. Migrate to Lua Plugins orgRPC Filters

