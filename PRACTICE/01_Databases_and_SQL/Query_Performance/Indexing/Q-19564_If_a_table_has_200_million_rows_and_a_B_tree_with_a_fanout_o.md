---
type: "practice"
difficulty: "Deep"
---

# If a table has 200 million rows and a B+ tree with a fanout of 500, roughly how many levels deep is the index?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ B-tree and BRIN Indexes](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/B-tree_and_BRIN_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To determine the depth of a B+ tree index in a database table with 200 million rows and a fanout of 500, we need to understand the structure of a B+ tree and how it affects query performance. A B+ tree is designed to minimize disk I/O operations by keeping a balanced distribution of keys across its levels, which are referred to as levels or depths.

The key idea here is to calculate the number of levels required to store 200 million rows with a fanout of 500. The fanout represents the number of child pointers each node in the tree can have, and it directly impacts how many keys (and thus rows) are stored at each level.

The formula to estimate the height ( h ) of a B+ tree is: [ n = f^h - 1 ] where ( n ) is the number of leaf nodes (which in this case would be equal to the number of rows), and ( f ) is the fanout. Rearranging for ( h ): [ h = \log_f(n + 1) ]

Given:

Number of rows, ( n = 200,000,000 )
Fanout, ( f = 500 )

Plugging in the values: [ h = \log_{500}(200,000,000 + 1) ]

Let's compute this using logarithms.

Step-by-step Calculation

To calculate the logarithm base 500 of a large number like ( 200,000,001 ), we can use the change of base formula: [ h = \frac{\log(200,000,001)}{\log(500)} ]

Using common logarithms (base 10) or natural logarithms (base ( e )): [ h \approx \frac{\log(200,000,001)}{\log(500)} ] [ h \approx \frac{8.301}{2.699} ] [ h \approx 3.047 ]

Since the height of a tree must be an integer and we need to ensure all rows are stored, we round up: [ h = 4 ]

#### Closing Summary

The index depth for a B+ tree with 200 million rows and a fanout of 500 is approximately 4 levels deep. This means that the query will require at most 3 disk accesses to traverse from the root to find a specific row, which is quite efficient.

#### Common Mistakes/Follow-ups
Mistake: Not converting ( n ) correctly to account for the exact number of rows.

#### Follow-up
Discussing how different fanouts would affect index depth and performance. For example, a lower fanout might lead to deeper trees, which can impact query performance due to more disk accesses.

Would you like me to elaborate on any specific part or have any other questions?

**Code:**

```python
import math

# Given values
n = 200_000_000 + 1
f = 500

# Calculate the height of the B+ tree
h = math.log(n, f)
h_rounded = math.ceil(h)

h_rounded
```

**Example input / output:**

```text
4
```

