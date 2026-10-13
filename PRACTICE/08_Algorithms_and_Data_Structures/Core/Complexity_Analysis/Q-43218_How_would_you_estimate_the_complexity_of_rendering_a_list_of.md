---
type: "practice"
difficulty: "Intermediate"
---

# How would you estimate the complexity of rendering a list of n items with nested filtering logic in a React frontend?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Complexity vs Real Performance](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Complexity_vs_Real_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked? The interviewer wants me to reason about the asymptotic cost of a React render the same way I'd reason about any algorithm — but applied to the UI layer, where the "work" is building a virtual DOM tree, running filter logic, and letting React diff/commit. The specific scenario: I have n items and I'm applying nested filtering logic before rendering the list.

What are they probing for?

Do I understand that a React render is just executing JavaScript, so normal Big-O reasoning applies?
Can I separate the data-transformation cost (my filtering) from React's reconciliation cost (diffing)?
Do I know the frontend-specific traps: re-filtering on every render, O(n·m) nested filters, missing memoization, and keys?
Can I connect complexity to actual UX symptoms (jank, dropped frames, slow typing in a search box)?

Key assumptions / clarifying questions I'd raise:

How is "nested" defined? Two independent filters chained (filter().filter()) → still O(n). A predicate that itself loops over m tags/children per item → O(n·m). A tree of nested items rendered recursively → cost is total nodes, not top-level n.
Does filtering run on every keystroke (search) or once per data load? That changes whether we care about the per-render cost.
Is n in the tens (cheap, ignore it) or tens of thousands (need virtualization)?

Plan: decompose one render into its phases, assign a cost to each, combine them, then discuss what dominates and how to reduce it.

From first principles: what happens in one render

A React render of a filtered list has three phases, and total cost is their sum:

Compute the derived data (filtering). This is plain JS. items.filter(pred) is O(n) calls to pred. If pred is O(1) (compare a field), filtering is O(n). If pred loops over m sub-elements (nested tags, a .some() over children, a substring/regex scan of length k), it's O(n·m) or O(n·k). Chained filters a.filter(f).filter(g) are O(n) + O(n') ≈ O(n), not multiplicative — a common point candidates get wrong.

Build the element tree (your JSX). Mapping the filtered result to <Row/> elements is O(f) where f is the number of items that survived filtering (f ≤ n). Each <Row/> is a cheap object allocation; the JSX itself doesn't touch the DOM yet.

