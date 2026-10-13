---
type: "practice"
difficulty: "Intermediate"
---

# Design a system for a real-time dashboard showing live order counts.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Order Tracking Design](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Order_Tracking_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a system for a real-time dashboard showing live order counts, we need to ensure low latency, high availability, and scalability. The core components of our solution will include a microservice architecture with a frontend dashboard, a backend service handling order counts, and a database or caching layer for storing the order data.

#### Triage

The question is essentially asking us to design a system capable of efficiently processing and displaying real-time order counts. This involves understanding the requirements, identifying key components, and architecting the solution to handle high traffic and low latency.

#### Assumptions
Orders are processed in a distributed environment.
The dashboard needs to update within seconds of an order being placed.
There is no upper limit on the number of orders or users accessing the dashboard.
We need to consider data consistency, scalability, and fault tolerance.

#### First-Principles Solution
Frontend Dashboard: This will be responsible for fetching and displaying real-time order counts. It can use WebSocket or long-polling techniques for low-latency updates.
Backend Service: This service will handle the logic of counting orders in real time. We need to ensure that this service is scalable, fault-tolerant, and can handle a high volume of requests.
Database/Cache Layer: A database or cache layer will store order counts to reduce read latency and improve performance.
Detailed Design

Frontend Dashboard

Use WebSocket for real-time updates, as it allows bi-directional communication between the client and server with minimal overhead.
Employ long-polling if WebSockets are not available or supported by all clients.

Backend Service

Implement the backend service using a microservices approach to ensure modularity and scalability.
Use stateless design principles to allow for easy scaling through horizontal partitioning.
Employ a load balancer to distribute incoming requests across multiple instances of the backend service.

Database/Cache Layer

#### Cons
ider using a caching layer like Redis to store order counts, reducing read latency and providing near-instantaneous updates.
Use an in-memory database or NoSQL solution for fast write operations and low-latency reads.
Implement a background job system (e.g., scheduled tasks) to periodically flush cache data back to the database.

Order Processing

Ensure that order processing is done using a reliable message broker like Apache Kafka, ensuring that no orders are lost in case of failures.
Use a distributed ledger or event sourcing mechanism to maintain consistent order counts across multiple instances.

Monitoring and Alerting

Implement robust monitoring and alerting systems (e.g., Prometheus, Grafana) to ensure the system is running smoothly and alert on any issues.
Define SLAs for response time and availability based on business requirements.

#### Closing Summary

To summarize, designing a real-time dashboard involves creating a scalable, fault-tolerant, and efficient backend service that can handle high volumes of order counts. The frontend needs to update in near real-time, which can be achieved using WebSockets or long-polling. A caching layer is essential for reducing read latency, while robust data processing and monitoring systems ensure the reliability and performance of the overall system.

#### Common Mistakes/Follow-Ups
Using a Single Database: Avoid relying solely on a database for real-time updates due to high latency.
Ignoring Scalability: Ensure that your design can scale horizontally by using microservices and load balancers.
Insufficient Monitoring: Not implementing proper monitoring and alerting systems can lead to unnoticed failures.
Code Blocks
PYTHON
# Example of a WebSocket setup in Python using asyncio
import asyncio

async def handle_client(websocket, path):
    while True:
        order_count = await get_order_count()  # Assume this function fetches the order count from cache
        await websocket.send(f"Current Order Count: {order_count}")

start_server = websockets.serve(handle_client, "localhost", 8765)

asyncio.get_event_loop().run_until_complete(start_server)
asyncio.get_event_loop().run_forever()

