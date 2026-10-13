---
type: "practice"
difficulty: "Deep"
---

# Why is appending to a dynamic array (like ArrayList or Python list) O(1) amortized despite resizing?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Amortized Complexity](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Amortized_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked: Explain why append/push_back on a growable array (Java ArrayList, Python list, C++ std::vector) is O(1) amortized even though some appends must resize by allocating a bigger backing array and copying every element (an O(n) operation).

What the interviewer is probing (deep understanding, SDE/SDE-2/3): Do I understand (1) the mechanics of the backing array + resize, (2) that resizes are rare and self-limiting, (3) the actual proof that total work over n appends is O(n) — i.e. the geometric series — and (4) the crucial detail that the growth factor must be multiplicative (geometric), not additive. Bonus senior points for naming the accounting/potential proofs, real growth factors, and the tail-latency caveat.

#### Assumptions
/clarifications: I'll assume appending at the end (not insert(0,...)), and the standard doubling model for the core proof, then note real-world factors (Java 1.5×, Python ~1.125×). "O(1) amortized" means average per-append over any sequence of n appends is constant — a guarantee, not a probabilistic average.

Plan: Describe the backing array → why a resize is O(n) → count how often resizes happen and how much they copy → sum the geometric series to get O(n) total → divide to get O(1) amortized → prove it a second way with the accounting method → discuss why geometric (not fixed) growth is essential → real-world numbers and the tail-latency caveat.

The mechanics: a fixed-size array under the hood

A dynamic array stores elements in a contiguous fixed-capacity backing array plus two numbers: size (elements in use) and capacity (slots allocated). Appending has two cases:

Fast path (common): size < capacity. Write the element at index size, increment size. This is O(1) — one write.
Slow path (rare): size == capacity, the array is full. We must (a) allocate a new, larger backing array, (b) copy all size existing elements into it, (c) then write the new element. Step (b) is O(size) = O(n).

So a single append is O(n) worst case. The claim is that averaged over many appends it's O(1). The whole argument is: the O(n) resizes are rare enough that their total cost, summed over all appends, is still only O(n).

Why resizes are rare and self-limiting: geometric growth

The trick is that when the array grows, it grows multiplicatively — typically doubling. If capacity doubles each time (1 → 2 → 4 → 8 → …), then after a resize we have a whole extra array's worth of free slots before the next resize. The number of appends between consecutive resizes keeps doubling, so resizes get exponentially farther apart.

Concretely, to reach n elements starting from capacity 1 with doubling, resizes occur at sizes 1, 2, 4, 8, …, up to the largest power of two ≤ n — that's only about log₂(n) resizes, and the copy costs are 1, 2, 4, …, ≈ n.

The proof: sum the geometric series (aggregate method)

Total work for n appends = (n cheap writes) + (all the copy work during resizes).

Copy work = 1 + 2 + 4 + 8 + … + (largest power of two < n). A geometric series with ratio 2 sums to less than 2n (a series summing up to value X totals < 2X). So:

Total = n + (< 2n) = < 3n = O(n).

Amortized cost per append = total / n = 3n / n = O(1). Done. The key algebraic fact is that a doubling series is dominated by its last term — all the earlier copies together cost less than the final copy, so the copies never accumulate into more than linear total work.

