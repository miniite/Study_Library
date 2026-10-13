---
type: "practice"
difficulty: "Intermediate"
---

# Design a reminder system that notifies users at a chosen date and time.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Notification System](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Notification_System.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a reminder system that notifies users at a chosen date and time, we need to consider several key aspects: user interface, data storage, scheduling mechanism, delivery methods, scalability, and security. Let's break down each of these components step by step.

#### Triaging the Question

The core question is about designing a system that can accurately trigger notifications based on specific dates and times chosen by users. The interviewer will likely be probing your understanding of database design, asynchronous processing, scheduling algorithms, notification mechanisms, error handling, and system scalability.

#### Assumptions
Users can set reminders with custom date and time.
Notifications can be sent via various methods (email, SMS, push notifications).
The system needs to handle a large number of users and reminders.
Data privacy and security are critical concerns.

#### Solution
from First Principles

User Interface:

The UI should allow users to create reminders by specifying the date, time, and notification method.
Users should be able to view and manage their reminders, such as editing or deleting them.

Data Storage:

Use a database like PostgreSQL or MySQL to store reminder data.
Schema might include fields for user_id, reminder_date, reminder_time, notification_method, and status.
SQL
CREATE TABLE reminders (
    id SERIAL PRIMARY KEY,
    user_id INT NOT NULL,
    reminder_date DATE NOT NULL,
    reminder_time TIME NOT NULL,
    notification_method VARCHAR(50) NOT NULL,
    status VARCHAR(20) DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

Scheduling Mechanism:

Implement a background job that periodically checks for upcoming reminders.
Use a task queue like Celery or RQ to handle this asynchronously.

Notification Delivery:

Integrate with third-party services (like Twilio for SMS, SendGrid for emails) for sending notifications.
For push notifications, use platforms like Firebase Cloud Messaging (FCM).

Scalability and Performance:

Use a distributed architecture to handle high load.
Implement caching mechanisms to reduce database hits.

Security:

Ensure that user data is securely stored and transmitted.
Follow best practices for handling sensitive information, such as using secure connections (HTTPS).

#### Closing Summary

Designing a reminder system involves multiple components working together smoothly. The key elements include robust UI design, efficient data storage, reliable scheduling, versatile notification delivery, scalability considerations, and strong security measures.

#### Common Mistakes/Follow-Ups
Mistake: Focusing too much on the frontend without considering backend complexities.

#### Follow-Up
Discussing how to handle edge cases like overlapping reminders or very long intervals between reminders.

**Comparison:**

```
Field	Description
id	Unique identifier
user_id	ID of the user who set the reminder
reminder_date	Date when the reminder is due
reminder_time	Time when the reminder is due
notification_method	Method by which the notification will be sent (email, sms, push)
status	Status of the reminder (pending, sent)
```

**Code:**

```python
from datetime import datetime, timedelta
import schedule
import time

def send_notification(user_id, method):
    if method == 'email':
        # Send email using an SMTP service
        pass
    elif method == 'sms':
        # Use Twilio to send SMS
        pass
    elif method == 'push':
        # Use Firebase Cloud Messaging (FCM)
        pass

def check_for_reminders():
    current_time = datetime.now()
    reminders_to_send = Reminder.query.filter(Reminder.reminder_date <= current_time.date(),
                                             Reminder.reminder_time < current_time.time()).all()
    for reminder in reminders_to_send:
        send_notification(reminder.user_id, reminder.notification_method)
        reminder.status = 'sent'

schedule.every(10).seconds.do(check_for_reminders)  # Adjust interval as needed
```

**Example input / output:**

```text
Input:
- User sets a reminder for "2023-10-01 14:30" via email.

Output:
- Email is sent to the user at 14:30 on 2023-10-01.
```

**Data:**

```sql
CREATE TABLE reminders (
    id SERIAL PRIMARY KEY,
    user_id INT NOT NULL,
    reminder_date DATE NOT NULL,
    reminder_time TIME NOT NULL,
    notification_method VARCHAR(50) NOT NULL,
    status VARCHAR(20) DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Flow:**

1. No
2. Yes
3. User sets a reminder
4. Checks current time
5. Reminders are due?
6. Store reminder in database
7. Sends notification usingchosen method
8. Mark reminder as sent

