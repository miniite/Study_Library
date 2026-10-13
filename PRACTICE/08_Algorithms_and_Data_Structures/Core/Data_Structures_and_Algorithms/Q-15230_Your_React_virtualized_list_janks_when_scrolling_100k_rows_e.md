---
type: "practice"
difficulty: "Intermediate"
---

# Your React virtualized list janks when scrolling 100k rows; explain what data structure you'd use to map scroll offset to row index and how it keeps lookups sub-linear.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Spatial Indexes](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Spatial_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a large number of rows in a React virtualized list, such as 100k rows, performance can degrade if we use inefficient data structures to map scroll offsets to row indices. The key challenge is to maintain sub-linear time complexity for lookups during scrolling operations. A suitable data structure that addresses this need is a jump table combined with a binary search tree (BST) or a combination of a hash map and binary search.

In summary, the approach involves creating a jump table to quickly narrow down the range of rows and then using a binary search within that range for precise index lookup. This ensures efficient scrolling performance by reducing the overall time complexity from linear to sub-linear.

#### Triage

The question is asking about an effective data structure strategy to handle large-scale row mapping in virtualized lists, specifically focusing on maintaining performance during scroll operations. The key points are:

Efficient Row Index Lookup: Need a fast way to map scroll offsets to row indices.
Sub-linear Complexity: Ensuring that the time complexity for lookups remains efficient (O(log n) or better).
Performance Under High Load: Handling 100k rows without significant jank.

#### First Principles Solution

Jump Table Initialization:

Create a jump table where each entry represents a segment of rows.
Each entry in the jump table contains the starting and ending row indices for that segment, along with the index of the first row in that segment.
JAVASCRIPT

#### cons
t jumpTable = [];

Building Jump Table:

Populate the jump table by dividing the total number of rows into manageable segments.
Each entry in jumpTable will be an object like { start: 0, end: 100, index: 0 }.
JAVASCRIPT
for (let i = 0; i < numRows; i += segmentSize) {

#### cons
t startRow = i;

#### cons
t endRow = Math.min(i + segmentSize - 1, numRows - 1);
  jumpTable.push({ start: startRow, end: endRow, index: startRow });
}

Scroll Offset to Row Index Mapping:

Given a scroll offset, use binary search on the jump table to find the appropriate segment.
Once the correct segment is identified, perform a binary search within that segment for exact row mapping.
JAVASCRIPT
function getRowIndex(offset) {

#### cons
t { start, end } = jumpTable[Math.floor(offset / (numRows / jumpTable.length))];
  return binarySearch(start, end, offset);
}

function binarySearch(start, end, offset) {
  while (start < end) {

#### cons
t mid = Math.floor((start + end) / 2);
    if ((mid - start) * (offset - jumpTable[start].index) > 0) {
      end = mid;
    } else {
      start = mid + 1;
    }
  }
  return start;
}

#### Closing Summary
and Common Mistakes
Common Mistake: Using a linear search approach for every scroll event. This leads to poor performance as the time complexity becomes O(n) with large datasets.
Alternative Approach: While jump tables are effective, another option is to use a combination of a hash map and binary search. The hash map stores row indices directly but may require frequent updates.
Code Blocks
JAVASCRIPT
// Example Jump Table Initialization

#### cons
t numRows = 100000;

#### cons
t segmentSize = 500; // Adjust based on performance needs

function buildJumpTable(numRows, segmentSize) {

#### cons
t jumpTable = [];
  for (let i = 0; i < numRows; i += segmentSize) {

#### cons
t startRow = i;

#### cons
t endRow = Math.min(i + segmentSize - 1, numRows - 1);
    jumpTable.push({ start: startRow, end: endRow, index: startRow });
  }
  return jumpTable;
}

#### cons
t jumpTable = buildJumpTable(numRows, segmentSize);

// Function to get row index from scroll offset
function getRowIndex(offset) {

#### cons
t { start, end } = jumpTable[Math.floor(offset / (numRows / jumpTable.length))];
  return binarySearch(start, end, offset);
}

function binarySearch(start, end, offset) {
  while (start < end) {

#### cons
t mid = Math.floor((start + end) / 2);
    if ((mid - start) * (offset - jumpTable[start].index) > 0) {
      end = mid;
    } else {
      start = mid + 1;
    }
  }
  return start;
}

