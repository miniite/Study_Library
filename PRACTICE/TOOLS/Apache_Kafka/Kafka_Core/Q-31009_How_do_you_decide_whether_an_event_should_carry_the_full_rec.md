---
type: "practice"
difficulty: "Basic"
---

# How do you decide whether an event should carry the full record or just an ID that the pipeline looks up later?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Quarantine Topics](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Quarantine_Topics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide whether an event should carry the full record or just an ID to be looked up later in a Kafka-based pipeline, we need to balance several factors including data redundancy, processing speed, storage overhead, and maintainability. The key idea is to optimize for the overall efficiency of the system while minimizing potential issues like latency and data integrity.

Firstly, let's triage the question: the core aspect here is understanding the trade-offs between carrying full records versus IDs in Kafka messages. The interviewer might be probing your ability to think through a real-world scenario, considering performance, storage requirements, and operational complexity.

#### Assumptions

The system uses Apache Kafka as its messaging framework.
Full records are larger than simple ID references.
The pipeline involves multiple steps or systems beyond just the initial event source and sink.

Step-by-step reasoning from first principles:

Size of Data: If full records are significantly larger, transmitting them can increase network latency and processing time, which might be critical in real-time applications.

Processing Speed vs. Storage Overhead: Carrying IDs allows for faster processing since the lookup step is generally quicker than deserializing a full record. However, it increases storage overhead on the system that performs the lookups.

Data Integrity and Latency: If the ID-based approach introduces additional latency due to the required lookups, this could affect the real-time nature of your application. Additionally, if the data sources are not reliable or if there's a risk of IDs becoming stale, full records might be necessary for accuracy.

System Complexity: Managing IDs and ensuring they map correctly can add complexity in error handling, especially in distributed systems where nodes might fail.

Scalability Considerations: In large-scale applications, the number of lookups required could become a bottleneck. Carrying full records may scale better if the data volume is manageable.

#### Cons
idering these factors:

If the application requires immediate access to detailed information and can handle potential delays due to larger payloads, it might make sense to carry full records.
For high-throughput real-time applications where minimizing latency and maximizing speed of processing are crucial, ID-based lookups could be more appropriate. However, this approach necessitates robust and performant lookup mechanisms.

#### Closing summary
The decision hinges on balancing these trade-offs based on the specific requirements and constraints of your application. A well-designed system would ideally use a combination of both strategies—carrying full records for critical use cases while using IDs where real-time processing is paramount.

#### Common mistakes
include:

Ignoring the size implications of carrying full records.
Not considering the complexity introduced by additional lookups.
Underestimating the importance of data integrity and latency in mission-critical applications.

**Comparison:**

| Key Consideration | Full Record Carried | ID Lookup |
|-------------------|---------------------|-----------|
| **Data Size**     | Larger payloads     | Smaller   |
| **Processing Speed** | Slower due to deserialization  | Faster due to lookups  |
| **Storage Requirements**  | Higher on producers and consumers  | Lower on producers; higher on lookup system  |
| **Latency**       | Can be high        | Low to moderate  |

**Code:**

```python
# Example code to illustrate simple ID-based lookup
def process_event(event_id):
    # Simulate a database query for fetching the full record based on event_id
    full_record = get_full_record_from_db(event_id)
    return process_full_record(full_record)

# Dummy function representing processing of the full record
def process_full_record(record):
    print(f"Processing: {record}")
```

**Example input / output:**

```

Input: {"event_id": "12345"}

Output:

CODE
Copy
Processing: {
    "id": "12345",
    "user_name": "JohnDoe",
    "email": "john.doe@example.com",
    "created_at": "2023-10-01T12:00:00Z"
}
```

**Data:**

```json
{
  "event_id": "12345",
  "full_record": {
    "id": "12345",
    "user_name": "JohnDoe",
    "email": "john.doe@example.com",
    "created_at": "2023-10-01T12:00:00Z"
  }
}
```

**Flow:**

1. Yes
2. No
3. Lookup Successful
4. Lookup Failed
5. Event Received
6. Is Full Record Required?
7. Process Full Record Directly
8. Store ID, Process Later
9. Handle Full Record Processing
10. Lookup Full Record
11. Process Lookup Result
12. Retry or Handle Error

