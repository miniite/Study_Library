---
type: "practice"
difficulty: "Deep"
---

# Why does serializer choice matter for Flink state, and what problems arise if you naively use Kryo for evolving telemetry POJOs?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Keyed vs Operator State](<../../../../TOOLS/Apache_Flink/Flink_Core/Keyed_vs_Operator_State.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand why serializer choice matters in Apache Flink's state management, we need to first appreciate the role of serializers in data serialization and deserialization during stream processing tasks. In Flink, the state is a critical component that holds temporary or transient data used by operators for stateful computations. Serializer choices significantly impact performance, memory usage, and the ability to handle evolving schemas, especially when dealing with telemetry data.

#### Triage

The question asks us to delve into why the serializer choice is crucial in Flink's state management, particularly focusing on the problems that arise from using Kryo for evolving telemetry POJOs. The key areas we will explore are:

Role of Serializers: How they enable efficient serialization and deserialization.
Evolving Schemas: Challenges when dealing with changing data structures.
Kryo vs Other Serializers: Why Kryo might not be the best choice for evolving telemetry POJOs.

#### First Principles Solution
Serializer Role:
In Flink, serializers are responsible for converting objects to bytes and vice versa, which is essential for reading/writing state from/to persistent storage or network communication.
Evolving Schemas:
Telemetry data often evolves over time, meaning the structure of POJOs (Plain Old Java Objects) that represent this data can change. This requires the serializer to handle schema evolution seamlessly without breaking.
Kryo Challenges:
Kryo is a fast and efficient binary serialization framework but has limitations when dealing with evolving schemas. If the structure of the telemetry POJO changes, Kryo will fail because it does not support incremental updates or adding new fields.
Detailed Explanation
Serializer Role in Flink State Management

In Flink, each operator’s state is managed by a StateTtlSerDe (for time-to-live) and a custom serializer. The serializer ensures that the data can be read back correctly after being serialized to bytes. Flink supports multiple serializers out of the box, including Kryo, JSON, and Avro.

Evolving Schemas

When dealing with telemetry data, it is common for new metrics or dimensions to be added over time. For example, a system might start collecting temperature readings but later add humidity readings as well. The serializer must handle such changes without breaking the application.

Kryo works by mapping each class and its fields into a byte stream. Once serialized, Kryo assumes that the structure of the data remains unchanged for subsequent reads. If an existing field is removed or new fields are added, Kryo will throw exceptions because it cannot map the incoming byte stream to the expected class structure.

Alternative Serializers
Avro: Supports schema evolution by allowing modifications to the schema over time. New fields can be added and existing ones removed without breaking deserialization.
JSON: Flexible but slower than binary serializers like Kryo due to overhead in parsing and generating JSON strings.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, the choice of serializer is crucial for handling evolving telemetry data in Flink. While Kryo offers high performance, it struggles with schema evolution. Using a more flexible serializer like Avro or adopting a structured format like JSON can better handle changes without breaking the application.

#### Common Mistakes

Overlooking Schema Evolution: Assuming that the structure of the data will not change over time.
Choosing Kryo for Evolving Data: Not recognizing its limitations with schema evolution.

#### Follow-Ups
How would you implement a custom serializer in Flink to handle evolving schemas?
What are the trade-offs between using Kryo and other serializers like Avro or JSON?

**Comparison:**

```
Field	Type	Initial Presence	New Schema
field1	int	Present	Not Added
field2	boolean	Present	Removed
```

**Code:**

```java
// Example of a simple custom serializer for handling evolving POJOs
public class CustomSerializer implements StateSerializer<MyPOJO> {
    @Override
    public void serialize(MyPOJO value, DataOutputView target) throws IOException {
        // Serialize fields in a way that supports schema evolution
        if (value.hasField1()) {
            target.writeInt(value.getField1());
        }
        if (value.hasField2()) {
            target.writeBoolean(value.getField2());
        }
    }

    @Override
    public MyPOJO deserialize(DataInputView source) throws IOException, SerializationException {
        // Deserialize fields based on the schema at runtime
        boolean hasField1 = source.readBoolean();
        int field1 = 0;
        if (hasField1) {
            field1 = source.readInt();
        }
        boolean hasField2 = source.readBoolean();
        boolean field2 = false;
        if (hasField2) {
            field2 = source.readBoolean();
        }

        return new MyPOJO(field1, field2);
    }

    @Override
    public TypeInformation<MyPOJO> serializedType() {
        return Types.OBJECT(MyPOJO.class);
    }
}
```

**Example input / output:**

```text
- **Input**: A telemetry POJO with `field1` and `field2`.
- **Output**: Serialized data with the correct field values.
- **Deserialization**: Correctly deserializes to a new POJO instance even if the schema has changed.
```

**Data:**

```json
{
  "field1": 37,
  "field2": true
}
```

**Flow:**

1. field1 present
2. field2 present
3. New Schema
4. Removed from NewSchema
5. Read field presence
6. field1 present
7. field1 absent
8. Read field2 presence
9. field2 present
10. field2 removed
11. Serialize MyPOJO
12. Field Presence?
13. Write field1
14. Write field2
15. Serialize End
16. Serialize End
17. Ignore field1
18. Do not write field2
19. Deserialize MyPOJO
20. Check for field1
21. Read and set field1
22. Skip reading field1
23. Check for field2
24. Read and set field2
25. Do not attempt to read field2
26. Field1 Handling
27. Field2 Handling

