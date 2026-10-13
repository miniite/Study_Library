---
type: "practice"
difficulty: "Basic"
---

# Design a system to send SMS alerts when an order status changes.

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Notification System](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Notification_System.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a system that sends SMS alerts when an order status changes, we need to consider several key components and their interactions. The core idea is to ensure timely and reliable communication between our e-commerce platform's backend services and the SMS gateway provider. Let’s break this down step-by-step.

#### Triaging the Question

The question asks us to design a system for sending SMS alerts when an order status changes. This means we need to focus on how orders are managed, what triggers the change in status, and how these status changes get communicated via SMS. The interviewer is likely probing your understanding of event-driven architectures, message queues, and integration with external services.

#### Assumptions
Order Status Change Triggers: There’s a backend service that manages order statuses.
SMS Gateway Provider: We use an external SMS gateway API to send messages.
Reliability and Scalability: The system should handle high volumes of orders without failing or losing alerts.
Step-by-Step Solution

Event Triggering:

When the status of an order changes, a webhook is triggered in our backend service.

Message Queue:

We use a message queue (e.g., RabbitMQ) to decouple the order processing service from the SMS sending logic. This ensures that even if the SMS service fails, orders can still be processed.

Order Processing Service:

The order status change is detected by this service.
It constructs an appropriate payload with the necessary details (order ID, new status, recipient’s phone number) and sends it to the message queue.

SMS Gateway Service:

An SMS gateway service listens for messages on the queue.
When a message arrives, it extracts the order information and sends the SMS through its API.

Database and Synchronization:

A database (like PostgreSQL) is used to store order statuses and any relevant metadata.
Proper synchronization mechanisms ensure that the order status in the database reflects the latest changes.

#### Closing Summary

By using a message queue, we can ensure that SMS alerts are sent reliably even if there are temporary issues with the external SMS gateway. The decoupling of services also makes the system more maintainable and scalable. However, it’s crucial to implement proper error handling and retries to avoid losing important messages.

#### Common Mistakes/Follow-Ups
Ignoring Error Handling: Make sure your system handles retries for failed deliveries.
Insufficient Synchronization: Ensure that order statuses are always updated correctly in the database.
Lack of Monitoring: Implement logging and monitoring to detect and resolve issues quickly.

**Comparison:**

```
Component	Description
Order Processing Service	Detects order status changes and sends messages to the message queue.
Message Queue (e.g., RabbitMQ)	Stores and processes order status change notifications.
SMS Gateway Service	Listens for messages on the queue and sends out SMS alerts.
```

**Flow:**

1. Yes
2. No
3. Order Status Change Detected
4. Is Order ProcessingSucceeded?
5. Send Message to Queue
6. Message Queue ProcessesNotification
7. SMS Delivery Success?
8. Retry Sending SMS
9. Notify Success/Failed Status