Reconciliation + commit (React's job). React diffs the new element tree against the previous one. With stable keys this is O(f) — one comparison per item. The commit (actual DOM mutations) is O(Δ), proportional to how many nodes actually changed, not to f. This is why keys matter: bad keys (using array index) turn a small logical change into a large DOM churn, and missing keys make React fall back to positional diffing that can re-render/re-mount many rows.

So the overall render is: O(n) to filter (or O(n·m) if the predicate is nested) + O(f) to build and diff + O(Δ) to commit. In the common case this collapses to O(n) time, O(f) extra space for the filtered array and element tree.

Where "nested" bites you

The word nested is the trap. Two realistic interpretations:

Nested predicate: each of n items has m attributes/tags and the filter checks all of them → O(n·m). If it's also case-insensitive substring search over a string of length k, add that factor → O(n·m·k). For 10k items × 20 tags on every keystroke, that's 200k+ string ops per character typed — enough to make the input feel laggy.
Nested data (tree): rendering a tree/comment-thread recursively costs the total node count N, not the number of roots. People say "I have 100 items" but the tree has 10,000 nodes. Complexity is O(N) in nodes.
The dominating factor: how often does this run?

For a search box, the filter runs on every render, and every keystroke triggers a render. So the real-world cost is renders × O(n·m). That's why the fix is rarely "make the filter asymptotically better" and usually "stop recomputing it":

useMemo(() => items.filter(pred), [items, query]) caches the filtered array so it only recomputes when inputs change — turning re-renders from re-filter (O(n·m)) back to O(1) when only unrelated state changed.
React.memo on <Row/> skips re-rendering rows whose props are unchanged, shrinking the reconciliation constant.
Debounce the query (150–250ms) so we filter a few times per burst of typing, not once per character.
Move the predicate out of the render body so it isn't reallocated each render (which would also bust useMemo/useCallback deps).
When O(n) per render is still too much

Big-O O(n) sounds fine, but at n = 50,000 even a linear build of DOM nodes blows the 16ms frame budget and janks. The answer isn't a better algorithm — it's not rendering n at all: windowing / virtualization (react-window, react-virtual, TanStack Virtual) renders only the ~20 visible rows, making DOM/commit cost O(visible) (effectively O(1) w.r.t. n). Filtering may still be O(n), but you can push that server-side (paginated API) so the client never holds all n.

Space complexity
Filtered array + element tree: O(f) (worst case O(n)).
useMemo cache: O(f) retained between renders (a deliberate space-for-time trade).
React's fiber tree / previous tree for diffing: O(nodes rendered) = O(f) (or O(visible) with virtualization).
Summary — what a strong answer emphasizes

Treat the render as an algorithm: filter cost + build/diff cost + commit cost. Nail down what "nested" means (nested predicate → O(n·m); nested data → O(N) in total nodes; chained filters → still O(n)). State the baseline as O(n) time / O(f) space per render, then pivot to the frontend-specific reality — the cost multiplies by how many times it re-runs, so memoization, React.memo, stable keys, debouncing, and virtualization are the real levers.

#### Common mistakes
saying chained filters are O(n²); ignoring that commit cost is O(Δ) not O(n); forgetting keys make diffing O(f) instead of degenerate; optimizing algorithm when the real fix is "don't recompute" or "don't render all n." Follow-up probes: "Why is an index key bad?", "How would you profile this?" (React Profiler / flame chart, look at commit durations), "At what n do you reach for virtualization?" (roughly when total rendered nodes exceed a few hundred to low thousands).

**Comparison:**

```
Scenario	Time / render	Space	Notes
Flat filter, O(1) predicate	O(n)	O(f)	Baseline; f = survivors
Chained filters a.filter().filter()	O(n)	O(f)	Additive, NOT O(n²)
Nested predicate (m tags/item)	O(n·m)	O(f)	The classic "nested" case
Nested tree data (N total nodes)	O(N)	O(N)	Cost is nodes, not roots
Reconciliation, stable keys	O(f)	O(f)	Index keys degrade this
Commit (DOM mutations)	O(Δ)	O(Δ)	Only what actually changed
+ useMemo	O(n·m) on dep change, else O(1)	O(f) cache	Space-for-time
+ virtualization	O(visible) ≈ O(1) in n	O(visible)	Render only viewport
```

**Code:**

```
JSX
Copy
// n items, "nested" predicate: each item checks m tags (O(n*m)).
// Baseline problem: this filter re-runs on EVERY render (every keystroke).
function List({ items, query }) {
  const q = query.toLowerCase();
  const filtered = items.filter(item =>
    item.tags.some(tag => tag.toLowerCase().includes(q)) // inner loop => O(m*k)
  );                                                       // whole thing O(n*m*k)
  return filtered.map(it => <Row key={it.id} item={it} />); // stable key => O(f) diff
}

// Optimized: memoize the derived data + memoize rows + debounce upstream.
const Row = React.memo(function Row({ item }) {
  return <li>{item.name}</li>;
});

function List({ items, query }) {
  const filtered = React.useMemo(() => {         // recompute only when deps change
    const q = query.toLowerCase();
    return items.filter(item =>
      item.tags.some(tag => tag.toLowerCase().includes(q))
    );
  }, [items, query]);                             // O(n*m) only on real input change

  return (
    <ul>
      {filtered.map(it => <Row key={it.id} item={it} />)}
    </ul>
  );
}

// For very large n: render only visible rows -> O(visible) DOM/commit.
import { FixedSizeList } from 'react-window';
function VirtualList({ filtered }) {
  return (
    <FixedSizeList height={600} itemCount={filtered.length} itemSize={40} width={"100%"}>
      {({ index, style }) => <div style={style}>{filtered[index].name}</div>}
    </FixedSizeList>
  );
}
```

**Example input / output:**

```

Dry run of the per-render cost model (predicate checks m tags each; k = avg tag length):

CODE
Copy
INPUT:  n = 10,000 items, m = 20 tags/item, query typed char-by-char (8 chars)
        no memoization, no debounce

Per keystroke render:
  filter  = n * m substring checks     = 10,000 * 20   = 200,000 ops
  build   = f element objects          (f = survivors) <= 10,000
  diff    = f (stable keys)            <= 10,000
  commit  = delta changed DOM nodes    ~ small

8 keystrokes -> ~8 * 200,000 = 1,600,000 filter ops  => visible input lag

WITH useMemo + debounce(200ms):
  filter runs ~2-3 times total (not 8), each 200,000 ops
  unrelated re-renders skip filtering entirely (cache hit) => O(1)

WITH react-window (n = 50,000):
  build/diff/commit = ~20 visible rows  => O(visible), constant vs n
```

**Flow:**

1. O(n) or O(n*m)
2. stable keys: O(f)
3. useMemo cache hit
4. React.memo skip
5. State/props change
6. Render function runs
7. filter items
8. Filtered array, size f
9. Map to JSX elements: O(f)
10. Reconcile vs prev tree
11. Compute diff Δ
12. Commit to DOM: O(Δ)

