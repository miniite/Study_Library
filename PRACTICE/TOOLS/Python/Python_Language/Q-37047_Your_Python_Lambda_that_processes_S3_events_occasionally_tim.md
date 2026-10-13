---
type: "practice"
difficulty: "Deep"
---

# Your Python Lambda that processes S3 events occasionally times out and the event gets retried, causing duplicate processing. How do you make the handler idempotent using an S3 object key or a DynamoDB dedup marker?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Idempotent Event Handlers](<../../../../TOOLS/Python/Python_Language/Idempotent_Event_Handlers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of duplicate processing in a Python Lambda that processes S3 events, we need to ensure our handler is idempotent. This means that even if an event is processed multiple times due to timeouts and retries, the outcome should be the same. One effective approach is to use either the S3 object key or DynamoDB for deduplication markers.

First, let’s clarify what the question is probing:

The Lambda function is designed to process events from an S3 bucket.
It occasionally times out during execution, leading to retries that can cause duplicate processing of the same event.
We need a strategy to handle these retries idempotently without altering the expected outcome.

#### Triage
Understanding the Problem: The Lambda function needs to ensure that events are processed only once even if it is triggered multiple times due to timeouts and retries.
Identifying the Key Components:
S3 Object Key: This can be used as a unique identifier for each object in the bucket.
DynamoDB Dedup Marker: We could use a separate DynamoDB table to track processed events.

#### First-Principles Solution
Using an S3 Object Key: One straightforward approach is to include the S3 object key within the event payload and check if it has already been processed.
DynamoDB Dedup Marker: A more robust solution involves using a DynamoDB table to track which events have already been processed.
Using S3 Object Key
Modify the Lambda function to include the S3 object key in its logic.
Check if the object key exists in the database or some other storage mechanism before processing the event.
If it does not exist, process the event and update the database with the object key.

Here is a sample implementation using the S3 object key:

PYTHON
import json
import boto3

# Initialize DynamoDB client
dynamodb = boto3.resource('dynamodb')
table_name = 'ProcessedEvents'
table = dynamodb.Table(table_name)

def lambda_handler(event, context):
    # Extracting the S3 event data from the event payload
    for record in event['Records']:
        s3_event = json.loads(record['s3']['object']['key'])

        # Check if the object key has already been processed
        response = table.get_item(Key={'ObjectKey': s3_event['key']})
        if 'Item' not in response:
            # Process the S3 event
            process_s3_event(s3_event)

            # Update DynamoDB to mark it as processed
            response = table.put_item(Item={'ObjectKey': s3_event['key'], 'Processed': True})

def process_s3_event(event):
    print(f"Processing event: {event}")
    # Your processing logic here

Using DynamoDB Dedup Marker
Use a separate DynamoDB table to store the keys of processed S3 objects.
When an event triggers the Lambda, check if the object key already exists in the DynamoDB table.

Here is a sample implementation using DynamoDB:

PYTHON
import json
import boto3

# Initialize DynamoDB client and S3 client
dynamodb = boto3.resource('dynamodb')
table_name = 'ProcessedEvents'
s3_client = boto3.client('s3')

def lambda_handler(event, context):
    # Extracting the S3 event data from the event payload
    for record in event['Records']:
        s3_event = json.loads(record['s3']['object']['key'])

        # Check if the object key has already been processed using DynamoDB
        response = table.get_item(Key={'ObjectKey': s3_event['key']})
        if 'Item' not in response:
            # Process the S3 event
            process_s3_event(s3_event)

            # Update DynamoDB to mark it as processed
            response = table.put_item(Item={'ObjectKey': s3_event['key'], 'Processed': True})

def process_s3_event(event):
    print(f"Processing event: {event}")
    # Your processing logic here

#### Closing Summary
+ Common Mistakes/Follow-Ups

#### Common Mistakes

Failing to check for the existence of processed events.
Not updating DynamoDB or another storage mechanism after processing an event.
Ignoring edge cases where objects are deleted and recreated.

#### Follow-Ups

Discuss how to handle different types of S3 operations (e.g., uploads, deletes).
Explore more advanced strategies like using batch writes for efficiency in DynamoDB.

#### Cons
ider the trade-offs between simplicity and robustness based on the scale of your application.
Self-Check

Correctness + completeness confirmed.

**Comparison:**

| S3 Object Key | Processed |
|--------------|----------|
| file.txt     | true     |

**Code:**

```python
import json
import boto3

# Initialize DynamoDB client
dynamodb = boto3.resource('dynamodb')
table_name = 'ProcessedEvents'
table = dynamodb.Table(table_name)

def lambda_handler(event, context):
    # Extracting the S3 event data from the event payload
    for record in event['Records']:
        s3_event = json.loads(record['s3']['object']['key'])

        # Check if the object key has already been processed
        response = table.get_item(Key={'ObjectKey': s3_event['key']})
        if 'Item' not in response:
            # Process the S3 event
            process_s3_event(s3_event)

            # Update DynamoDB to mark it as processed
            response = table.put_item(Item={'ObjectKey': s3_event['key'], 'Processed': True})

def process_s3_event(event):
    print(f"Processing event: {event}")
    # Your processing logic here
```

**Example input / output:**

```

Input:

JSON
Copy
{
  "Records": [
    {
      "s3": {
        "bucket": {
          "name": "example-bucket"
        },
        "object": {
          "key": "file.txt"
        }
      }
    }
  ]
}

Output:

The process_s3_event function is called to process the event.
If the object key "file.txt" does not exist in DynamoDB, it processes the file and adds a record for "file.txt".

Dry Run:

JSON
Copy
{
  "Records": [
    {
      "s3": {
        "bucket": {
          "name": "example-bucket"
        },
        "object": {
          "key": "file.txt"
        }
      }
    }
  ]
}
Lambda processes the event and updates DynamoDB with the key.

Response:

JSON
Copy
{
  "StatusCode": 200,
  "Message": "File processed successfully."
}
```

**Data:**

```json
{
  "Records": [
    {
      "s3": {
        "bucket": {
          "name": "example-bucket"
        },
        "object": {
          "key": "file.txt"
        }
      }
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Event Triggers Lambda
4. Check if object key exists inDynamoDB
5. Process the event
6. Update DynamoDB withprocessed object key
7. Retry after timeout

