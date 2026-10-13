---
type: "practice"
difficulty: "Basic"
---

# What is the time complexity of accessing an element in an array by its index?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Set Operation Complexity](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Set_Operation_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage
— what is really being asked

The question is: "What is the time complexity of accessing an element in an array by its index?" The one-line answer the interviewer wants is O(1) — constant time. But at a basic-concept level, an interviewer asking this is not just checking whether you memorized "O(1)". They are probing whether you understand why it is O(1) — i.e., whether you understand how an array is laid out in memory and how the hardware turns an index into a memory address. A strong fresher answer states the result and explains the mechanism from first principles. Let me do exactly that.

#### Assumptions
/ clarifications: I'll assume a classic contiguous, fixed-size-element array (like a C array, a Java int[], or the backing store of a Python list / JS array), accessed as arr[i] for a valid index i. I am talking about random access by index, not searching for a value.

From first principles — how an array is stored

An array stores its elements contiguously in memory: one right after another, with no gaps, and every element takes the same number of bytes. For example, an array of 32-bit integers stores each element in 4 bytes, back to back.

Because of this regular layout, the computer does not need to "walk" through the array to find element i. It can compute the exact memory address of element i using simple arithmetic:

CODE
address(arr[i]) = base_address + (i * element_size)
base_address = where the array starts in memory (known once the array is created).
element_size = bytes per element (a fixed constant, e.g., 4 for an int).
i = the index you want.

So to read arr[i], the CPU does one multiplication, one addition, and one memory read — three operations, no matter how big the array is.

Why that makes it O(1)

Big-O describes how the amount of work grows as the input size n grows. Here, the number of operations to fetch arr[i] is the same whether the array has 10 elements or 10 million elements: it is always "compute the address, then read." The work does not depend on n and it does not depend on how large i is. When the work is bounded by a constant amount regardless of n, we call it constant time, O(1).

Contrast this with a linked list, where element i is only reachable by starting at the head and following i "next" pointers. There, accessing index i takes O(i) time (O(n) in the worst case), because there is no formula to jump straight to the element — the elements are scattered in memory and only chained together. This contrast is the cleanest way to show you understand why arrays are special: it is the contiguous, fixed-size layout that buys you O(1) indexing.

Common follow-up nuances (good to volunteer)
Read vs. search: Accessing by index is O(1). Searching for a value in an unsorted array is O(n) because you may have to scan every element. Interviewers sometimes see if you conflate the two — keep them separate.
Dynamic arrays (Python list, Java ArrayList, JS Array, C++ vector): Indexing is still O(1) — internally they are backed by a contiguous buffer. It is appending (which may occasionally trigger a resize/copy) that is "amortized O(1)", not indexing.
Cache friendliness: Beyond the theoretical O(1), contiguous storage is also fast in practice because the CPU cache loads nearby elements together. That's a bonus real-world point, though the complexity claim is O(1) regardless.
Space: The access itself uses O(1) extra space — you don't allocate anything proportional to n just to read one element.
Summary — what a strong answer emphasizes

Accessing an array element by index is O(1), constant time, because elements are stored contiguously and are equal-sized, so the address of element i is computed directly as base + i × element_size in a fixed number of steps — independent of the array's length. A strong answer states O(1), justifies it with the address formula, and contrasts it with a linked list's O(n) indexing to prove genuine understanding rather than memorization. The common mistake to avoid is confusing indexed access (O(1)) with searching for a value (O(n)).

**Comparison:**

```
Operation	Array (contiguous)	Linked list
Access element by index i	O(1)	O(n)
Search for a value (unsorted)	O(n)	O(n)
Reason for index access cost	address = base + i×size (direct)	must follow i pointers from head
```

**Code:**

```
C
Copy
// Conceptually, arr[i] is just pointer arithmetic + one dereference:
int value = *(base_address + i * sizeof(int));  // O(1)

// Which is exactly what arr[i] compiles to:
int value = arr[i];   // O(1): compute address, read once
```

**Example input / output:**

```

Array of ints (element_size = 4 bytes), base_address = 1000:

CODE
Copy
arr = [ 50, 60, 70, 80, 90 ]
index:  0   1   2   3   4

Access arr[3]:
  address = 1000 + (3 * 4) = 1012
  read memory at 1012  -> 80
Steps taken: 1 multiply + 1 add + 1 read  (same for arr[0] or arr[999999])
Output: 80   (in O(1) time)
```

