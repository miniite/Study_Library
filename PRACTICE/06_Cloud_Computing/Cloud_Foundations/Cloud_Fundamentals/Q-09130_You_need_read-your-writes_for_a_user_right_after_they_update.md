---
type: "practice"
difficulty: "Deep"
---

# You need read-your-writes for a user right after they update their profile, but you are serving reads from a lagging replica. How would you offer that guarantee without sending every read to the primary?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Read Your Writes with Replicas](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Read_Your_Writes_with_Replicas.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure "read-your-writes" (RYW) while serving reads from a lagging replica, we need to implement a mechanism that allows users to immediately see their updated profile information without all read requests hitting the primary database. This involves leveraging cloud-native features such as eventual consistency, caching, and event-driven architectures.

This solution requires a deep understanding of how databases operate in a distributed environment, especially with replicas and write propagation delays. The core idea is to use an event-driven architecture where writes are propagated to the replica asynchronously, while reads can still be served quickly from the replica. This way, users experience immediate updates without impacting the primary database's performance.

**Comparison:**

```
Event	Description
Update	User updates their profile
SNS Publish	Notification of update
```

**Code:**

```python
# Example of using AWS SNS and SQS for asynchronous write propagation
import boto3

def update_profile(user_id, new_info):
    # Update the user profile in the database
    client = boto3.client('dynamodb')
    response = client.update_item(
        TableName='Users',
        Key={'UserId': {'S': user_id}},
        UpdateExpression="set #info = :newInfo",
        ExpressionAttributeNames={'#info': 'Profile'},
        ExpressionAttributeValues={':newInfo': {'M': new_info}}
    )

    # Publish an event to SNS
    sns_client = boto3.client('sns')
    topic_arn = "arn:aws:sns:us-east-1:<account-id>:user-update-topic"
    message = {
        'UserId': user_id,
        'NewInfo': new_info
    }
    response = sns_client.publish(
        TopicArn=topic_arn,
        Message=str(message)
    )
```

**Example input / output:**

```

Input: User updates their profile with new name and email.

PYTHON
Copy
update_profile('12345', {'Name': 'Jane Doe', 'Email': 'janedoe@example.com'})

Output:

DynamoDB updated with the new information.
SNS message published: {"UserId": "12345", "NewInfo": {"Name": "Jane Doe", "Email": "janedoe@example.com"}}.
```

**Data:**

```json
{
  "User": {
    "UserId": "12345",
    "Profile": {
      "Name": "John Doe",
      "Email": "johndoe@example.com"
    }
  }
}
```

**Flow:**

1. User Updates Profile
2. Write to DynamoDB
3. SNS Publish
4. Replica Receives Event andApplies Changes
5. Read from Replica (ImmediateUpdate)

