---
type: "practice"
difficulty: "Deep"
---

# A MapReduce job is failing with Java heap OutOfMemory in the mapper. Walk me through the container memory settings, JVM heap, and record buffer parameters you would inspect.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ YARN Troubleshooting](<../../../../TOOLS/Hadoop/Hadoop_Ecosystem/YARN_Troubleshooting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a Hadoop MapReduce job fails due to an OutOfMemoryError in the mapper, it's crucial to understand how container memory is allocated and managed within the YARN framework. This involves examining several key parameters related to container memory settings, JVM heap size, and record buffer configurations.

Firstly, let's clarify what we're dealing with: Hadoop MapReduce jobs run on containers that are allocated by the YARN resource manager. These containers have a fixed amount of memory (both for the JVM heap and non-heap spaces), which can be adjusted based on job requirements and cluster configuration. The record buffer size is another critical parameter related to how data is processed within the mapper.

#### Triaging the Question

The question is asking us to inspect specific container memory settings, including:

YARN Container Memory Settings: This includes both the allocated memory for the JVM heap (heap size) and non-heap spaces.
JVM Heap Size: The maximum amount of memory the Java Virtual Machine can use.
Record Buffer Size: The buffer used by the map task to store key-value pairs before they are passed to the reducer.

#### Assumptions
We assume that the job is configured with default or custom settings for these parameters.
We need to consider edge cases and potential misconfigurations that could lead to OutOfMemoryError.

#### First-Principles Solution

To resolve the issue, we will methodically inspect and adjust each of these parameters:

YARN Container Memory Settings:

By default, YARN containers are configured with a certain memory setting (e.g., 2GB). This can be checked via yarn-site.xml configuration files.
The key properties to look at include:
XML
<property>
    <name>yarn.nodemanager.resource.memory-mb</name>
    <value>2048</value>
</property>
We can increase this value if the job requires more memory. However, increasing it too much may lead to sub-optimization of resource utilization.

JVM Heap Size:

The JVM heap size is controlled by the mapreduce.map.java.opts and mapreduce.reduce.java.opts properties.
For example:
XML
<property>
    <name>mapreduce.map.java.opts</name>
    <value>-Xmx2048m</value>
</property>
Similarly, for the reducer:
XML
<property>
    <name>mapreduce.reduce.java.opts</name>
    <value>-Xmx1024m</value>
</property>
Adjusting these settings can help prevent OutOfMemoryError by ensuring the JVM has enough memory to process the data.

Record Buffer Size:

The record buffer size is controlled by the property mapreduce.task.io.sort.mb.
For example:
XML
<property>
    <name>mapreduce.map.sort.spill.percent</name>
    <value>0.85</value>
</property>
This parameter determines when spill to disk occurs and can be adjusted based on the expected size of intermediate data.

#### Closing Summary

In summary, we need to inspect and potentially adjust YARN container memory settings, JVM heap sizes for both map and reduce tasks, as well as record buffer parameters. Each of these adjustments will help in managing memory usage effectively within the MapReduce job, preventing OutOfMemoryError failures.

#### Common Mistakes/Follow-Ups
Not considering the balance between JVM heap and non-heap memory: Overly increasing the JVM heap might lead to excessive garbage collection pauses.
Ignoring edge cases where small records dominate over large ones: Ensuring that the record buffer size is appropriate for the expected data distribution is crucial.
Failing to monitor job performance post-adjustment: Regular monitoring and tuning based on actual job run-time behavior are essential.

**Code:**

```xml
<!-- Example of YARN container memory configuration -->
<property>
    <name>yarn.nodemanager.resource.memory-mb</name>
    <value>2048</value>
</property>

<!-- Example JVM heap size for map tasks -->
<property>
    <name>mapreduce.map.java.opts</name>
    <value>-Xmx2048m</value>
</property>

<!-- Record buffer spill threshold configuration -->
<property>
    <name>mapreduce.task.io.sort.spill.percent</name>
    <value>0.85</value>
</property>
```

