---
type: "practice"
difficulty: "Intermediate"
---

# How would you enrich a raw metric stream with static host-to-service mappings that change occasionally without redeploying the job?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Flink vs Spark Streaming](<../../../../TOOLS/Apache_Flink/Flink_Core/Flink_vs_Spark_Streaming.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the problem of enriching a raw metric stream with static host-to-service mappings in an Apache Flink application, we need to design a solution that can dynamically update the mappings without requiring a full redeployment of the job. This involves separating the logic for processing metrics from the logic for updating the mappings and ensuring efficient handling of these updates.

#### Triaging the Question

The core task is to enrich raw metric streams with static host-to-service mappings.
We need to handle frequent but occasional changes in mappings without stopping or redeploying the Flink job.
This requires a strategy that can dynamically update the mapping logic while keeping the main data processing pipeline intact.

Key Steps to Solve:

Separate Concerns: Keep the core metric processing logic separate from the logic for updating mappings.
Externalize Mappings: Store the host-to-service mappings in an external, easily updatable source like a database or a file that can be refreshed without stopping Flink jobs.
Periodic Refresh Mechanism: Implement a mechanism to periodically refresh these mappings within the running job.

#### First-Principles Solution

Define the Data Stream and Mappings:

Assume we have a raw metric stream coming from an IoT device or some monitoring system, with metrics like hostID and timestamp.
The static host-to-service mappings are stored in a database (e.g., MySQL) that can be queried at runtime.

Flink Job Setup:

Configure a Flink job to consume the raw metric stream.
Use a DataStream source to read from the metrics stream and store it in a DataStream<Metric> where each element is a tuple like (hostID, timestamp, value).

Mapping Externalization:

Store the host-to-service mappings in an external database or file that can be updated independently.
Define a method to periodically fetch these mappings using Flink's Table API or plain Java/Scala code.

Dynamic Mapping Update Mechanism:

Implement a side task or background thread that runs concurrently with the main data processing pipeline.
This side task updates a local cache of host-to-service mappings every 5-10 minutes, for example.
Use Flink's state management features to keep this local cache updated and queryable by the main processing logic.
No
Yes
DataStream source
Metric Stream
Is Mappings Updated?
Fetch New Mappings
Update Local Cache
Local Cache
Metric Processing Logic
Processed Metrics
Sink

Processing with Mapped Values:

Once the local cache is updated, the metric processing logic queries this cache to enrich each incoming metric.
For example, if a metric has hostID = 123, the logic looks up its service name in the cache and adds it as an additional field.

Handling Edge Cases:

Ensure that there is no data loss or processing delay during mapping updates.

#### Cons
ider implementing a fallback mechanism for stale mappings (e.g., using cached values until the next update).

#### Closing Summary
+ Common Mistakes/Follow-Ups:

The key to solving this problem lies in keeping the mapping logic separate from the main processing pipeline and ensuring efficient, non-disruptive updates.

#### Common mistakes
include not properly managing state or not handling concurrent access issues when updating mappings.

#### Follow-up
questions could involve discussing the performance implications of different update frequencies or exploring alternative solutions like using a more sophisticated caching mechanism.

**Comparison:**

```
host_id	service_name
123	web_server
456	db_instance
```

**Code:**

```java
// Example code to fetch and cache host-to-service mappings in Flink
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.table.api.bridge.java.StreamTableEnvironment;

public class MetricEnrichmentJob {
    public DataStream<MetricWithService> enrichMetrics(DataStream<Metric> metrics) throws Exception {
        // Assuming MySQL is used as the external database for mappings
        try (Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/mapping_db", "user", "password")) {
            String query = "SELECT * FROM host_service_mapping";
            Statement statement = conn.createStatement();
            ResultSet resultSet = statement.executeQuery(query);

            MapFunction<Metric, MetricWithService> enricher = new Enricher(resultSet);
            return metrics.map(enricher);
        }
    }

    static class Enricher implements MapFunction<Metric, MetricWithService> {
        private final HashMap<Integer, String> hostToServiceMap;

        public Enricher(ResultSet resultSet) throws SQLException {
            // Initialize the mapping cache from the ResultSet
            while (resultSet.next()) {
                int hostId = resultSet.getInt("host_id");
                String serviceName = resultSet.getString("service_name");
                hostToServiceMap.put(hostId, serviceName);
            }
        }

        @Override
        public MetricWithService map(Metric metric) throws Exception {
            // Enrich the metric with service name based on host ID
            int hostId = metric.getHostID();
            String serviceName = hostToServiceMap.getOrDefault(hostId, "unknown");
            return new MetricWithService(metric, serviceName);
        }
    }

    static class Metric {
        int hostID;
        long timestamp;
        double value;

        public int getHostID() { return hostID; }
    }

    static class MetricWithService extends Metric {
        String serviceName;

        public MetricWithService(Metric metric, String serviceName) {
            super.metric = metric;
            this.serviceName = serviceName;
        }
    }
}
```

**Example input / output:**

```text
Input:
- Raw metric stream with hostIDs like (123, 456)

Output:
- Enriched metrics with service names like ((123, web_server), (456, db_instance))
```

**Data:**

```json
{
  "host_service_mapping": [
    {"host_id": 123, "service_name": "web_server"},
    {"host_id": 456, "service_name": "db_instance"}
  ]
}
```

**Flow:**

1. No
2. Yes
3. DataStream source
4. Metric Stream
5. Is Mappings Updated?
6. Fetch New Mappings
7. Update Local Cache
8. Local Cache
9. Metric Processing Logic
10. Processed Metrics
11. Sink

