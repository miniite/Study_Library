---
type: "practice"
difficulty: "Basic"
---

# What is data compression in Hadoop and why might you compress the output of a job?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ MapReduce Combiners](<../../../../TOOLS/Hadoop/Hadoop_Ecosystem/MapReduce_Combiners.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Data compression in Hadoop refers to the process of reducing the size of data stored or transferred, which can significantly improve storage efficiency and reduce network bandwidth usage during data processing tasks. At a basic level, compressing data before storing it in Hadoop clusters or transferring it between nodes can save disk space and enhance the overall performance of your Hadoop applications.

Hadoop jobs often generate large volumes of data as output, and without proper compression, this could lead to increased storage costs and longer processing times due to the higher I/O overhead. By compressing the output, you can reduce these issues, making your jobs more efficient.

To understand why data compression is beneficial in Hadoop, let’s break it down from first principles.

**Comparison:**

```
Compressed Data	Description
4570616c206f666673657420737472696e672068657265736b20696e2045415020636f6d707574656d73	Hexadecimal representation of the compressed string
```

**Code:**

```
Example of how data might be compressed before and after storage

import zlib

def compress_data(data): return zlib.compress(data.encode())

compressed_data = compress_data("This is a sample string to demonstrate compression.") print(f"Compressed Data: {compressed_data}")
```

**Example input / output:**

```

Input: "This is a sample string to demonstrate compression."

Output:

JSON
Copy
{
  "compressed": "4570616c206f666673657420737472696e672068657265736b20696e2045415020636f6d707574656d73"
}
```

**Data:**

```

Sample data:

JSON
Copy
{
  "input": "This is a sample string to demonstrate compression.",
  "compressed": "4570616c206f666673657420737472696e672068657265736b20696e2045415020636f6d707574656d73"
}
```

**Flow:**

1. Yes
2. No
3. Generate Data
4. Compress Data?
5. Compress and Store
6. Store Uncompressed Data
7. Reduce Storage Costs andBandwidth Usage

