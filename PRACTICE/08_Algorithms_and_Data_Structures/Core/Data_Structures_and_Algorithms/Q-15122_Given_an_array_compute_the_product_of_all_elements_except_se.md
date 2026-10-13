---
type: "practice"
difficulty: "Deep"
---

# Given an array, compute the product of all elements except self without using division.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Prefix and Suffix Products](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Prefix_and_Suffix_Products.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What the question is really asking: Given an integer array nums, return an array out where out[i] is the product of every element except nums[i]. The two constraints are the whole point: (1) no division, and (2) the interviewer usually adds "in O(n) time." A naive product-of-all-then-divide is banned, and the brute-force O(n²) (for each i, multiply the other n−1 elements) is the trap to beat.

What the interviewer is probing for: Can you spot that the answer for each index is (product of everything to the left) × (product of everything to the right)? Do you understand why division is forbidden (zeros make it undefined/wrong, and interviewers want to see prefix/suffix thinking)? Can you then optimize the extra space from O(n) two arrays down to O(1) auxiliary by reusing the output array and carrying a running suffix product?

Clarifying questions I'd ask:

Can the array contain zeros? (Yes — this is the key case, and it's why division is banned.)
Can it contain negatives? (Yes — sign just flows through multiplication naturally.)
Are we guaranteed the full product fits in the integer type? Most versions promise the answer fits in 32-bit; if not I'd use 64-bit or BigInteger.
Does the output array count against space complexity? (Convention: the returned array does not count; only extra space beyond it does.)

Plan: (1) Establish the left×right insight. (2) Give the clean two-pass O(n) time / O(n) space version. (3) Optimize to O(1) auxiliary space. (4) Handle zeros/negatives and prove correctness. (5) Complexity and follow-ups.

From first principles to the solution
The core insight

out[i] = (nums[0] * ... * nums[i-1]) * (nums[i+1] * ... * nums[n-1]).

Call the first factor left[i] (product of all elements strictly to the left of i) and the second right[i] (product of all elements strictly to the right). Then out[i] = left[i] * right[i]. The empty product (nothing to the left of index 0, nothing to the right of index n−1) is defined as 1, the multiplicative identity — this is a common off-by-one bug if you forget it.

If we can compute all left[i] and all right[i] cheaply, we're done. And we can: left is a prefix product you build left-to-right in one pass, and right is a suffix product you build right-to-left in one pass. No division anywhere.

Version 1 — two arrays, O(n) time, O(n) extra space
CODE
left[0] = 1
for i in 1..n-1:  left[i]  = left[i-1]  * nums[i-1]
right[n-1] = 1
for i in n-2..0:  right[i] = right[i+1] * nums[i+1]
for i in 0..n-1:  out[i]   = left[i] * right[i]

This is correct and clear. But the interviewer will push: can you drop the two auxiliary arrays?

Version 2 — O(1) auxiliary space (the answer they want)

Two tricks combine:

Reuse the output array as the left array. Fill out[i] with the prefix product first.
Carry the suffix product in a single scalar as you sweep right-to-left, multiplying it into out[i] on the fly, and only then updating the scalar with nums[i].
CODE
out[0] = 1
for i in 1..n-1:  out[i] = out[i-1] * nums[i-1]   # out now holds left[]
suffix = 1
for i in n-1..0:
    out[i] = out[i] * suffix                       # left[i] * right[i]
    suffix = suffix * nums[i]                       # extend suffix for the next-left index

The only extra memory is the scalar suffix, so O(1) auxiliary space (the output array doesn't count by convention). Time is O(n) with two linear passes.

Why division is genuinely wrong, not just disallowed

The "compute total product P, then out[i] = P / nums[i]" approach fails on zeros:

One zero in the array: P is 0, and 0 / nums[i] is 0 for the zero's own slot (should be the product of the non-zero elements), and 0/0 is undefined at the zero. You'd need special-casing.
Two or more zeros: every output is 0.

The prefix/suffix method handles all three cases with zero special-casing because multiplication naturally propagates zeros: any index whose left or right span contains a zero gets a 0, and the single-zero index gets the product of all the non-zero elements. That elegance is the payoff of the approach.

Negatives

Nothing special — sign is just carried through multiplication. An even count of negatives to the side yields positive, odd yields negative. The algorithm is sign-correct automatically.

Overflow

If products can exceed the integer range, use a 64-bit type or BigInteger (Python integers are arbitrary precision, so no issue there). Overflow is a data-type concern, independent of the algorithm.

Complexity
Time: O(n) — two (or three) single passes, no nested loop.
Auxiliary space: O(1) for the optimized version (excluding the output array); O(n) for the clear two-array version.
Both are optimal in time: you must read all n inputs at least once.
Summary — what a strong answer emphasizes

Lead with the left[i] * right[i] decomposition and the empty-product-equals-1 base case. Deliver the two-pass prefix/suffix solution, then the space optimization that reuses the output array and carries a scalar suffix. Crucially, explain why division is forbidden — zeros — and show the prefix/suffix method handles one-zero, multi-zero, and negatives with no branching. State O(n)/O(1) and mention overflow as a type concern.

#### Common mistakes
& follow-up probes
Off-by-one: using nums[i] instead of nums[i-1]/nums[i+1] when building prefix/suffix, or forgetting the base value 1.
Falling back to division and then bolting on zero-counting logic — works but signals you missed the intended pattern.

#### Follow-ups
"Now division is allowed — do it in one pass with zero-count logic" (count zeros: 0 zeros → P/nums[i]; 1 zero → all 0 except the zero's slot which is product of non-zeros; ≥2 zeros → all 0). "What if it's a stream and you can't index backward?" "Generalize to a 2D grid / product-except-row-and-column."

**Comparison:**

```
Approach	Division?	Time	Extra space	Handles zeros cleanly
Brute force (multiply other n−1 each i)	No	O(n²)	O(1)	Yes
Total product then divide	Yes	O(n)	O(1)	No — needs zero-count special-casing
Prefix + suffix arrays	No	O(n)	O(n)	Yes
Prefix in output + suffix scalar	No	O(n)	O(1)	Yes
```

**Code:**

```

def product_except_self(nums): n = len(nums) out = [1] * n

CODE
Copy
# Pass 1: out[i] = product of everything to the LEFT of i (prefix product)
for i in range(1, n):
    out[i] = out[i - 1] * nums[i - 1]

# Pass 2: multiply in the product of everything to the RIGHT, carried in a scalar
suffix = 1
for i in range(n - 1, -1, -1):
    out[i] *= suffix          # left[i] * right[i]
    suffix *= nums[i]         # extend the running suffix product

return out
CODE
Copy

```java
// O(n) time, O(1) auxiliary space (output array not counted)
public int[] productExceptSelf(int[] nums) {
    int n = nums.length;
    int[] out = new int[n];

    out[0] = 1;
    for (int i = 1; i < n; i++) {
        out[i] = out[i - 1] * nums[i - 1];   // prefix products
    }

    int suffix = 1;
    for (int i = n - 1; i >= 0; i--) {
        out[i] *= suffix;                    // multiply in suffix product
        suffix *= nums[i];
    }
    return out;
}
```

**Example input / output:**

```

Example: nums = [1, 2, 3, 4], expected out = [24, 12, 8, 6].

Dry-run of the O(1) version:

Pass 1 (prefix into out):

CODE
Copy
i=0: out=[1, _, _, _]                (base)
i=1: out[1]=out[0]*nums[0]=1*1=1  -> [1,1,_,_]
i=2: out[2]=out[1]*nums[1]=1*2=2  -> [1,1,2,_]
i=3: out[3]=out[2]*nums[2]=2*3=6  -> [1,1,2,6]

Pass 2 (suffix scalar, right to left):

CODE
Copy
suffix=1
i=3: out[3]=6*1=6 ; suffix=1*4=4      -> [1,1,2,6]
i=2: out[2]=2*4=8 ; suffix=4*3=12     -> [1,1,8,6]
i=1: out[1]=1*12=12 ; suffix=12*2=24  -> [1,12,8,6]
i=0: out[0]=1*24=24 ; suffix=24*1=24  -> [24,12,8,6]

Result: [24, 12, 8, 6] ✓

Zero case: nums = [0, 4, 3] -> [12, 0, 0]

out[0] = 4*3 = 12 (product of non-zeros)
out[1] = 03 = 0, out[2] = 04 = 0 — handled with no special-casing.

Two-zero case: nums = [0, 2, 0] -> [0, 0, 0] ✓
```

**Flow:**

1. nums[]
2. Pass 1: out[i] = product of allLEFT of i (prefix)
3. suffix = 1
4. Pass 2 (right->left): out[i] *=suffix; suffix *= nums[i]
5. out[i] = left[i] * right[i] (nodivision)

