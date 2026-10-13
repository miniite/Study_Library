---
type: "practice"
difficulty: "Other"
---

# Iterator Over a Run-Length Compressed Sequence

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Design](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A sequence of non-negative integers is stored in a compressed run-length form. The compression is given as an integer array runs of even length (0-indexed) that is read in pairs:

For every even index i, the value runs[i] is a count and runs[i + 1] is the symbol. Together they mean: the integer runs[i + 1] appears runs[i] times, back to back, in the expanded (decompressed) sequence.

For example, the compressed array runs = [2, 4, 0, 7, 3, 9] describes the pairs (2, 4), (0, 7), (3, 9), which expand to [4, 4] followed by nothing (a count of 0) followed by [9, 9, 9], i.e. the full sequence [4, 4, 9, 9, 9]. Note that a count may be 0, in which case that symbol contributes no elements at all.

Design a reader that walks through the expanded sequence from left to right, consuming elements on demand without ever materializing the whole (possibly enormous) expansion. Implement a class RunReader:

RunReader(int[] runs) — Initializes the reader with the compressed array runs.
int fetch(int k) — Consumes (discards) the next k elements of the expanded sequence and returns the last element consumed. If fewer than k elements remain, consume everything that is left and return -1.

#### Cons
ecutive calls to fetch continue from wherever the previous call stopped; once the sequence is exhausted every further call returns -1.

#### Cons
traints:

2 <= runs.length <= 1000
runs.length is even.
0 <= runs[i] <= 10^9
1 <= k <= 10^9
At most 1000 calls will be made to fetch.

#### Example
s & test cases
TEXT
Input: RunReader([2,4,0,7,3,9]); fetch(1) -> Output: 4
Why: Expanded sequence is [4,4,9,9,9]; consuming 1 element takes the first 4.

Input: (same reader, continuing) fetch(2) -> Output: 9
Why: Next two elements are 4 then 9; the last one consumed is 9. Remaining: [9,9].

Input: (same reader, continuing) fetch(1) -> Output: 9
Why: One element left in the current run of 9s is taken. Remaining: [9].

Input: (same reader, continuing) fetch(2) -> Output: -1
Why: Only one element (9) remains but two were requested; the sequence is exhausted, so return -1.

Input: RunReader([3,5,4,5]); fetch(7) -> Output: 5
Why: Whole expansion is [5,5,5,5,5,5,5] (three 5s then four 5s); consuming exactly 7 returns 5.

Input: RunReader([0,8,0,3,2,6]); fetch(1) -> Output: 6
Why: The first two runs have count 0 and contribute nothing, so the very first real element is 6.

#### Approach

Keep a single forward-moving pointer at the current "count" cell and a copy of the compressed array so the leftover of a partially consumed run can be recorded in place. On each fetch(k), walk forward over runs whose remaining count is strictly less than k, subtracting each count from k, until you reach a run whose count is at least k (partial consumption) or fall off the end (exhausted -> return -1). Then decrement that run's count by k and return its symbol. The key insight is that the expanded sequence is never built; the pointer plus the mutable count give O(1) amortized progress, which is essential because counts and k can be as large as 10^9.

Complexity

Time: amortized O(1) per fetch. A single call is O(r) where r is the number of runs it fully skips, but the index pointer only ever moves forward across the array over the object's whole lifetime, so the total work across all calls is O(runs.length + number_of_calls). Space: O(runs.length) for the mutable copy of the compressed array; O(1) extra per call. Using 64-bit accumulation avoids overflow since counts and k reach 10^9.

Python
PYTHON
from typing import List

class RunReader:
    def __init__(self, runs: List[int]):
        self.runs = runs[:]   # mutable copy; front count shrinks as consumed
        self.idx = 0          # index of current count cell

    def fetch(self, k: int) -> int:
        # Skip runs fully consumed by this request.
        while self.idx < len(self.runs) and self.runs[self.idx] < k:
            k -= self.runs[self.idx]
            self.idx += 2
        if self.idx == len(self.runs):
            return -1                       # exhausted
        self.runs[self.idx] -= k            # consume k from current run
        return self.runs[self.idx + 1]
JavaScript
JAVASCRIPT
class RunReader {

#### cons
tructor(runs) {
        this.runs = runs.slice(); // mutable copy; front count shrinks as consumed
        this.idx = 0;             // index of current count cell
    }

    fetch(k) {
        // Skip runs fully consumed by this request.
        while (this.idx < this.runs.length && this.runs[this.idx] < k) {
            k -= this.runs[this.idx];
            this.idx += 2;
        }
        if (this.idx === this.runs.length) return -1; // exhausted
        this.runs[this.idx] -= k;                     // consume k from current run
        return this.runs[this.idx + 1];
    }
}
Java
JAVA
class RunReader {
    private final long[] runs;   // mutable copy; front count shrinks as consumed
    private int idx = 0;         // index of current count cell

    public RunReader(int[] runs) {
        this.runs = new long[runs.length];
        for (int i = 0; i < runs.length; i++)
            this.runs[i] = runs[i];
    }

    public int fetch(long k) {
        // Skip runs fully consumed by this request.
        while (idx < runs.length && runs[idx] < k) {
            k -= runs[idx];
            idx += 2;
        }
        if (idx == runs.length)
            return -1;                 // exhausted
        runs[idx] -= k;                // consume k from the current run
        return (int) runs[idx + 1];
    }
}
C++
CPP
#include <vector>
using namespace std;

class RunReader {
public:
    RunReader(vector<int>& runs) : runs(runs.begin(), runs.end()) {}

    int fetch(long long k) {
        // Skip runs fully consumed by this request.
        while (idx < runs.size() && runs[idx] < k) {
            k -= runs[idx];
            idx += 2;
        }
        if (idx == runs.size())
            return -1;                 // exhausted
        runs[idx] -= k;                // consume k from the current run
        return static_cast<int>(runs[idx + 1]);
    }

private:
    vector<long long> runs;            // mutable copy
    size_t idx = 0;                    // current count cell
};
C
C
#include <stdlib.h>

typedef struct {
    long long *runs;  /* mutable copy: front count is decremented as we consume */
    int size;
    int idx;          /* index of the current count cell */
} RunReader;

RunReader *runReaderCreate(int *runs, int runsSize) {
    RunReader *it = (RunReader *)malloc(sizeof(RunReader));
    it->runs = (long long *)malloc(sizeof(long long) * runsSize);
    for (int i = 0; i < runsSize; i++)
        it->runs[i] = runs[i];
    it->size = runsSize;
    it->idx = 0;
    return it;
}

int runReaderFetch(RunReader *it, long long k) {
    /* Skip whole runs that are fully consumed by this request. */
    while (it->idx < it->size && it->runs[it->idx] < k) {
        k -= it->runs[it->idx];
        it->idx += 2;
    }
    if (it->idx == it->size)
        return -1;                    /* ran off the end -> exhausted */
    it->runs[it->idx] -= k;           /* consume k from the current run */
    return (int)it->runs[it->idx + 1];
}

void runReaderFree(RunReader *it) {
    free(it->runs);
    free(it);
}

#### Follow-up
questions
Support a peek() that returns the next element without consuming it.
Make the structure immutable / thread-safe so multiple readers can iterate the same compressed data independently.
Handle a streaming source where new (count, symbol) pairs are appended over time rather than a fixed array.
Return how many elements were actually consumed when the request runs past the end, not just -1.

