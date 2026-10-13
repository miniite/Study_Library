---
type: "practice"
difficulty: "Deep"
---

# How do watermarks in a streaming SQL engine decide when a window closes, and what happens to a late event that arrives after the watermark and the allowed lateness both pass?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ SKIP LOCKED Job Queues](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/SKIP_LOCKED_Job_Queues.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Watermarks in a streaming SQL engine are crucial for handling out-of-order events and ensuring timely processing of data within time windows. The core concept revolves around defining when a window is considered "closed" based on the watermark, which tracks the maximum timestamp seen so far. When a late event arrives after both the watermark and the allowed lateness period have passed, it determines whether this event should be processed in the current or subsequent window.

To fully understand watermarks and their behavior, we need to break down the problem from first principles: what is a watermark, how does it evolve over time, and what happens when events are late. We will also explore common scenarios and pitfalls.

**Comparison:**

| Time       | Event   | Watermark    | Decision                   |
|------------|---------|--------------|----------------------------|
| 2023-10-01T14:05:00.000Z | Order 1 | "2023-10-01T14:05:00.000Z" | Close current window, process Event 1 |
| 2023-10-01T14:10:00.000Z | Order 2 | "2023-10-01T14:10:00.000Z" | No window closes, process Event 2 in the current window |

**Code:**

```sql
-- Example of setting up a watermark in Apache Flink SQL (Pseudo-code)
CREATE TABLE orders (
  order_id INT,
  customer_name STRING,
  order_time TIMESTAMP(3),
  WATERMARK FOR order_time AS order_time - INTERVAL '5' SECONDS;
);
```

**Example input / output:**

```text
Input:
- Order at time T = "2023-10-01T14:05:00.000Z" (order_id=1)
- Order at time T + 5s = "2023-10-01T14:10:00.000Z" (order_id=2)

Watermark logic:
- Watermark initially set to 0 seconds
- When an event arrives, the watermark is updated to the maximum timestamp of all events seen so far

Output at time T + 5s:
- Window closes for orders with timestamps <= "2023-10-01T14:05:00.000Z" (order_id=1)
- Event with order_id=2 is processed in the current window
```

**Data:**

```json
{
  "orders": [
    { "order_id": 1, "customer_name": "Alice", "order_time": "2023-10-01T14:00:00.000Z" },
    { "order_id": 2, "customer_name": "Bob", "order_time": "2023-10-01T14:05:00.000Z" },
    { "order_id": 3, "customer_name": "Charlie", "order_time": "2023-10-01T14:10:00.000Z" }
  ]
}
```

**Flow:**

1. Time Windows
2. Yes
3. No
4. Event arrives
5. Is event late?
6. Watermark > Allowed Lateness
7. Close current window, processevent if within allowed lateness
8. Watermark <= AllowedLateness
9. Process event in the currentwindow
10. Window closes at watermarktime

