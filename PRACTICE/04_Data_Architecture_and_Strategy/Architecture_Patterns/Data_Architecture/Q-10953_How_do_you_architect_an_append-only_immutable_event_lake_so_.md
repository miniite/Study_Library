---
type: "practice"
difficulty: "Deep"
---

# How do you architect an append-only immutable event lake so that a bug producing corrupt events can be corrected without violating the immutability contract downstream consumers rely on? Contrast compaction-with-tombstones, correction-events, and versioned rewrites and their consumer-visible semantics.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ GDPR Deletion in Immutable Lakes](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/GDPR_Deletion_in_Immutable_Lakes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When architecting an append-only immutable event lake, ensuring that a bug producing corrupt events can be corrected without violating downstream immutability contracts is crucial. The key challenge here lies in maintaining data integrity while preserving the historical data lineage. This requires careful consideration of different approaches such as compaction-with-tombstones, correction-events, and versioned rewrites.

First, let's break down what each approach entails:

Compaction-With-Tombstones: This method involves periodically scanning the event lake to identify and mark corrupted events with tombstone entries. Tombstones indicate that a specific event is invalid or corrupt, allowing downstream systems to ignore it. However, this approach can introduce data staleness issues as it may take time for tombstones to propagate.

Correction-Events: This involves creating new events that correct the errors in the original corrupted events. These correction events are then appended to the event lake, ensuring that all downstream consumers see a consistent view of corrected data over time. The challenge here is ensuring that the sequence and semantics of events remain intact.

Versioned Rewrites: This approach involves rewriting entire segments of the event lake with updated versions of the corrupted events. However, this can be resource-intensive and disrupts immutability guarantees for downstream consumers who expect a read-only view of historical data.

Now, let's explore each method in more detail:

Compaction-With-Tombstones
Core Idea

#### Pros
Simple to implement; only requires marking invalid entries.

#### Cons
Introduces staleness; downstream systems might need to handle tombstone events differently.

#### Implementation
Steps
Event Scanning: Periodically scan the event lake for corrupt events.
Tombstone Generation: For each corrupt event, generate a corresponding tombstone entry.
Tombstone Propagation: Append the tombstones to the end of the event lake, ensuring they are visible to all downstream consumers.
Correction-Events
Core Idea

#### Pros
Ensures consistency by maintaining the original sequence and semantics; allows for transparent correction.

#### Cons
Requires careful handling to ensure that new events do not disrupt existing data lineage.

#### Implementation
Steps
Event Detection: Detect corrupt events through validation logic during event ingestion or periodic checks.
Correction Event Generation: For each detected corrupt event, generate a corresponding correction event.
Event Appending: Append the correction events to the end of the event lake in chronological order.

#### Cons
umer Handling: Downstream consumers need to handle both original and corrected events appropriately.
Versioned Rewrites
Core Idea

#### Pros
Completely replaces corrupted segments, ensuring full data integrity.

#### Cons
Resource-intensive; disrupts immutability guarantees for historical data.

#### Implementation
Steps
Segment Detection: Identify the corrupted segment(s) in the event lake.
Rewrite Generation: Generate a new version of the segment with corrected events.
Segment Replacement: Replace the old segment with the new version in the event lake.
Version Tracking: Maintain a version history to track changes and ensure immutability for older segments.

#### Closing Summary

Each approach has its trade-offs, but correction-events are often preferred because they maintain data lineage while ensuring consistency. Tombstones can be used as a temporary measure or in less critical systems, whereas versioned rewrites should be reserved for scenarios where complete data integrity is non-negotiable.

#### Common Mistakes

Failing to consider the impact on downstream consumers.
Overlooking the need for robust validation and correction mechanisms.
Neglecting performance implications of frequent compaction or rewriting operations.

**Comparison:**

```
Event ID	Data
123	valid_data
456	null
789	corrected_valid_data
```

**Code:**

```sql
-- Example SQL for tombstone generation
INSERT INTO event_lake (event_id, event_data) VALUES ('corrupt_event', NULL);

-- Example SQL for correction event generation
INSERT INTO event_lake (event_id, event_data) VALUES ('corrected_event', 'new_valid_data');
```

**Example input / output:**

```text
Input:
Event Lake (with tombstones)
123 - valid_data
456 - null

Output after correction-event generation:
123 - valid_data
456 - null
789 - corrected_valid_data
```

**Data:**

```json
{
  "events": [
    { "event_id": "123", "data": "valid_data" },
    { "event_id": "456", "data": null }, // corrupt event marked by tombstone
    { "event_id": "789", "data": "corrected_valid_data" } // correction event
  ]
}
```

**Flow:**

1. Detect Corrupt Events
2. Generate Tombstones
3. Validate and Detect Errors
4. Generate Correction Events
5. Identify Corrupted Segment
6. Generate New VersionedSegment
7. Append Tombstone to EventLake
8. Generate Correction Event andAppend
9. Replace Old Segment withNew Version

