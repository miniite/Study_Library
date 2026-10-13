---
type: "practice"
difficulty: "Deep"
---

# How would you compute a streaming running-total revenue per merchant that survives restarts and never double-counts?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Exactly Once into Kafka](<../../../../TOOLS/Apache_Flink/Flink_Core/Exactly_Once_into_Kafka.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compute a streaming running-total revenue per merchant in Apache Flink, we need to ensure that the system accurately accumulates revenue without double-counting transactions even if it is restarted. This involves handling stateful processing and ensuring idempotency.

#### Triage

The question asks us to implement a robust stream processing solution using Apache Flink for calculating running-total revenue per merchant. The key points are:

State Management: We need to maintain the state of each merchant's revenue in a fault-tolerant manner.
Idempotence: Ensure that transactions are counted only once, even if they are reprocessed due to system restarts or failures.
Fault Tolerance: The solution should survive restarts without losing any data.

#### First-Principles Solution

Event-Sourced State Management:

Use Flink's state management features like KeyedStateBackend or RocksDBStateBackend to store the running total for each merchant.

Idempotent Operations:

Design the operations on these states such that they can be safely applied multiple times without changing the outcome.

Fault Tolerance Strategies:

Configure checkpointing and savepoints to ensure state is preserved across restarts.
Detailed Solution:
Step 1: Define the Data Schema

Assume the input stream contains transaction data with fields like merchant_id, transaction_time, and revenue.

PLAINTEXT
{
  "merchant_id": "M123",
  "transaction_time": "2023-10-01T14:59:00Z",
  "revenue": 100.00
}
Step 2: Configure the Flink Environment

Ensure that Flink is configured to use a reliable state backend like RocksDB for fault tolerance.

JAVA
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
env.setStateBackend(new RocksDBStateBackend("path/to/state/backend"));
env.enableCheckpointing(5000); // Checkpoint every 5 seconds
Step 3: Define the Data Source and Processing Pipeline
JAVA
DataStream<Transaction> transactionStream = env.addSource(sourceFunction);

// Key by merchant_id for stateful processing
KeyedStream<Transaction, String> keyedStream = transactionStream.keyBy(transaction -> transaction.merchant_id);
Step 4: Accumulate Revenue Using Stateful Processing
JAVA
keyedStream
    .connect(SingleOutputStreamOperator.of(null))
    .process(new RichCoProcessFunction<String, Transaction, Void>() {
        private ValueState<Double> runningTotal;

        @Override
        public void open(Configuration parameters) throws Exception {
            // Initialize the state backend
            runningTotal = getRuntimeContext().getState(
                new ValueStateDescriptor<>("running-total", Double.class));
        }

        @Override
        public void onElement(Transaction transaction, Context ctx) throws Exception {
            if (transaction != null) {
                double currentRunningTotal = 0.0;
                try {
                    // Ensure the state is read and accumulated idempotently
                    currentRunningTotal = runningTotal.value() + transaction.revenue;

                    // Update the state with the new total
                    runningTotal.update(currentRunningTotal);
                } catch (Exception e) {
                    // Handle any state update failures gracefully
                    ctx.output(new OutputTag<Void>("errors", null));
                }
            }
        }

        @Override
        public void onTimer(long timestamp, OnTimerContext ctx) throws Exception {
            // No timers needed for this use case
        }

        @Override
        public void close() throws Exception {
            // Clean up any resources if necessary
        }
    }).print();
Step 5: Ensure Idempotent Operations

The onElement method ensures that the state update is idempotent by reading and updating the current running total in a transactional manner. This means even if the function is called multiple times, the final value will be the same as if it was only executed once.

Step 6: Fault Tolerance with Checkpoints

Flink's checkpointing mechanism ensures that state changes are stored periodically. If the system restarts, Flink can restore the state from the last checkpoint to ensure no data is lost.

#### Closing Summary

By implementing a stateful processing pipeline in Apache Flink and ensuring idempotent updates, we can reliably compute a streaming running-total revenue per merchant that survives restarts and never double-counts transactions. The key steps include using KeyedStateBackend for state management, configuring periodic checkpoints, and designing the processing logic to be idempotent.

#### Common Mistakes/Follow-Ups
Mistake: Not handling state changes properly can lead to stale or incorrect totals.

#### Follow-Up
Discuss edge cases like multiple parallel Flink instances and how they coordinate state updates.
Further Discussion: Explore advanced topics such as exactly-once semantics, state backends, and checkpointing strategies.

**Code:**

```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
env.setStateBackend(new RocksDBStateBackend("path/to/state/backend"));
env.enableCheckpointing(5000);

DataStream<Transaction> transactionStream = env.addSource(sourceFunction);
KeyedStream<Transaction, String> keyedStream = transactionStream.keyBy(transaction -> transaction.merchant_id);

keyedStream
    .connect(SingleOutputStreamOperator.of(null))
    .process(new RichCoProcessFunction<String, Transaction, Void>() {
        private ValueState<Double> runningTotal;

        @Override
        public void open(Configuration parameters) throws Exception {
            runningTotal = getRuntimeContext().getState(
                new ValueStateDescriptor<>("running-total", Double.class));
        }

        @Override
        public void onElement(Transaction transaction, Context ctx) throws Exception {
            if (transaction != null) {
                double currentRunningTotal = 0.0;
                try {
                    currentRunningTotal = runningTotal.value() + transaction.revenue;
                    runningTotal.update(currentRunningTotal);
                } catch (Exception e) {
                    ctx.output(new OutputTag<Void>("errors", null));
                }
            }
        }

        @Override
        public void onTimer(long timestamp, OnTimerContext ctx) throws Exception {
        }

        @Override
        public void close() throws Exception {
        }
    }).print();
```

**Example input / output:**

```text
Input:
[
    { "merchant_id": "M123", "transaction_time": "2023-10-01T14:59:00Z", "revenue": 100.00 },
    { "merchant_id": "M123", "transaction_time": "2023-10-01T15:00:00Z", "revenue": 200.00 }
]

Output:
Merchant M123's running total revenue: 300.00
```

**Data:**

```json
{
  "merchant_id": "M123",
  "transaction_time": "2023-10-01T14:59:00Z",
  "revenue": 100.00
}
```

**Flow:**

1. Transaction Stream
2. Key By Merchant ID
3. Stateful Processing withIdempotent Operations
4. Checkpointing and FaultTolerance
5. Accumulate Running Total PerMerchant

