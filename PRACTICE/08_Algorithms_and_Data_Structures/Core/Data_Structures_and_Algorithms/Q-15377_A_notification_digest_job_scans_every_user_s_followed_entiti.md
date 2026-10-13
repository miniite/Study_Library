---
type: "practice"
difficulty: "Deep"
---

# A notification digest job scans every user's followed entities nightly and takes 12 hours; how would you invert it with a per-entity subscriber index so work scales with what actually changed?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Sliding Window Aggregates](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Sliding_Window_Aggregates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this problem, we need to rethink our approach from scanning every user's followed entities nightly to an efficient system that only processes changes. This involves creating a per-entity subscriber index that allows us to focus on updates made by users or entities. The goal is to minimize the overall workload and ensure that the job scales gracefully with growing numbers of users, entities, and changes.

#### Triage

The core question here is: How can we optimize the notification digest job so it only processes what has changed instead of scanning all user subscriptions every night? This requires understanding the current data structure, identifying bottlenecks, and designing a new architecture that leverages an index to track changes efficiently.

#### Assumptions

Each user follows multiple entities.
Entities can change frequently (e.g., a post gets updated).
The system needs to send notifications based on these changes.
We aim for a scalable solution that handles increased loads without requiring constant full scans.

#### First Principles Solution
Current System Analysis: The current 12-hour nightly job scans all user subscriptions, which is inefficient and does not scale well with growth.
Per-Entity Subscriber Index: Introduce an index where each entity (e.g., a post) maintains a list of users who are following it. This allows us to track changes on entities independently from the users.
Change Tracking: Implement a mechanism to detect and process only the updates by maintaining a history or timestamp for each change.
Efficient Processing: Use batch processing techniques to handle notifications efficiently, ensuring that we don't overwhelm our system with too many requests at once.
Step-by-Step Solution

Update Entity Tracking:

For every user-subscribed entity, maintain a mapping from the entity ID to a list of users who are following it.

Change Logging:

Whenever an entity changes (e.g., new post), log this change with a timestamp and any relevant metadata.

Notification Queue:

Instead of processing all subscriptions nightly, set up a notification queue that processes updates as they occur or in batches.
For each update, check the subscriber index to identify affected users and add them to the notification queue for sending notifications.

Batch Processing:

Implement batch processing jobs that run at regular intervals (e.g., every 5 minutes) to clear the notification queue and send out notifications.
Ensure these jobs are lightweight and can handle many small batches efficiently.

#### Closing Summary

By using a per-entity subscriber index and change tracking, we can significantly reduce the load on our system. This approach ensures that only relevant changes trigger notifications, reducing the overall processing time and making the job more scalable with growing data volumes.

#### Common Mistakes/Follow-Ups
Ignoring Change History: Failing to maintain an accurate history of updates can lead to missed or incorrect notifications.
Overhead in Batch Processing: Ensuring that batch jobs are efficient and don't overwhelm the system is crucial. Consider using asynchronous message queues (e.g., RabbitMQ, Kafka) for handling notification tasks.
Scalability Issues with Large Indices: As the number of entities grows, managing the subscriber index becomes more complex. Consider partitioning or sharding strategies to distribute the load.

**Comparison:**

```
Entity ID	Subscribers	Last Updated
123	[101, 105]	2023-10-01T14:00:00Z
```

**Code:**

```python
# Example code for updating the subscriber index
def update_subscriber_index(entity_id, user_ids):
    entity = get_entity_by_id(entity_id)
    if not entity:
        return

    for user_id in user_ids:
        add_user_to_entity_subscription(user_id, entity_id)

# Example function to process changes and notify subscribers
def process_entity_changes(changes):
    for change in changes:
        entity_id, timestamp, metadata = change
        subscribers = get_subscribers_for_entity(entity_id)
        for subscriber in subscribers:
            send_notification(subscriber, metadata)
```

**Example input / output:**

```text
Input:
- A batch of entity changes
- An index of subscribers for each entity

Output:
- Notifications sent to appropriate users based on changes in entities they follow
- Updated state of the subscriber index
```

**Data:**

```json
{
  "entities": {
    "entity123": {
      "subscribers": [101, 105],
      "last_updated": "2023-10-01T14:00:00Z"
    }
  },
  "changes": [
    {"entity_id": "entity123", "timestamp": "2023-10-02T15:00:00Z", "metadata": {"post_title": "New Post Title"}}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Entity Change Detected
4. Is Entity Already Tracked?
5. Update Subscriptions Index
6. Notify Subscribers via Queue
7. Process Queue Batch
8. Send Notifications
9. Add New Entity to Tracker
10. Log Change for FutureReference

