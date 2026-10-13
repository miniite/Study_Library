---
type: "practice"
difficulty: "Deep"
---

# Design a system to serve real-time stock price ticks to millions of concurrent web/mobile clients — how do you fan out updates efficiently without overwhelming clients?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Real-time Price Feeds](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Real-time_Price_Feeds.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a system that efficiently serves real-time stock price ticks to millions of concurrent web and mobile clients, we need to focus on scaling both horizontally and vertically while ensuring low latency. The goal is to fan out updates without overwhelming individual clients or the overall infrastructure.

#### Triage
The core challenge here is to distribute real-time data updates (stock tick prices) efficiently across a large number of clients in near-real time, with minimal latency. This involves understanding client-side constraints and server-side optimization techniques. The system needs to handle high throughput while avoiding unnecessary load on clients.

Key Assumptions:

The system must support millions of concurrent clients.
Stock tick updates need to be delivered as quickly as possible (ideally, within milliseconds).
Clients can tolerate some degree of latency or disconnection for a short period.
The solution should handle both web and mobile clients.

#### Solution

Step-by-Step First Principles Solution

Event Source: Stock prices are continuously updated by financial exchanges. These updates must be ingested as fast as possible, ideally in milliseconds.

Ingestion Layer:

Use a low-latency message broker like Kafka or Nats to ingest stock tick events.
Kafka is well-suited for this scenario due to its high throughput and distributed nature.

Message Routing:

Design topic/partitioning schemes that ensure load balancing across brokers.
For example, use ticker symbols as part of the key, distributing updates evenly among partitions.

Fan-Out Layer:

Use a pub/sub model to fan out messages to multiple clients.
Implement efficient pub/sub patterns using technologies like Redis Streams or RabbitMQ for low-latency delivery.

Client-Side Handling:

Clients (web and mobile) should implement efficient subscription mechanisms, such as long-polling or WebSocket connections.
Use HTTP/2 and WebSocket for bidirectional communication where possible.

Caching Layer:

Implement caching to reduce the load on backend services. Use Redis or Memcached to store recent stock prices.
Cache updates so that clients can fetch them locally if they miss an update during a brief disconnection.

Load Balancing and Scalability:

Ensure the system is horizontally scalable by distributing traffic across multiple instances of the service.
Use load balancers (e.g., HAProxy, Nginx) to distribute incoming requests evenly.

Monitoring and Alerting:

Implement comprehensive monitoring using tools like Prometheus for performance metrics.
Set up alerts for high latency or connection issues to ensure quick remediation.

Resilience and Fault Tolerance:

Design the system to handle failures gracefully by implementing retries, backoff strategies, and circuit breakers.
Use techniques like leader election for pub/sub topics if needed.

Security Considerations:

Implement encryption for data in transit and at rest.
Ensure proper authentication and authorization mechanisms are in place.

#### Closing Summary

The key to efficient real-time stock price updates is a combination of robust event ingestion, efficient message routing, smart client-side handling, caching, and resilient infrastructure. By leveraging technologies like Kafka, Redis, and WebSocket, we can build a scalable and high-performance system that meets the requirements of millions of concurrent clients.

#### Common Mistakes
Overloading individual clients with too many updates.
Not using appropriate pub/sub patterns for low-latency delivery.
Failing to cache frequently accessed data, leading to unnecessary backend load.
Underestimating the importance of client-side optimization and disconnection handling.

**Comparison:**

```
Broker	Partitions
Kafka	8
Redis	6
Client Type	Connection Method	Example Code
Web	WebSocket	wss://stock-feed.example.com
Mobile	WebSocket	wss://mobile-stocks.example.com
```

**Code:**

```
Example Kafka producer code

from kafka import KafkaProducer

producer = KafkaProducer(bootstrap_servers='kafka:9092') future = producer.send('stock-ticks', b'{"symbol": "AAPL", "price": 150.34}') result = future.get(timeout=60) # Ensure delivery

CODE
Copy

```python
# Example WebSocket client code
import websocket

def on_message(ws, message):
    print(message)

ws = websocket.WebSocketApp("wss://stock-feed.example.com", on_message=on_message)
ws.run_forever()
```

**Example input / output:**

```
Input: Stock price tick event sent to Kafka topic stock-ticks with payload {"symbol": "AAPL", "price": 150.34}
Output: Event is delivered to WebSocket clients subscribed to the corresponding topic.
JSON
Copy
{
  "client_id": "1234",
  "message": {"symbol": "AAPL", "price": 150.35}
}
```

**Data:**

```json
{
  "topic": "stock-ticks",
  "partitions": [
    {"key": "AAPL", "replicas": [1, 2]},
    {"key": "MSFT", "replicas": [3, 4]}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Real-time Stock Ticks
4. Kafka Broker
5. Is client connected?
6. WebSocket Client
7. Caching Layer (Redis)
8. Client Fetches Local Cache
9. Process Tick

