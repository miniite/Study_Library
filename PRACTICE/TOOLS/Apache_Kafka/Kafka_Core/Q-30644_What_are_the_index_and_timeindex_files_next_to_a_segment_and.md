---
type: "practice"
difficulty: "Expert"
---

# What are the .index and .timeindex files next to a segment, and how do they speed up offset and timestamp lookups?

**Difficulty:** Expert · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Claim Check Pattern](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Claim_Check_Pattern.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Apache Kafka, segments (also known as partition segments) in the log directory store the actual message data. However, alongside each segment file are two auxiliary files: .index and .timeindex. These files significantly enhance the performance of offset and timestamp lookups by providing a compact and efficient way to locate specific messages based on offsets or timestamps.

Firstly, let's break down what these files do:

The .index file is a mapping of message offsets to their corresponding positions within the segment. This allows Kafka to quickly find the byte position in the log where a given offset starts.
The .timeindex file maps timestamps to offsets and also provides the number of messages with that timestamp, helping to locate records more accurately.

Now, let's dive into how these files work together:

Index File (.index):

The index file is a binary file containing an array of OffsetMetadata objects. Each object stores the offset along with the corresponding byte position in the segment.
This allows Kafka to perform binary searches on the index, making lookups extremely fast even as the log grows.

TimeIndex File (.timeindex):

The timeindex file is more complex and contains metadata for each timestamp present in the log.
It includes a TimestampMetadata object that maps timestamps to ranges of offsets and byte positions within the segment where those messages are stored.
This helps in quickly narrowing down the range of messages with a specific timestamp, further speeding up lookups.

By utilizing both .index and .timeindex, Kafka can efficiently find records based on either their offset or timestamp. For example:

To retrieve a message by offset: Use binary search on the index file.
To retrieve messages by time: Use the timeindex to quickly identify the segment and offset range, then use the index for precise location.

Here's an example of how these files might be used in practice:

PLAINTEXT
User requests a message with timestamp 1234567890.
TimeIndex file is consulted first.
It returns that messages between offsets 100-200 have this timestamp.
Then, the .index file for segment containing offset 150 (estimated from timeindex) is used to find exact byte position within the segment.

This approach ensures that lookups are highly performant even as Kafka clusters handle massive volumes of data.

