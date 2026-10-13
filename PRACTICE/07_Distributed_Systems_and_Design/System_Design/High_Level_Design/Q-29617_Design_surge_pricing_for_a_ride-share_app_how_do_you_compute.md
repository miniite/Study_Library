---
type: "practice"
difficulty: "Deep"
---

# Design surge pricing for a ride-share app — how do you compute demand/supply ratios per geo-zone in near-real-time and roll out price changes consistently to clients?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Real-time Price Feeds](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Real-time_Price_Feeds.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Surge pricing involves dynamically adjusting prices based on supply and demand. For a ride-share app, we need to design a system that computes real-time demand/supply ratios per geo-zone and uses these ratios to trigger and apply price changes consistently across multiple clients. The key steps involve data collection, processing, decision-making, and consistent client updates.

#### Triaging the Question

The question requires designing a high-level architecture for surge pricing in a ride-share app. This involves:

Real-time demand/supply computation per geo-zone.
Decision logic to determine if prices should be increased based on the demand/supply ratios.

#### Cons
istent client updates when price changes are made.

#### Assumptions
and Key Concepts
Geo-zones: The app operates in different geographical regions or zones.
Demand/Supply Ratios: These are calculated by monitoring the number of rides requested (demand) versus available drivers (supply).
Real-Time Data Collection: Data on demand, supply, and other metrics need to be collected and processed in near-real-time.

#### Cons
istent Client Updates: The system needs to notify clients about price changes in a coordinated manner.

#### First Principles Solution

Data Collection

Collect real-time data from various sources such as ride requests, driver locations, weather conditions, and time of day.
Use APIs or streaming services like Kafka for near-real-time updates.

Demand/Supply Calculation

For each geo-zone, calculate the demand (number of rides requested) and supply (available drivers).
Compute a demand/supply ratio to determine if there is high demand relative to supply, indicating potential surge pricing.

Decision Logic

Define thresholds for when prices should be increased or decreased.
Implement an algorithm that decides on price changes based on the calculated ratios and predefined rules.

#### Cons
istent Client Updates

Use a message queue system like RabbitMQ to ensure consistent and reliable delivery of updates to all clients.
Design a client-side mechanism to handle price change notifications, ensuring they are applied correctly and in sync with other clients.
Detailed Solution

Data Collection Layer

Ride Requests API: A REST API that collects ride requests from users.
JSON
POST /requests
{
  "userId": "user123",
  "pickupLocation": "New York, NY",
  "dropoffLocation": "Central Park, NY"
}
Driver Locations API: A REST API that reports the current location of drivers.
JSON
POST /driver-locations
{
  "driverId": "driver456",
  "location": "Times Square, NY"
}

Real-Time Data Processing

Use a real-time streaming platform like Apache Kafka to process data in near-real-time.
BASH
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic ride-requests
Define schemas for the collected data using Avro or Protobuf.

Demand/Supply Calculation

Use a microservice that aggregates and computes demand/supply ratios.
PYTHON
# Python snippet to calculate demand/supply ratio
def calculate_ratio(rides, drivers):
    return len(rides) / len(drivers)

Decision Logic

Implement an algorithm that checks if the calculated ratio exceeds predefined thresholds.
PYTHON
# Example threshold values
HIGH_DEMAND_THRESHOLD = 3.0
LOW_SUPPLY_THRESHOLD = 2.0

def should Surge(price_ratio):
    return price_ratio > HIGH_DEMAND_THRESHOLD and drivers < LOW_SUPPLY_THRESHOLD

#### Cons
istent Client Updates

Use a message queue system like RabbitMQ to ensure consistent updates.
BASH
bin/rabbitmq-server start
Design a client-side mechanism that subscribes to price change notifications.
PYTHON
# Python snippet for subscribing to price changes
import pika

def on_message_received(ch, method, properties, body):
    print("Price changed: %s" % body)

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()
channel.queue_declare(queue='price_changes')
channel.basic_consume(on_message_received, queue='price_changes')
channel.start_consuming()

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Closing Summary
The solution involves collecting real-time data, processing it to compute demand/supply ratios, making decisions based on these ratios, and ensuring consistent client updates.

#### Common Mistakes
Failing to account for edge cases such as sudden spikes in demand or supply changes.
Not using a reliable message queue system, leading to inconsistent client updates.
Ignoring the need for robust error handling and fallback mechanisms.

#### Follow-Ups
How would you handle situations where there are no available drivers?
What other factors could influence surge pricing besides demand/supply ratios?
How can you ensure that price changes are applied consistently across all clients?

**Comparison:**

```
Geo-Zone	Demand	Supply	Ratio
Midtown	1 request	1 driver	1.0
```

**Code:**

```python
# Example code snippets for calculation and decision logic
```

**Example input / output:**

```text
Input:
Ride request: user123 from New York to Central Park.
Driver location update: driver789 is at Midtown.

Output:
Calculated demand/supply ratio for the midtown zone is 2.0.
Decision logic determines that no surge pricing change is needed.
```

**Data:**

```json
{
  "ride_requests": [
    {"userId": "user123", "pickupLocation": "New York, NY", "dropoffLocation": "Central Park, NY"},
    {"userId": "user456", "pickupLocation": "Times Square, NY", "dropoffLocation": "Manhattan Bridge, NY"}
  ],
  "driver_locations": [
    {"driverId": "driver789", "location": "Midtown, NY"},
    {"driverId": "driver101", "location": "Upper West Side, NY"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Collect Ride Requests
4. Process Data
5. Compute Demand/SupplyRatio
6. Decision Logic
7. Notify Clients of Price Change
8. No Action
9. Queue Message for Clients

