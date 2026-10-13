---
type: "practice"
difficulty: "Intermediate"
---

# Generate all permutations of a given string and describe the recursion tree.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Permutations](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Permutations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is being asked: Write code to generate all permutations of a given string, and describe the recursion tree that the algorithm explores.

What the interviewer is probing: Whether you can turn "all orderings" into a clean recursive/backtracking algorithm, whether you understand the shape and size of the recursion tree (branching factor, depth, number of leaves), whether you get the choose → recurse → un-choose bookkeeping right, and whether you handle the practical wrinkle: duplicate characters producing duplicate permutations. The "describe the recursion tree" part is the tell — they want you to reason about why there are n! permutations and where the work happens, not just paste a snippet.

Clarifying questions worth stating:

Should output include duplicates when the string has repeated characters (e.g. "aab"), or unique permutations only? (I'll handle both; the de-dup version is the interesting one.)
Any ordering requirement on the output? (Sorting the input gives lexicographic order and enables clean de-duplication.)
String length bounds? (n! blows up fast — n=10 is ~3.6M, n=12 is ~479M — so this is only feasible for small n.)

Plan: Use backtracking. At each recursion depth we choose one unused character to place next; we recurse to fill the remaining positions; then we un-choose to restore state for the sibling branches. I'll describe the tree, its size, and the de-dup trick, then give complexity.

Building the algorithm from first principles

A permutation is an arrangement — pick a first character, then a first-of-the-rest, and so on until no characters remain. That "pick one of the remaining, then solve the smaller problem" structure is naturally recursive.

State I carry:

path — the characters chosen so far (the prefix we're building).
used[] — a boolean array marking which input indices are already in path (so each character is used exactly once per permutation).

The recursive step:

Base case: if path.length == n, the prefix is a full-length arrangement — record it and return.
Recursive case: loop over every index i; if used[i] is false, choose it (append s[i] to path, mark used[i]=true), recurse to fill the next position, then un-choose (pop from path, set used[i]=false) so the next sibling iteration starts from a clean state.

The un-choose step is essential: because we reuse the same path/used across sibling branches, we must undo each choice on the way back up, or later branches inherit stale state.

Describing the recursion tree (the heart of the question)

Think of an implicit tree:

The root is the empty prefix "" — nothing chosen yet.
Depth 0 → 1: the root has n children, one per possible first character.
Depth 1 → 2: each of those nodes has n−1 children (any character except the one already used).
In general, a node at depth d (d characters already placed) has n − d children.
Depth n: the tree bottoms out. Each leaf is a complete permutation.

So the branching factor decreases by one at each level: n, then n−1, then n−2, … , 1. The number of leaves is therefore n × (n−1) × (n−2) × … × 1 = n! — which is exactly the count of permutations. That is the clean justification for "there are n! permutations": it falls straight out of the tree's shrinking branching factor.

Total nodes in the tree is n! + n!/1! + n!/2! + … ≈ e·n!, so it's Θ(n!) nodes overall, dominated by the leaf level. Each leaf does O(n) work to copy the permutation into the output, giving the complexities below.

For the concrete case s = "abc" (n=3) the tree is:

CODE
                       "" (root)
        ┌───────────────┼───────────────┐
       a                b                c            <- depth 1: 3 branches
     ┌──┴──┐          ┌──┴──┐          ┌──┴──┐
    ab    ac        ba    bc        ca    cb          <- depth 2: 2 branches each
    │      │         │     │         │     │
   abc    acb       bac   bca       cab   cba         <- depth 3: leaves = 3! = 6

Reading the six leaves left to right gives all 3! = 6 permutations: abc, acb, bac, bca, cab, cba.

Handling duplicate characters (the practical extension)

If the input has repeats, e.g. "aab", the naive algorithm emits duplicate permutations (the two as are interchangeable, so a₁a₂b and a₂a₁b look identical). The standard fix:

Sort the string first, so equal characters are adjacent.
At each recursion level, skip a character if it equals the previous character AND that previous equal character is not currently used (s[i] == s[i-1] && !used[i-1]). This means "only start a run of equal characters in a fixed canonical order," which prunes the duplicate subtrees.

This turns the leaf count from n! down to the number of distinct permutations, n! / (k₁! · k₂! · …) where kⱼ are the multiplicities. It's a pruning of the same recursion tree — whole duplicate subtrees are cut off before recursing.

Complexity
Time: Θ(n × n!) — there are n! leaves and copying each length-n permutation to the output costs O(n). (With duplicates and pruning, it's proportional to the number of distinct permutations × n.)
Space: O(n) auxiliary for the recursion stack, path, and used[] (depth ≤ n). The output itself is Θ(n × n!) since we store every permutation — usually not counted as "extra" space, but worth stating.
What a strong answer emphasizes & common mistakes
Correctly derives n! leaves from the decreasing branching factor (n, n−1, …, 1) — that's the "describe the recursion tree" payoff the interviewer wants.
Gets the choose/recurse/un-choose bookkeeping right (the missing un-choose is the #1 bug).
Proactively handles duplicates with sort + skip, and states the distinct-count formula.
States realistic feasibility limits (n! explodes — don't offer this for n > ~10 without warning).

#### Common mistakes
forgetting to un-mark used[i] after recursion (stale state → wrong/missing outputs); appending the shared path object to the result instead of a copy (all entries end up identical/empty at the end); using string concatenation heavily (fine correctness-wise but O(n²) copies — a StringBuilder/char array is cleaner); for the de-dup case, using a HashSet on final strings (works but wasteful) instead of the sort-and-skip prune.

#### Follow-up
probes: "Generate the k-th permutation directly" (factorial number system — O(n²), no full tree); "next permutation in-place" (the classic O(n) suffix-reversal algorithm); "iterative / Heap's algorithm" for permutations that swaps in place with fewer allocations.

**Comparison:**

```
Aspect	Value	Why
Branching factor at depth d	n − d	d chars already placed, n−d remain
Number of leaves	n!	product of n·(n−1)·…·1
Total nodes	≈ e·n! (Θ(n!))	sum over all levels, leaf level dominates
Time	Θ(n · n!)	n! leaves × O(n) to copy each permutation
Extra space (excl. output)	O(n)	recursion depth + path + used[]
Distinct perms with dup chars	n! / (k₁!·k₂!·…)	kⱼ = multiplicity of each distinct char
```

**Code:**

```java
import java.util.*;

class Permutations {
    // ---- All permutations (assumes distinct chars, or keeps duplicates) ----
    public List<String> permute(String s) {
        List<String> out = new ArrayList<>();
        boolean[] used = new boolean[s.length()];
        backtrack(s, new StringBuilder(), used, out);
        return out;
    }

    private void backtrack(String s, StringBuilder path, boolean[] used, List<String> out) {
        if (path.length() == s.length()) {   // base case: a full-length leaf
            out.add(path.toString());        // copy the prefix into the output
            return;
        }
        for (int i = 0; i < s.length(); i++) {
            if (used[i]) continue;           // each index used once per permutation
            used[i] = true; path.append(s.charAt(i));   // choose
            backtrack(s, path, used, out);               // recurse (fill next slot)
            path.deleteCharAt(path.length() - 1); used[i] = false; // un-choose
        }
    }

    // ---- Unique permutations only (handles duplicate characters) ----
    public List<String> permuteUnique(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);                      // group equal chars so we can skip runs
        List<String> out = new ArrayList<>();
        boolean[] used = new boolean[a.length];
        btUnique(a, new StringBuilder(), used, out);
        return out;
    }

    private void btUnique(char[] a, StringBuilder path, boolean[] used, List<String> out) {
        if (path.length() == a.length) { out.add(path.toString()); return; }
        for (int i = 0; i < a.length; i++) {
            if (used[i]) continue;
            // skip a duplicate: only start a run of equal chars in canonical order
            if (i > 0 && a[i] == a[i - 1] && !used[i - 1]) continue;
            used[i] = true; path.append(a[i]);
            btUnique(a, path, used, out);
            path.deleteCharAt(path.length() - 1); used[i] = false;
        }
    }
}
```

**Example input / output:**

```text
permute("abc")  ->  [abc, acb, bac, bca, cab, cba]   (3! = 6 results)

Recursion tree traversal order (pre-order = output order):
  ""
   a -> ab -> abc*        (record abc)
        ab <- backtrack, next child of a
   a -> ac -> acb*
   (backtrack up to root, next child)
   b -> ba -> bac*
   b -> bc -> bca*
   c -> ca -> cab*
   c -> cb -> cba*
  (* = leaf recorded)

permuteUnique("aab")  ->  [aab, aba, baa]    (only 3! / 2! = 3 distinct)
  Without the skip you'd get 6 entries with aab/aba/baa each doubled.
```

**Flow:**

1. '' (root)
2. a
3. b
4. c
5. ab
6. ac
7. ba
8. bc
9. ca
10. cb
11. abc (leaf)
12. acb (leaf)
13. bac (leaf)
14. bca (leaf)
15. cab (leaf)
16. cba (leaf)

