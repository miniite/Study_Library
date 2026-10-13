---
type: "practice"
difficulty: "Expert"
---

# Design an idempotent payment-write path on DynamoDB so a client retry never double-charges. What key and condition expression do you use, and how does TTL interact with your idempotency records?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ DynamoDB Modeling and Capacity](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/DynamoDB_Modeling_and_Capacity/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an idempotent payment-write path in DynamoDB that ensures a client retry never double-charges, we need to leverage unique identifiers (keys) and condition expressions to ensure atomicity. Specifically, we will use a combination of the PutItem operation with a proper key structure and a condition expression based on the existence of an idempotency record. Additionally, Time-to-Live (TTL) can be utilized to automatically clean up expired records.

The key idea is to store each payment attempt as a unique record in DynamoDB with a timestamp or sequence number as part of the key. The condition expression ensures that if the same payment attempt is retried, it will fail because the item already exists. TTL helps manage stale data by automatically expiring old records after a set period.

**Comparison:**

```
Key	Value
paymentId	12345
sequenceNumber	1
Condition Expression	Explanation
attribute_not_exists(#pk)	Ensures the item does not exist in DynamoDB, preventing double-charges on retries.
```

**Code:**

```python
import boto3

# Initialize DynamoDB client
dynamodb = boto3.resource('dynamodb')
table_name = 'PaymentTable'
table = dynamodb.Table(table_name)

def create_payment_item(payment_id, amount):
    # Define the unique key using a combination of payment ID and sequence number
    key = {
        "paymentId": payment_id,
        "sequenceNumber": 1  # Incrementing this would allow for multiple attempts per id if needed
    }

    # Condition expression to ensure idempotency
    condition_expression = "attribute_not_exists(#pk)"  # No item with the same primary key should exist

    # PutItem operation with condition check
    table.put_item(
        Item=key,
        ConditionExpression=condition_expression
    )
```

**Example input / output:**

```text
Input:
- `payment_id = '12345'`
- `amount = 9.99`

Output:
- Item successfully written to DynamoDB if it doesn't already exist

Input:
- Same `payment_id` and `sequenceNumber` as above, attempting a retry

Output:
- Failure due to the condition expression not being satisfied (item already exists)
```

**Data:**

```json
{
    "paymentId": "12345",
    "sequenceNumber": 1
}
```

**Flow:**

1. Yes
2. No
3. Client Initiates Payment
4. Puts Item with ConditionExpression
5. Check Item Exists?
6. Return Error - AlreadyProcessed
7. Write Item to DynamoDB,Increment Sequence Number ifRetrying
8. Process Payment and UpdateState

