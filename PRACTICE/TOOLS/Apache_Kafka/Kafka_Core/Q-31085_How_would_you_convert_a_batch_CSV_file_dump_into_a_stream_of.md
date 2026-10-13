---
type: "practice"
difficulty: "Intermediate"
---

# How would you convert a batch CSV file dump into a stream of Kafka events, and what care would you take with the header row and types?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Producer Batching](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Producer_Batching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To convert a batch CSV file dump into a stream of Kafka events, we need to process each line from the CSV as it arrives in Kafka topics. This involves splitting the CSV content, properly handling headers, and ensuring type accuracy for the data. At an intermediate level, the focus will be on understanding the basics of Kafka producers and serialization; at a more advanced level, considerations like scalability, fault tolerance, and performance optimization will come into play.

**Code:**

```python
from kafka import KafkaProducer
import csv

class CSVToKafkaStream:
    def __init__(self, broker_list, topic_name):
        self.producer = KafkaProducer(bootstrap_servers=broker_list,
                                      value_serializer=lambda v: v.encode('utf-8'))

    def process_csv(self, file_path):
        with open(file_path, 'r') as file:
            csv_reader = csv.reader(file)
            header = next(csv_reader)  # Read and skip the header row
            for row in csv_reader:
                self.send_to_kafka(row, header)

    def send_to_kafka(self, row_data, headers):
        message_key = ','.join(headers).encode('utf-8')
        value = ','.join(row_data).encode('utf-8')
        self.producer.send(self.topic_name, key=message_key, value=value)
```

**Example input / output:**

```

Assume we have a CSV file data.csv with the following content:

CSV
Copy
id,name,age
1,Alice,30
2,Bob,25
3,Charlie,35

The Python script will produce messages like:

Key: b'id,name,age', Value: b'1,Alice,30'
Key: b'id,name,age', Value: b'2,Bob,25'
Key: b'id,name,age', Value: b'3,Charlie,35'
```

**Flow:**

1. Yes
2. No
3. Read CSV File
4. Skip Header Row
5. Is Line a Data Row?
6. Send to Kafka
7. Sent to Kafka Topic

