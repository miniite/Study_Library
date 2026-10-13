---
type: "practice"
difficulty: "Deep"
---

# Explain what amortized time complexity means using a dynamic array as an example.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Sorting Algorithms](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Sorting_Algorithms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question asks me to explain amortized time complexity and to make it concrete with a dynamic array (Java ArrayList, C++ std::vector, Python list, Go slice) as the running example. The interviewer is a strong Google/Amazon/Microsoft/Meta panel probing "deep understanding" — so they don't want the textbook one-liner. They want to see that I understand:

What amortized cost is and how it differs from average-case and worst-case.
Why a single append can be O(n) yet a sequence of appends is O(1) each.
The actual proof (aggregate / accounting / potential method) that the amortized cost is O(1), and specifically why the growth factor must be multiplicative (e.g. 2x), not additive.
The trade-offs and failure modes: it is not a probabilistic guarantee, the worst single operation is still O(n), and it interacts with memory, latency tails, and real-time systems.

#### Assumptions
/ clarifications I'd state: I'll assume the classic "double the capacity when full" growth policy, and I'll treat push_back/append at the end of the array. I'll note that inserting in the middle is a different O(n) story unrelated to amortization.

Plan: define amortized cost → describe the dynamic array mechanics → prove O(1) three ways → contrast with average-case → discuss growth factor and real-world implications → summarize.

What "amortized" actually means

Amortized analysis answers a specific question: when a sequence of operations is performed, what is the average cost per operation, guaranteed across the whole sequence — not assuming any randomness?

The key phrase is guaranteed across the whole sequence. This is what distinguishes it from average-case:

Worst-case looks at the single most expensive operation in isolation.
Average-case assumes a probability distribution over inputs and takes an expectation — it can be defeated by an adversary who feeds you the bad case every time.
Amortized makes no probabilistic assumption. It says: even in the absolute worst sequence of operations, the total cost divided by the number of operations is bounded. Expensive operations are "paid for" by many cheap ones that necessarily precede them. It is a deterministic worst-case guarantee over a sequence.

So "amortized O(1) append" means: any sequence of n appends costs O(n) total, hence O(1) each on average, and this holds even for the worst possible sequence — there is no distribution or luck involved.

The dynamic array mechanics

A dynamic array stores elements in a contiguous backing buffer with a size (elements used) and a capacity (slots allocated). Appending:

If size < capacity: write the element, size++. This is O(1) — cheap.
If size == capacity (buffer full): allocate a new buffer of capacity × 2, copy all size existing elements into it, free the old buffer, then write the new element. This is O(n) — the expensive "resize."

So most appends are O(1), but the ones that trigger a resize are O(n). The worst single append is genuinely O(n) — that part of the naive worry is correct. The insight is that resizes become exponentially rarer as the array grows.

Proof that appends are amortized O(1)
Method 1 — Aggregate analysis

#### Cons
ider n appends starting from an empty array with doubling. Resizes happen when size crosses 1, 2, 4, 8, …, up to n. The copy cost of all resizes is:

1 + 2 + 4 + 8 + … + n ≤ 2n

That's a geometric series that sums to less than twice its largest term. Add the n O(1) writes and total work is ≤ 2n + n = 3n = O(n). Divide by n operations → O(1) amortized per append. The whole trick is the geometric series: because capacity doubles, the total copy work is dominated by the last resize, and the sum stays linear.

