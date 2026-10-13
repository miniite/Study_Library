---
type: "practice"
difficulty: "Deep"
---

# Your batch job is O(n) but consumes O(n) memory and OOMs at scale; how would you redesign it to bounded/constant memory while keeping throughput acceptable?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Streaming vs In-memory Space](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Streaming_vs_In-memory_Space.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle this problem, we need to redesign a batch job that is currently (O(n)) in both time and space complexity, but consumes unbounded memory leading to Out-of-Memory (OOM) errors at scale. Our goal is to optimize it for bounded or constant memory while maintaining acceptable throughput.

Triaging the Problem

The core of this question asks us to improve the job’s memory usage without significantly compromising its performance. The interviewer is probing our understanding of algorithmic optimization, space-time trade-offs, and how to handle large datasets efficiently.

#### Assumptions
and Initial Analysis
Current Job: (O(n)) time complexity means it processes each element once.
Memory Consumption: (O(n)) memory usage implies the job stores significant state or intermediate results in memory.
OOM Error: This suggests the current implementation is not scalable for large datasets.

#### First-Principles Solution

To redesign the job, we need to break down the problem and identify where improvements can be made. Here are the steps:

Identify Redundant Data Structures: Review the data structures used. Are there any that can be replaced with more space-efficient alternatives?
Stream Processing: Convert batch processing into stream processing if applicable.
Partial Results Storage: Store only necessary intermediate results and ensure they do not grow unbounded.
Step-by-Step Solution
1. Review Data Structures

Let’s assume the current implementation uses an array or list to store all elements. This can be replaced with a generator function that processes data one at a time, eliminating memory usage issues.

PYTHON
def process_data_stream(data_source):
    for item in data_source:
        # Process each item here
2. Stream Processing

Convert the batch job into a stream job where we handle data incrementally rather than storing everything in memory. This can be achieved using generator functions or streaming libraries like Apache Kafka Streams.

PYTHON
from kafka import KafkaProducer

def process_stream(stream):
    producer = KafkaProducer(bootstrap_servers='localhost:9092')
    for item in stream:
        # Process each item here
        result = process_item(item)
        producer.send('output-topic', value=result)

# Example usage
process_stream(data_source)
3. Partial Results Storage

Store only necessary intermediate results, and ensure they are managed efficiently. For example, if sorting or aggregating data is required, consider using in-place algorithms or external sort techniques.

PYTHON
def partial_sort(stream):
    sorted_list = []
    for item in stream:
        # Maintain a small window of elements to keep memory bounded
        if len(sorted_list) < 1000:
            sorted_list.append(item)
        else:
            sorted_list.sort()
            yield min(sorted_list)

# Example usage
for result in partial_sort(data_source):
    process_result(result)

#### Closing Summary
and Common Mistakes

By converting batch processing to stream processing, we can achieve bounded memory usage while maintaining acceptable throughput. Common mistakes include:

Ignoring Streaming Techniques: Not recognizing the benefits of streaming for large datasets.
Overcomplicating Algorithms: Failing to simplify algorithms where in-place or space-efficient techniques exist.
Not Considering External Storage: Overlooking external storage solutions like databases or distributed file systems.
Common Follow-Ups
How would you handle very large data sizes?
What if the job needs to maintain state across batches?
Can you provide an example of a similar real-world scenario?

**Code:**

```python
def process_data_stream(data_source):
    for item in data_source:
        # Process each item here

from kafka import KafkaProducer

def process_stream(stream):
    producer = KafkaProducer(bootstrap_servers='localhost:9092')
    for item in stream:
        result = process_item(item)
        producer.send('output-topic', value=result)

def partial_sort(stream):
    sorted_list = []
    for item in stream:
        if len(sorted_list) < 1000:
            sorted_list.append(item)
        else:
            sorted_list.sort()
            yield min(sorted_list)
```

**Example input / output:**

```text
Input Stream:
- item1, item2, ...

Process Stream Output (Kafka Producer):
- Sent to 'output-topic' with processed data

Partial Sort Example:
- Input Stream: ["item3", "item4", ...]
- Processed Result: [min(sorted_list)]
```

**Data:**

```json
{
    "data_source": ["item1", "item2", ...],
    "output_topic": "output-topic"
}
```

