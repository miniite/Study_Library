---
type: "practice"
difficulty: "Deep"
---

# How would you compute a streaming distinct-user count per campaign using approximate sketches like HyperLogLog inside a Flink aggregate?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Multi-granularity Windows](<../../../../TOOLS/Apache_Flink/Flink_Core/Multi-granularity_Windows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compute a streaming distinct-user count per campaign using approximate sketches like HyperLogLog within Apache Flink, we need to leverage Flink's powerful state and aggregation capabilities along with HyperLogLog's space-efficient cardinality estimation algorithm. This approach ensures low memory usage while providing near-accurate counts of unique users across campaigns in real-time.

**Code:**

```java
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.table.api.bridge.java.StreamTableEnvironment;
import org.apache.flink.table.api.Table;
import org.apache.flink.table.descriptors.Kafka;
import org.apache.flink.table.descriptors.RocksDBStateBackend;
import com.google.common.collect.ImmutableList;

public class DistinctUserCountAggregation {

    public static void main(String[] args) throws Exception {
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        final StreamTableEnvironment tableEnv = StreamTableEnvironment.create(env);

        // Setting up Kafka as the source
        tableEnv.connect(new Kafka())
                .version("universal")
                .topic("user-activity-stream")
                .property("bootstrap.servers", "localhost:9092")
                .deserializerColumnNames(ImmutableList.of("campaignId", "userId"))
                .inference("rowtime('TIMESTAMP mills'),campaignId, userId")
                .withFormat(new Kafka())
                .withSchema(
                        "campaignId STRING, userId STRING"
                )
                .registerTableSource("userActivityStream");

        // Creating a table for the source data
        Table userActivity = tableEnv.from("userActivityStream");

        // Applying HyperLogLog aggregation to count distinct users per campaign
        Table distinctUserCountPerCampaign = userActivity
                .groupBy("campaignId")
                .select("campaignId, COUNT_HLL(userId) as distinctUserCount");

        // Registering the result back into a table for further processing or sink
        tableEnv.createTemporaryView("distinctUserCountPerCampaign", distinctUserCountPerCampaign);

        // Executing the job
        env.execute("Streaming Distinct User Count per Campaign");
    }
}
JAVA
Copy

---
```

**Example input / output:**

```

No specific input-output examples are needed here, as the code itself serves as a clear illustration of how to set up the stream processing pipeline. However, an example run might look like:

Input: A continuous stream of user activity events with fields campaignId and userId.
Output: An update in the distinctUserCountPerCampaign view showing the approximate number of distinct users for each campaign.

Example input event:

CODE
Copy
{"campaignId": "C001", "userId": "U0001"}
{"campaignId": "C002", "userId": "U0001"}
{"campaignId": "C002", "userId": "U0002"}

Output from distinctUserCountPerCampaign view:

CODE
Copy
campaignId | distinctUserCount
------------|-----------------
C001       | 1
C002       | 2
```

**Flow:**

1. User Activity Stream
2. CampaignId, UserId
3. Reduce State per Campaign
4. HyperLogLog Sketch
5. Distinct User Count perCampaign