Method 2 — Accounting (banker's) method

Charge each append 3 units (its amortized cost). A cheap write costs 1 unit actually; bank the extra 2. When a resize copies element x, that copy is paid for by the 2 saved units x deposited earlier (one for copying itself, one held in reserve for the next growth cycle). Because doubling means every element between the last resize and this one deposited enough credit, the bank balance never goes negative → amortized cost ≤ 3 = O(1).

Method 3 — Potential method

Define potential Φ = 2·size − capacity. It's ≥ 0 right after a resize and grows by 2 per cheap append. Amortized cost = actual cost + ΔΦ. A cheap append: actual 1, ΔΦ = 2 → amortized 3. A resize of a full array (size = capacity = k): actual is k (copy) + 1, but Φ drops from 2k − k = k to 2(k+1) − 2k = 2, so ΔΦ = 2 − k, giving amortized (k+1) + (2 − k) = 3. Every case → O(1).

All three converge on the same answer, which is exactly what a "deep understanding" answer should show.

Why the growth factor matters (multiplicative, not additive)

The magic depends on multiplicative growth. Suppose instead we grow by a fixed additive amount — say capacity += 1 (or += 100). Then resizes happen every 1 (or 100) appends, and the copy costs form an arithmetic series 1 + 2 + 3 + … + n ≈ n²/2 = O(n²) total → O(n) amortized per append. Amortization collapses. Only geometric growth keeps the series linear.

The chosen factor is a time/space trade-off:

2× wastes up to ~50% of memory right after a resize, and (a subtle point) the freed buffers can never be reused to satisfy the next allocation because 1+2+4+…+2^k < 2^{k+1}.
1.5× (used by MSVC's vector, Facebook's folly::fbvector) wastes less memory and can let the allocator reuse freed blocks, at the cost of slightly more frequent copies. Still O(1) amortized because any factor > 1 keeps the series geometric.
Amortized ≠ average-case, and its failure modes

A crucial distinction for senior interviews: amortized O(1) is not a statement about typical or expected cost — it is a hard guarantee over the sequence. But it is not a per-operation latency guarantee. The individual append that triggers a resize is still O(n) and will show up as a latency spike / tail latency. This matters:

Real-time / low-latency systems (audio, HFT, robotics) may forbid a single O(n) stall, so they pre-reserve() capacity, use a fixed-size ring buffer, or use structures with true worst-case O(1) ops.
Memory pressure: doubling near a large size briefly needs old + new buffer simultaneously (≈1.5–2× the data live at once) and can spike RSS or fail an allocation.
The guarantee is per-sequence, not resettable adversarially — that's its strength over average-case, which an adversary could break.

Contrast this with a hash table's O(1) average-case lookup, which does assume a good hash distribution and can degrade to O(n) under adversarial keys — a genuinely probabilistic claim, unlike the dynamic array's deterministic amortized one.

Summary — what a strong answer emphasizes

Amortized complexity is the guaranteed average cost per operation across an entire sequence, with no probabilistic assumption. For a dynamic array with doubling: individual appends are O(1) except rare O(n) resizes, but because capacity grows geometrically, the total copy cost over n appends is a geometric series bounded by 2n, so each append is amortized O(1). I'd prove it via aggregate/accounting/potential, stress that multiplicative growth is essential (additive → O(n) amortized), distinguish amortized from average-case (deterministic vs. probabilistic), and flag the real-world caveat that the worst single op is still O(n) and causes tail-latency and transient memory spikes.

#### Common mistakes
to avoid: saying "amortized = average-case" (wrong — no distribution involved); claiming every append is O(1) (the resize one isn't); and forgetting that the argument only works because of geometric growth. A great follow-up probe: "prove additive growth is O(n) amortized" — which the arithmetic-series argument above answers directly.

**Comparison:**

```
Notion	What it measures	Assumes randomness?	Single-op worst case	Example
Worst-case	Most expensive single op	No	Yes, this IS it	Resize append = O(n)
Average-case	Expected cost over input distribution	Yes	Can be worse	Hash lookup O(1) avg, O(n) adversarial
Amortized	Total cost / #ops over a sequence, guaranteed	No	Still O(n)	Dynamic-array append = O(1) amortized
Growth policy	Resize frequency	Total copy cost over n appends	Amortized per append	Memory waste
×2 (doubling)	rarer geometrically	≤ 2n (geometric)	O(1)	up to ~50%
×1.5	slightly more frequent	O(n) (geometric)	O(1)	~33%, buffers reusable
+k (additive)	every k appends	~n²/(2k) (arithmetic)	O(n)	low but slow
```

**Code:**

```

// Minimal dynamic array showing the amortized-O(1) append with doubling. class DynamicArray { private int[] data = new int[1]; // backing buffer private int size = 0; // elements used private int capacity = 1; // slots allocated

CODE
Copy
public void append(int value) {
    if (size == capacity) {       // full -> the rare O(n) resize
        capacity *= 2;            // MULTIPLICATIVE growth is what makes it O(1) amortized
        int[] bigger = new int[capacity];
        System.arraycopy(data, 0, bigger, 0, size); // copy all n existing elements
        data = bigger;
    }
    data[size++] = value;         // the common O(1) write
}

public int get(int i) { return data[i]; } // always O(1)
public int size()     { return size; }

}

CODE
Copy

```java
// Counter-example: ADDITIVE growth degrades to O(n^2) total -> O(n) amortized.
// Replace the resize line above with:  capacity += 1;
// Now a resize fires on EVERY append; total copy work = 1+2+...+n ~ n^2/2.
```

**Example input / output:**

```text
Dry run of 8 appends into an empty array (capacity starts at 1, doubling):

append #  size before  capacity  action                       copy cost
   1          0           1      write (fits)                     0
   2          1           1      FULL -> grow to 2, copy 1, write 1
   3          2           2      FULL -> grow to 4, copy 2, write 2
   4          3           4      write (fits)                     0
   5          4           4      FULL -> grow to 8, copy 4, write 4
   6          5           8      write (fits)                     0
   7          6           8      write (fits)                     0
   8          7           8      write (fits)                     0

Total copy cost = 1 + 2 + 4 = 7  (< 2*8 = 16)   [geometric series]
Total writes    = 8
Total work      = 15 for 8 appends  ->  ~1.9 per append  ->  O(1) amortized

Resizes occur at sizes 1, 2, 4, ... -> exponentially rarer as n grows.
```

**Flow:**

1. No, buffer has room
2. Yes, buffer full
3. paid for by the many cheapops before it
4. append value
5. size == capacity?
6. data at size = value; size++
7. O 1 : common case
8. capacity *= 2
9. allocate new buffer
10. copy all n existing elements
11. write new value; size++
12. O n : rare, exponentially lessfrequent