A second proof: the accounting (banker's) method

Charge each append 3 units of "amortized cost."

1 unit pays for writing the new element (the fast-path work).
2 units are saved as credit on the newly added element.

When a resize copies size elements: since the last resize doubled capacity, at least size/2 of the current elements were added after that resize, and each of those carries 2 saved credits → 2 × (size/2) = size credits available — exactly enough to pay 1 unit to move each of the size elements. Credit never goes negative, so the flat 3-per-append charge is always sufficient → each append is O(1) amortized. (The potential method with Φ = 2·size − capacity gives the same result.)

Why the growth factor MUST be geometric — the counter-example

If instead the array grew by a fixed increment k (say +10 slots) each time, a resize would occur every k appends, and each resize copies O(current size). Total copy work = 10 + 20 + 30 + … + n ≈ n²/(2k) = O(n²) → amortized O(n) per append. That's a disaster — it's why no serious dynamic array uses additive growth.

Any factor > 1 works. Doubling (×2) is simplest for proofs. Real libraries pick different factors to trade memory waste against copy frequency:

Java ArrayList: grows ~1.5× (oldCap + (oldCap >> 1)).
C++ std::vector: implementation-defined, commonly 1.5× (GCC/Clang) or 2× (MSVC).
Python list: grows ~1.125× (newsize + (newsize >> 3) + (3 or 6)) — a modest over-allocation.
Go slices: ~2× for small, tapering toward 1.25× for large.

Smaller factors (1.25–1.5) waste less memory but resize more often; all are still O(1) amortized because any fixed factor > 1 yields a convergent geometric series. (Aside: 1.5× is sometimes preferred over 2× because the freed old blocks can be reused by the allocator, reducing fragmentation.)

The nuance: amortized ≠ worst-case per operation

O(1) amortized does not mean every append is fast. The append that triggers a resize is genuinely O(n) — it allocates and copies millions of elements. Amortization means the total over n appends is O(n), so throughput is constant per element. But a single append can still cause a latency spike. In p99-sensitive systems (payments/fintech hot paths, real-time services), that spike matters. Mitigations:

Pre-size: new ArrayList<>(expectedCount) / vec.reserve(n) / lst = [None]*n — avoids all resizes when the size is known.
De-amortize: use structures that spread copy work across operations if a hard per-op bound is required.

Also note: geometric growth means up to ~50–100% temporary memory overhead at the moment of resize (old + new arrays coexist during the copy), and up to ~(factor−1) steady-state slack. That's the space cost you pay for O(1)-amortized time.

Summary — what a strong answer emphasizes
An append is O(1) on the fast path; only a full array forces an O(n) resize (allocate bigger + copy all).
Because capacity grows geometrically (doubling), resizes are exponentially rare and their copy costs form a geometric series 1+2+4+…+n < 2n, so all resizes together cost only O(n).
Total work over n appends = O(n) ⇒ O(1) amortized per append; provable by aggregate, accounting, or potential methods.
The magic ingredient is the multiplicative growth factor > 1; additive growth would make it O(n) amortized (O(n²) total).
Amortized O(1) ≠ per-op O(1): the resizing append still spikes to O(n) — pre-size or de-amortize when tail latency matters.

#### Common mistakes
saying "n appends are O(n²)" (forgetting the series converges); confusing amortized with average-case (there's no input distribution — it's a worst-case-sequence guarantee); forgetting that fixed-increment growth ruins the bound; thinking every append is O(1) (the resizing one isn't).

**Comparison:**

```
Growth strategy	Resizes for n appends	Total copy work	Amortized append	Verdict
Double (×2)	~log₂ n (~20 for 1M)	1+2+…+n < 2n	O(1)	Standard, simplest proof
×1.5 (Java ArrayList)	~log₁.₅ n	~2n	O(1)	Less memory slack
×1.125 (Python list)	~log₁.₁₂₅ n	~8n	O(1)	Tight memory, more copies
Fixed +k (additive)	n/k	10+20+…+n ≈ n²/2k	O(n) ❌	Never used — O(n²) total
Per-append view	Cost	Frequency
Fast path (space available)	O(1) — one write	Almost always
Slow path (array full → resize)	O(n) — alloc + copy all	~log(n) times total
Averaged over n appends	O(1) amortized	—
```

**Code:**

```python
# Instrumented growable array: prove total copies over n appends is O(n) (=> O(1) amortized)
class GrowArray:
    def __init__(self, factor=2):
        self.factor = factor
        self.cap = 1
        self.n = 0
        self.buf = [None] * self.cap
        self.resizes = 0
        self.total_copies = 0

    def append(self, x):
        if self.n == self.cap:                     # RARE slow path: O(n)
            self.cap = max(self.cap + 1, int(self.cap * self.factor))
            new = [None] * self.cap
            for i in range(self.n):                # copy all existing
                new[i] = self.buf[i]
            self.total_copies += self.n
            self.resizes += 1
            self.buf = new
        self.buf[self.n] = x                        # COMMON fast path: O(1)
        self.n += 1

for f in (2.0, 1.5, 1.125):
    a = GrowArray(factor=f)
    N = 1_000_000
    for i in range(N):
        a.append(i)
    print(f"factor={f}: appends={N}, resizes={a.resizes}, "
          f"total_copies={a.total_copies}, copies/append={a.total_copies/N:.3f}")
```

**Example input / output:**

```text
Output (N = 1,000,000 appends):

factor=2.0:   appends=1000000, resizes=20, total_copies=1048575, copies/append=1.049
factor=1.5:   appends=1000000, resizes=35, total_copies=~2.0M,    copies/append≈2.0
factor=1.125: appends=1000000, resizes=~92, total_copies=~8.0M,   copies/append≈8.0

Reading it:
- Resizes grow like log_factor(n): only ~20 resizes for a MILLION appends at 2x.
- copies/append stays a small CONSTANT (independent of N) => O(1) amortized.
- Smaller factor -> more resizes & more total copies, but still a constant per append
  (constant grows to ~1/(factor-1)), never O(n). This IS the amortized-O(1) guarantee.
- Contrast: additive growth (+k) would give copies/append ~ N/(2k) -> grows with N -> O(N).
```

**Flow:**

1. No, has room
2. Yes, full
3. single resize still spikes toO(n)
4. append(x)
5. size == capacity?
6. buf[size] = x; size++O(1) — the common case
7. allocate new = capacity ×factorcopy all size elements O(n)then write x
8. Resizes occur at cap =1,2,4,8,...,ncopy costs 1+2+4+...+n < 2n
9. Total over n appends = n +<2n = O(n)
10. Amortized = O(n)/n = O(1) perappend
11. tail-latency risk → pre-size /reserve()

