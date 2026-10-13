---
type: "practice"
difficulty: "Deep"
---

# Design a claim-check pattern for a topic carrying 20 MB image payloads. What goes in Kafka, what goes in object storage, and how do you handle cleanup?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Claim Check Pattern](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Claim_Check_Pattern.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

The claim-check pattern involves storing metadata about large files (like images) in a messaging system like Kafka while keeping the actual file content in an object storage system such as S3. The key is to design this with careful consideration of performance, durability, and cost efficiency.

#### Triaging the Question

The question is asking for a detailed design of a claim-check pattern where image payloads (up to 20 MB each) are stored and processed using Kafka while ensuring efficient storage and retrieval from object storage. The interviewer wants to understand how we handle both data flow and cleanup processes, including potential optimizations and trade-offs.

Step-by-Step Solution

Topic Design in Kafka:

Message Structure: Each message will contain a metadata payload (including image details like ID, timestamp, and other relevant information) and a reference to the actual image stored in object storage.
Partitioning Strategy: To ensure high throughput and prevent hotspots, partition the topic based on unique identifiers of the images. This could be done using a hash function over the image ID.

Object Storage:

S3 Buckets: Store each 20 MB image in an S3 bucket with proper naming conventions to facilitate easy retrieval.
Versioning and Tagging: Enable versioning for durability and tagging for metadata management, such as lifecycle policies or tags indicating the object's status (e.g., active, archived).

Claim-Check Pattern Implementation:

Producer Flow:
When an image is uploaded:
Compute a unique ID.
Upload the image to S3 with appropriate metadata tagging.
Publish a Kafka message containing the metadata and the S3 object URL or key.

#### Cons
umer Flow:
Upon receiving a message, fetch the image from its S3 location for processing (e.g., resizing, watermarking).

Cleanup Mechanism:

Retrying Logic: Implement retry logic in producers/consumers to handle temporary failures without immediate deletion of images.
Cleanup Strategy:
Manual Cleanup: Provide a mechanism for manual cleanup if required by business rules (e.g., after some period, delete old versions).
Automated Cleanup: Use S3 lifecycle policies or scheduled tasks to automatically move or delete outdated images based on retention policies.
Garbage Collection: Consider periodic garbage collection of metadata entries in Kafka that are no longer needed.

#### Closing Summary
and Common Mistakes

In summary, the key elements of this design include efficient use of Kafka for metadata storage, leveraging S3 for image content, and implementing robust cleanup mechanisms. A common mistake is underestimating the importance of proper partitioning and lifecycle management to maintain system performance and cost-effectiveness.

#### Common Mistakes/Follow-Ups
Inadequate Partitioning: Poorly designed partitions can lead to hotspots or suboptimal performance.
Ignoring Cost Implications: Not accounting for S3 storage costs, especially when dealing with high volumes of images over time.
Overlooked Metadata Management: Failing to properly manage metadata in Kafka, leading to increased data complexity and maintenance overhead.

**Comparison:**

```
Partition	Key Function
P0	Hash(image_id)
P1	Hash(image_id + timestamp)
P2	Hash(image_id % 3)
Cleanup Strategy	Action
Manual Cleanup	Admin removes entries from Kafka metadata
Automated Cleanup	S3 lifecycle policy deletes old versions
```

**Code:**

```python
# Example code snippet for producer logic
import boto3

def upload_image_to_s3(file_path, s3_bucket_name):
    s3 = boto3.client('s3')
    image_key = f"images/{uuid.uuid4()}/{file_path.split('/')[-1]}"
    s3.upload_file(file_path, s3_bucket_name, image_key)
    return image_key

def produce_kafka_message(image_data, s3_image_url):
    # Kafka message structure
    metadata = {
        "image_id": str(uuid.uuid4()),
        "timestamp": datetime.now().isoformat(),
        "s3_url": s3_image_url,
        "other_metadata": {...}
    }

    producer.send('image_topic', value=metadata)

# Example code snippet for consumer logic
def consume_kafka_message(message):
    s3_image_url = message['s3_url']
    # Process the image from S3
```

**Example input / output:**

```text
Input:
- User uploads an image to S3 with key "images/uuid/image.png"

Output:
- Kafka message published with metadata {"image_id": "unique-id-123", ...}

Process:
- Consumer receives the message, fetches the image from S3 and processes it.
```

**Data:**

```json
{
  "image_metadata": {
    "image_id": "unique-id-123",
    "timestamp": "2023-10-01T14:56:37.123Z",
    "s3_url": "https://my-bucket.s3.amazonaws.com/images/uuid/image.png"
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. Producer Uploads Image
5. Is Image Size > 20MB?
6. Upload to S3
7. Generate Unique ID
8. Produce Kafka Message withMetadata
9. Send to Kafka Topic
10. Consumer Receives KafkaMessage
11. Fetch from S3?
12. I
13. Process Image and SaveResults
14. Fetch S3 Image URL fromMetadata
15. Download Image from S3
16. Process Image and SaveResults
17. Cleanup Strategy
18. Manual Cleanup: AdminRemoves Entries
19. Automated Cleanup: LifecyclePolicies Delete Old Versions

