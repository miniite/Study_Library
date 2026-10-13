---
type: "practice"
difficulty: "Deep"
---

# How does the default HDFS block placement policy use rack awareness to place the three replicas?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ HDFS Data Integrity](<../../../../TOOLS/Hadoop/Hadoop_Ecosystem/HDFS_Data_Integrity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary: In Hadoop, the default block placement policy uses rack awareness to ensure high availability and fault tolerance by distributing data across different racks in a cluster. This strategy helps mitigate network partitioning issues and ensures that at least one replica of any block is always available.

#### Triaging the Question

The question is asking about how Hadoop's default block placement policy utilizes rack awareness to place three replicas of each file block. The key points being probed are:

Understanding Rack Awareness: How does rack awareness work in a HDFS cluster?
Block Placement Policy: What factors influence where a block replica is placed?
Replica Distribution Strategy: How do the three replicas get distributed across different racks?

#### First-Principles Solution
Understanding Rack Awareness

Rack awareness in Hadoop ensures that data is spread out across multiple racks to improve fault tolerance and performance. A rack is typically defined as a set of machines connected over a high-speed network, often with lower latency compared to the rest of the cluster.

Block Placement Policy

The default block placement policy for HDFS involves placing three replicas of each file block. The primary replica (Replica 1) is always placed on the same datanode that stores the data being written. The secondary and tertiary replicas are then distributed across different racks to ensure redundancy and availability.

Replica Distribution Strategy

Here’s a step-by-step breakdown:

Primary Replica Placement:
The primary replica of each block (Replica 1) is always placed on the datanode where the write operation originated.
Secondary Replica Placement:
The secondary replica (Replica 2) is typically placed on a different node within the same rack as the primary replica to reduce network latency and improve performance.
Tertiary Replica Placement:
The tertiary replica (Replica 3) is then placed on a datanode in a completely different rack, ensuring that even if one rack fails, the data remains available.

#### Closing Summary

In summary, Hadoop's default block placement policy uses rack awareness to place three replicas of each file block by first placing the primary replica on the write node, then placing the secondary replica on another node within the same rack, and finally placing the tertiary replica in a different rack. This strategy ensures high availability and fault tolerance while minimizing network latency.

#### Common Mistakes/Follow-Ups
Incorrect Rack Placement: Placing all three replicas within the same rack can lead to issues if that rack fails.
Ignoring Network Latency: Failing to consider network topology in replica placement can affect performance.
Manual Override: Always be aware that manual overrides or custom placement policies might interfere with this default behavior.

**Comparison:**

```
Replica	Node	Rack
1	Node-0	Rack-A
2	Node-1	Rack-B
3	Node-2	Rack-C
```

**Code:**

```java
// No code needed for explanation, but here's a simple Hadoop job submission example:
Job job = Job.getInstance(conf, "example job");
job.setJarByClass(ExampleJob.class);
Path inputPath = new Path("hdfs://namenode:9000/input");
Path outputPath = new Path("hdfs://namenode:9000/output");
FileInputFormat.addInputPath(job, inputPath);
FileOutputFormat.setOutputPath(job, outputPath);
job.setMapperClass(Map.class);
job.setReducerClass(Reduce.class);
job.setOutputKeyClass(Text.class);
job.setOutputValueClass(IntWritable.class);
boolean success = job.waitForCompletion(true);
```

**Example input / output:**

```text
Input:
- file1.txt on Node-0 in Rack-A

Output (Replicas):
1. Replica 1 - Node-0, Rack-A
2. Replica 2 - Node-1, Rack-B
3. Replica 3 - Node-2, Rack-C
```

**Data:**

```json
{
  "input": {
    "path": "/user/hadoop/input",
    "files": ["file1.txt", "file2.txt"]
  },
  "output": {
    "replicas": [
      { "rack": "Rack-A", "node": "Node-0" },
      { "rack": "Rack-B", "node": "Node-1" },
      { "rack": "Rack-C", "node": "Node-2" }
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. Start
7. Is Primary Replica?
8. Place on Write Node
9. Place Secondary Replica?
10. On Different Node in SameRack
11. Place Tertiary Replica?
12. On Different Rack
13. End

