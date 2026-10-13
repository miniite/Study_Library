---
type: "practice"
difficulty: "Basic"
---

# What is the default block size in HDFS and why does Hadoop use such large blocks compared to normal file systems?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ HDFS Data Integrity](<../../../../TOOLS/Hadoop/Hadoop_Ecosystem/HDFS_Data_Integrity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The default block size in Hadoop Distributed File System (HDFS) is 128 MB, although it can be configured differently. This large block size is a key design decision that optimizes performance and scalability in distributed environments. Let's dive into the reasoning behind this choice from first principles.

Firstly, why does Hadoop use such large blocks? The primary reason is to reduce the number of I/O operations required for data processing tasks. In a traditional file system, small files lead to frequent seeks and reads, which can become bottlenecks. By using larger block sizes, Hadoop minimizes the overhead associated with reading and writing data, leading to more efficient operations.

Secondly, large blocks help in parallel processing. When multiple mappers are processing different parts of a file simultaneously, having fewer but larger blocks means that each mapper has access to more data at once, thus improving overall throughput.

Lastly, the large block size helps in handling the overhead of network communication and disk I/O. Each time a block is read from HDFS, there's some overhead involved in setting up the connection and processing the block. Larger blocks mean fewer such operations for the same amount of data, making each operation more efficient.

In summary, using 128 MB as the default block size in HDFS strikes a balance between reducing I/O operations, enabling parallelism, and managing network and disk overhead effectively.

**Code:**

```python
# Example of setting block size in Hadoop configuration (hdfs-site.xml)
<property>
    <name>dfs.block.size</name>
    <value>134217728</value>  # 128 MB in bytes
</property>
```

