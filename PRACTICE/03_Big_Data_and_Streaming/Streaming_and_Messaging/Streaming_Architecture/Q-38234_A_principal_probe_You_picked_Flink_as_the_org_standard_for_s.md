---
type: "practice"
difficulty: "Expert"
---

# A principal probe: 'You picked Flink as the org standard for stream processing. In five years, what's the most likely reason a successor architect rips it out?' Give an honest answer about where your bet is most fragile.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Log-first vs Processor-first Platforms](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Log-first_vs_Processor-first_Platforms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

As an AIOps Architect/Principal, choosing Apache Flink as the standard stream processing framework was a well-considered decision, given its robustness and versatility. However, in five years, it's most likely that a successor architect might rip out Flink if it fails to address emerging scalability challenges or faces significant performance bottlenecks under new workloads. The fragility of our bet lies primarily in three areas: technology evolution, competitive landscape, and organizational needs.

Firstly, the technology ecosystem around stream processing is rapidly evolving. New frameworks like Apache Beam and KStreams continue to gain traction due to their ease of use and broader compatibility with big data ecosystems. If Flink struggles to keep up with these advancements or if it becomes too cumbersome for developers, a successor architect might opt for an alternative.

Secondly, the competitive landscape plays a crucial role. New entrants like Starcounter and even traditional database vendors like Oracle and Microsoft are increasingly focusing on real-time analytics solutions. These technologies may offer better performance, lower costs, or more seamless integration with existing systems, making Flink less attractive over time.

Lastly, organizational needs evolve, and the architecture must adapt accordingly. As our organization scales, it might require a solution that can handle more diverse data types or provide faster deployment cycles. If Flink cannot meet these demands without significant refactoring, it could become a bottleneck for innovation and agility within the team.

In summary, while Flink is robust today, its future fragility stems from its inability to keep pace with technological advancements, competitive pressures, and changing organizational requirements. A successor architect might reconsider its use if it becomes an anchor rather than a facilitator of progress.

**Comparison:**

```
Time Window	Locations
[1634567890, 1634567900)	A
[1634567900, 1634567910)	B,C
Example Flink Job Logic
Source: Read from a sensor data stream.
Filter: Keep only records where the temperature is above 20.
Group By & Aggregation: Group by location and calculate average temperature.
Sink: Output the results to another system (e.g., Kafka, database).
```

**Code:**

```python
# Example Flink Streaming Job in Python (simplified)
from pyflink.dataset import ExecutionEnvironment
from pyflink.table import TableConfig, DataTypes, BatchTableEnvironment

env = ExecutionEnvironment.get_execution_environment()
t_config = TableConfig()
t_env = BatchTableEnvironment.create(env, t_config)

source_table = t_env.from_path("sensor_data")
processed_table = source_table.filter("temperature > 20") \
                             .group_by(...).select(...)

result = processed_table.execute()
```

**Example input / output:**

```text
Input:
[
  {"timestamp": 1634567890, "location": "A", "temperature": 18},
  {"timestamp": 1634567900, "location": "B", "temperature": 21},
  {"timestamp": 1634567910, "location": "C", "temperature": 25}
]

Output:
[
  {"timestamp": 1634567900, "location": "B", "temperature": 21},
  {"timestamp": 1634567910, "location": "C", "temperature": 25}
]
```

**Data:**

```json
{
  "sensor_data": [
    {"timestamp": 1634567890, "location": "A", "temperature": 18},
    {"timestamp": 1634567900, "location": "B", "temperature": 21},
    {"timestamp": 1634567910, "location": "C", "temperature": 25}
  ]
}
```

**Flow:**

1. Sensor Data Source
2. Filter
3. Processed Data
4. Database
5. Visualization Tool

