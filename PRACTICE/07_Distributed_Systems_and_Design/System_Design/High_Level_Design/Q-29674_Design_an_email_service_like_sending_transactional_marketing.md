---
type: "practice"
difficulty: "Deep"
---

# Design an email service (like sending transactional + marketing email at scale) — how do you handle sender reputation, bounce/complaint feedback, and per-domain throttling?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distributed Rate Limiter Design](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Distributed_Rate_Limiter_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To design a high-level system for handling transactional and marketing emails at scale, we must focus on three key areas: maintaining sender reputation, processing bounce and complaint feedback, and implementing per-domain throttling. We will leverage domain-specific SMTP servers, a scalable email processing pipeline, real-time analytics, and an automated feedback loop to ensure reliability and efficiency.

#### Triaging the Question

The question is asking how to design an email service that can handle both transactional and marketing emails at scale while ensuring high sender reputation, effectively managing bounces and complaints, and implementing per-domain throttling. The interviewer is probing your understanding of email infrastructure, scalability, reliability, and system architecture principles.

#### Assumptions
We are designing a system for a large enterprise with millions of users.
The system needs to handle both transactional (e.g., account verification) and marketing emails (e.g., newsletters).
Bounce rates and complaint rates must be monitored closely.
Per-domain throttling is necessary to prevent abuse and ensure fair usage.
Step-by-Step Solution
1. Email Infrastructure

We start by setting up a domain-specific SMTP server for each major domain. This ensures that emails from different domains can be processed independently, reducing the risk of cross-domain issues and allowing us to apply per-domain policies more effectively.

SMTP Server
Domain 1
Domain 2
Domain N
2. Email Pipeline

The email pipeline will be designed as a microservices architecture, where each step is modular and can be scaled independently.

Email Reception: SMTP servers receive incoming emails.
Queue Management: Emails are queued for processing using a message queue system like RabbitMQ or Kafka.
Validation & Categorization: Validate the email content (e.g., syntax, attachments) and categorize it as transactional or marketing.
Scheduling & Delivery: Use an asynchronous delivery mechanism to send emails at appropriate times. For transactionals, prioritize immediate delivery; for marketing, schedule based on user preferences.
Feedback Loop: Collect bounce and complaint feedback from email service providers (ESPs).
3. Sender Reputation Management

Sender reputation is crucial for maintaining deliverability. We will use a combination of blacklists, whitelists, and real-time analytics.

Email Server
Blacklist Service
Whitelist Service
Blacklist Service: Blocks emails from known spam sources.
Whitelist Service: Allows trusted senders to bypass certain checks.
Real-Time Analytics: Monitors email performance and updates sender reputation scores in real-time.
4. Bounce & Complaint Handling

Bounces and complaints must be handled promptly to maintain good sender reputation.

Email Service
Bounce Handler
Complaint Handler
Bounce Handler: Processes bounces, updates the email metadata with bounce information, and retries delivery if possible.
Complaint Handler: Tracks complaints using feedback loops from ESPs. Adjusts sender policies based on complaint rates to avoid being blacklisted.
5. Per-Domain Throttling

Throttling ensures fair usage across different domains and prevents abuse.

SMTP Server
Throttle Domain 1
Throttle Domain 2
Throttle Domain N
Throttle Service: Implements rate limiting based on historical data and current usage. Adjusts throttling dynamically to prevent abuse.

#### Closing Summary

In summary, designing a scalable email service involves setting up domain-specific SMTP servers, building a robust microservices architecture for email processing, managing sender reputation through real-time analytics, handling bounces and complaints effectively, and implementing per-domain throttling to ensure fair usage. This approach ensures high reliability, maintainable code, and efficient operation at scale.

#### Common Mistakes
Failing to properly categorize emails (transactional vs. marketing) can lead to improper handling.
Not using a message queue system can result in data loss or processing delays.
Ignoring real-time analytics for sender reputation management can degrade deliverability over time.
Inadequate throttling can lead to abuse and wasted resources.

**Comparison:**

| Domain | SMTP Server       | Throttle Settings    |
|--------|------------------|---------------------|
| A      | smtp.exampleA.com | Rate limit: 1000/day |
| B      | smtp.exampleB.com | Rate limit: 500/day  |

**Code:**

```python
# Example of a simple feedback handler function
def handle_feedback(email_id, is_bounce, is_complaint):
    if is_bounce:
        update_bounce_stats(email_id)
    elif is_complaint:
        update_complaint_stats(email_id)
```

**Example input / output:**

```text
Input:
- Email id: 12345, sender: user@example.com, recipient: customer@example.org, subject: Order Confirmation

Output:
- Bounce Handler updates bounce stats for email_id: 12345 with status: permanent and reason: mailbox full.
```

**Data:**

```json
{
  "Email": {
    "id": "12345",
    "sender": "user@example.com",
    "recipient": "customer@example.org",
    "subject": "Order Confirmation"
  },
  "Feedback": {
    "type": "bounce",
    "status": "permanent",
    "reason": "mailbox full"
  }
}
```

**Flow:**

1. Email Reception
2. Queue Management
3. Validation & Categorization
4. Scheduling & Delivery
5. Bounce & Complaint Handling
6. Per-Domain Throttling

