---
type: "practice"
difficulty: "Basic"
---

# What is Change Data Capture (CDC) and why is it useful in a data pipeline?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ CDC Fundamentals and Ordering](<../../../../02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/CDC_Fundamentals_and_Ordering.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Change Data Capture (CDC) is a technique used to track changes in database records over time, making it invaluable in maintaining up-to-date data pipelines. Essentially, CDC allows you to efficiently capture only the modifications that occur in real-time, ensuring that your downstream systems always have the most recent and accurate data.

The core idea behind CDC is simple yet powerful: instead of copying entire tables or datasets, you focus on the changes that are actually happening. This can significantly reduce the amount of data transferred and processed, making your pipelines more efficient and scalable.

Let's break down why CDC is useful in a data pipeline:

Tracking Changes: CDC records only the insertions, updates, and deletions made to a database, allowing you to maintain an audit trail or continuously sync data between different systems.
Efficiency: By capturing only changes, CDC reduces the amount of data that needs to be processed, which can save significant computational resources.
Real-Time Updates: It ensures that your data pipelines stay up-to-date with minimal latency, making it ideal for applications that require real-time insights or analysis.

Now let's dive into how CDC works and why it’s so useful:

#### First Principles Solution
1. Identifying Change Events

To implement CDC, you first need to identify which events (inserts, updates, deletes) are happening in your database. This is typically done using database-specific features like triggers or log files.

For example, in PostgreSQL, you can use the INSERT, UPDATE, and DELETE statements to modify a table. You might also have TRUNCATE operations that affect large portions of data at once.

2. Capturing Change Events

Once change events are identified, they need to be captured efficiently. This can be done by:

Using database triggers: These are database objects that automatically execute when a specific event occurs.
Utilizing log files: Most databases maintain transaction logs (like the Write-Ahead Log in PostgreSQL) that record all changes.
3. Transforming and Storing Changes

The captured change events need to be transformed into a format suitable for your data pipeline. This might involve:

Writing these changes to an external storage system, like Apache Kafka or AWS Kinesis.
Transforming the raw events (like timestamps, IDs) into more structured formats.
4. Synchronizing Data

Finally, the captured changes are used to sync data in real-time between different systems. This might involve:

Synchronizing a read replica of your primary database with secondary systems.
Using incremental updates to keep tables or datasets up-to-date.

#### Example
Workflow

Imagine you have an e-commerce platform with a MySQL database. Every time a product is updated, deleted, or new products are added, CDC would capture these changes and send them to a Kafka topic. Your data pipeline then processes this stream of events in real-time, ensuring that the data stored in your analytics databases remains accurate.

#### Closing Summary

In summary, Change Data Capture (CDC) enables efficient management of real-time data by focusing on what actually changes rather than everything. This makes it incredibly useful for keeping data pipelines up-to-date and scalable. Common pitfalls include not properly setting up triggers or log files, and failing to account for edge cases like cascading deletes.

#### Common Mistakes/Follow-Ups
Not Handling Edge Cases: Forgetting to handle complex transactions, such as nested updates or deletes.
Choosing the Wrong Tool: Selecting a tool that doesn’t support CDC natively can complicate implementation.
Latency Issues: Ensuring minimal delay between changes in the source database and their reflection in downstream systems.

**Comparison:**

```
tableDiagram
    Product_changes -->|ACTION| Triggers
    Triggers ----> CDC_systems
```

**Code:**

```sql
-- Example of creating a trigger for CDC in PostgreSQL
CREATE OR REPLACE FUNCTION public.product_change_log()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'DELETE' THEN
        -- Log delete event
        INSERT INTO product_changes (product_id, action, timestamp) VALUES (OLD.product_id, 'delete', NOW());
    ELSIF TG_OP = 'INSERT' OR TG_OP = 'UPDATE' THEN
        -- Log insert or update event
        IF NEW.price <> OLD.price THEN
            INSERT INTO product_changes (product_id, price, timestamp) VALUES (NEW.product_id, NEW.price, NOW());
        END IF;
    END IF;
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;

-- Attaching the trigger to a table
CREATE TRIGGER product_change_trigger
AFTER INSERT OR UPDATE OR DELETE ON products
FOR EACH ROW EXECUTE FUNCTION product_change_log();
```

**Example input / output:**

```text
Input:
- Product price update event

Output:
- {"product_id": 12345, "action": "update", "timestamp": "2023-10-01T12:00:00Z", "price": 9.99}
```

**Data:**

```json
{
  "product_changes": [
    {
      "product_id": 12345,
      "action": "update",
      "timestamp": "2023-10-01T12:00:00Z",
      "price": 9.99
    },
    {
      "product_id": 67890,
      "action": "delete",
      "timestamp": "2023-10-01T14:30:00Z"
    }
  ]
}
```

**Flow:**

1. Source_DB
2. Captures Changes
3. Transforms and ProcessesEvents
4. Synchronizes Real-Time Data

