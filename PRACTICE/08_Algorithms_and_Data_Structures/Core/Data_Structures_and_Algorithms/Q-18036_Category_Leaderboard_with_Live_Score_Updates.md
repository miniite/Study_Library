---
type: "practice"
difficulty: "Other"
---

# Category Leaderboard with Live Score Updates

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Heap (Priority Queue)](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Heap_(Priority_Queue).md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Design a data structure that tracks scored items grouped into categories and can always report the current leader of any category, even as scores change over time.

Each item has a unique name, belongs to exactly one category, and carries an integer score. The structure must support two live operations: changing an item's score, and querying the top item of a category.

Implement a class CategoryLeaderboard with the following API:

CategoryLeaderboard(products, categories, scores) — initializes the leaderboard from three arrays of equal length n. For each index i, products[i] is the (unique) name of an item, categories[i] is the category it belongs to, and scores[i] is its initial score.
updateScore(product, newScore) — sets the score of the item named product to newScore. The item is guaranteed to already exist in the leaderboard; its category never changes.
topProduct(category) — returns the name of the item in category that currently has the highest score. If two or more items in that category are tied for the highest score, return the one whose name is lexicographically smallest. The queried category is guaranteed to contain at least one item.

A string x is lexicographically smaller than a string y if x comes first in dictionary order, that is, either x is a prefix of y, or at the first position i where they differ, x[i] precedes y[i] in the alphabet.

#### Cons
traints

1 <= n <= 2 * 10^4
n == products.length == categories.length == scores.length
1 <= products[i].length, categories[i].length <= 10
products[i] and categories[i] consist of lowercase English letters.
1 <= scores[i] <= 10^8
All names in products are distinct.
Every product passed to updateScore is an existing item name.
Every category passed to topProduct contains at least one item.
At most 2 * 10^4 calls in total are made to updateScore and topProduct.

#### Example
s & test cases
TEXT
Input:
  CategoryLeaderboard(["aria","nimbus","cobalt","dune","ember","flint"], ["audio","video","video","photo","video","audio"], [9,12,8,15,14,7])
  topProduct("audio")        -> "aria"
  topProduct("video")        -> "ember"
  updateScore("cobalt", 16)
  topProduct("video")        -> "cobalt"
  updateScore("ember", 16)
  topProduct("video")        -> "cobalt"
Why: In "audio" the scores are aria=9, flint=7, so aria leads. In "video" ember=14 beats nimbus=12 and cobalt=8. After cobalt rises to 16 it becomes the top "video" item. After ember also rises to 16 both cobalt and ember are tied at 16, so the lexicographically smaller name "cobalt" wins.

Input:
  CategoryLeaderboard(["zeta","alpha"], ["book","book"], [5,5])
  topProduct("book")         -> "alpha"
Why: Both items start tied at score 5, so the tie is broken by name; "alpha" is lexicographically smaller than "zeta".

Input:
  CategoryLeaderboard(["solo"], ["misc"], [1])
  topProduct("misc")         -> "solo"
Why: Minimum case — a single item in a single category is trivially its own leader.

Input:
  CategoryLeaderboard(["red","blue"], ["hue","hue"], [10,4])
  topProduct("hue")          -> "red"
  updateScore("red", 1)
  topProduct("hue")          -> "blue"
Why: Initially red=10 leads blue=4. Lowering red to 1 drops it below blue=4, so blue overtakes and becomes the new leader — verifying that score decreases are handled, not just increases.

#### Approach

Keep three maps: each item's current score, each item's category, and one max-heap per category ordered by (score descending, name ascending) so the heap top is exactly the desired leader. A binary heap can't cheaply update the priority of an existing element, so use lazy deletion: on updateScore, just push a brand-new (newScore, name) entry and leave any older entry for that item in place. On topProduct, look at the heap top and compare its stored score against the item's current score in the map; if they differ, this entry is stale (superseded by a later push), so pop it and continue. The first entry whose stored score still equals the item's current score is the true current leader. Ties are resolved by the comparator preferring the lexicographically smaller name. This is the classic "heap + lazy deletion" design pattern; an ordered set / balanced BST keyed by (score, name) per category is an alternative giving true O(log n) updates and deletes.

Complexity

Let n be the initial item count and U the number of updateScore calls. Construction is O(n) with heapify (O(n log n) if items are pushed one by one). updateScore is O(log m) to push into the category heap of size m. topProduct is O((1 + d) log m) where d is the number of stale entries discarded; since every entry is pushed once and popped at most once across all queries, the total stale-pop work is bounded by O((n + U) log m), i.e. amortized O(log m) per query. Space is O(n + U) for the heap entries plus O(n) for the maps.

Python
PYTHON
import heapq
from typing import List

class CategoryLeaderboard:
    def __init__(self, products: List[str], categories: List[str], scores: List[int]):
        self.score = {}      # product -> current score
        self.cat_of = {}     # product -> category
        self.heaps = {}      # category -> min-heap of (-score, name)
        for product, category, sc in zip(products, categories, scores):
            self.score[product] = sc
            self.cat_of[product] = category
            self.heaps.setdefault(category, []).append((-sc, product))
        for heap in self.heaps.values():
            heapq.heapify(heap)

    def update_score(self, product: str, new_score: int) -> None:
        self.score[product] = new_score
        heapq.heappush(self.heaps[self.cat_of[product]], (-new_score, product))

    def top_product(self, category: str) -> str:
        heap = self.heaps[category]
        # (-score, name): smallest tuple = highest score, then smallest name
        while heap:
            neg_score, name = heap[0]
            if -neg_score == self.score[name]:   # entry still current
                return name
            heapq.heappop(heap)                  # stale entry, drop it
        return ""
JavaScript
JAVASCRIPT
class MaxHeap {

#### cons
tructor() { this.a = []; }
    // higher score first; tie -> smaller name first
    _better(x, y) { return x[0] !== y[0] ? x[0] > y[0] : x[1] < y[1]; }
    push(v) {

#### cons
t a = this.a; a.push(v);
        let i = a.length - 1;
        while (i > 0) {

#### cons
t p = (i - 1) >> 1;
            if (this._better(a[i], a[p])) { [a[i], a[p]] = [a[p], a[i]]; i = p; }
            else break;
        }
    }
    top() { return this.a[0]; }
    pop() {

#### cons
t a = this.a, last = a.pop();
        if (a.length) {
            a[0] = last;
            let i = 0;
            for (;;) {

#### cons
t l = 2 * i + 1, r = 2 * i + 2;
                let best = i;
                if (l < a.length && this._better(a[l], a[best])) best = l;
                if (r < a.length && this._better(a[r], a[best])) best = r;
                if (best === i) break;
                [a[i], a[best]] = [a[best], a[i]]; i = best;
            }
        }
    }
    get size() { return this.a.length; }
}

class CategoryLeaderboard {

#### cons
tructor(products, categories, scores) {
        this.score = new Map();   // product -> current score
        this.catOf = new Map();   // product -> category
        this.heaps = new Map();   // category -> lazy max-heap
        for (let i = 0; i < products.length; i++) {
            this.score.set(products[i], scores[i]);
            this.catOf.set(products[i], categories[i]);
            if (!this.heaps.has(categories[i])) this.heaps.set(categories[i], new MaxHeap());
            this.heaps.get(categories[i]).push([scores[i], products[i]]);
        }
    }

    updateScore(product, newScore) {
        this.score.set(product, newScore);
        this.heaps.get(this.catOf.get(product)).push([newScore, product]);
    }

    topProduct(category) {

#### cons
t h = this.heaps.get(category);
        while (h.size) {

#### cons
t [s, name] = h.top();
            if (this.score.get(name) === s) return name;   // still valid
            h.pop();                                        // stale, discard
        }
        return "";
    }
}
Java
JAVA
import java.util.*;

class CategoryLeaderboard {
    private final Map<String, Integer> score = new HashMap<>();   // product -> current score
    private final Map<String, String> catOf = new HashMap<>();    // product -> category
    private final Map<String, PriorityQueue<Entry>> heaps = new HashMap<>();

    private static final class Entry {
        final int score;
        final String name;
        Entry(int score, String name) { this.score = score; this.name = name; }
    }

    // highest score first; on a tie, lexicographically smaller name first
    private static final Comparator<Entry> ORDER = (a, b) ->
        a.score != b.score ? Integer.compare(b.score, a.score) : a.name.compareTo(b.name);

    public CategoryLeaderboard(String[] products, String[] categories, int[] scores) {
        for (int i = 0; i < products.length; i++) {
            score.put(products[i], scores[i]);
            catOf.put(products[i], categories[i]);
            heaps.computeIfAbsent(categories[i], c -> new PriorityQueue<>(ORDER))
                 .add(new Entry(scores[i], products[i]));
        }
    }

    public void updateScore(String product, int newScore) {
        score.put(product, newScore);
        heaps.get(catOf.get(product)).add(new Entry(newScore, product));  // lazy insert
    }

    public String topProduct(String category) {
        PriorityQueue<Entry> pq = heaps.get(category);
        while (!pq.isEmpty()) {
            Entry top = pq.peek();
            if (score.get(top.name).intValue() == top.score) return top.name;  // valid
            pq.poll();                                                          // stale
        }
        return "";
    }
}
C++
CPP
#include <string>
#include <vector>
#include <queue>
#include <unordered_map>
using namespace std;

class CategoryLeaderboard {
    struct Cmp {
        // top of heap = highest score; on a tie, the lexicographically smaller name
        bool operator()(const pair<int, string>& a, const pair<int, string>& b) const {
            if (a.first != b.first) return a.first < b.first;
            return a.second > b.second;
        }
    };
    using PQ = priority_queue<pair<int, string>, vector<pair<int, string>>, Cmp>;

    unordered_map<string, int> score;       // product -> current score
    unordered_map<string, string> catOf;    // product -> category
    unordered_map<string, PQ> heaps;        // category -> lazy max-heap

public:
    CategoryLeaderboard(vector<string>& products, vector<string>& categories, vector<int>& scores) {
        for (size_t i = 0; i < products.size(); ++i) {
            score[products[i]] = scores[i];
            catOf[products[i]] = categories[i];
            heaps[categories[i]].push({scores[i], products[i]});
        }
    }

    void updateScore(const string& product, int newScore) {
        score[product] = newScore;
        heaps[catOf[product]].push({newScore, product});   // lazy: old entry left behind
    }

    string topProduct(const string& category) {
        auto& pq = heaps[category];
        while (!pq.empty()) {
            auto [s, name] = pq.top();
            if (score[name] == s) return name;             // entry still reflects current score
            pq.pop();                                      // stale entry, discard
        }
        return "";
    }
};
C
C
#include <stdlib.h>
#include <string.h>

/* ---- open-addressing string -> int hash map (fixed capacity, no rehash) ---- */
typedef struct { char **keys; int *vals; int cap, size; } SMap;

static unsigned long sm_hash(const char *s) {
    unsigned long h = 1469598103934665603UL;
    while (*s) { h ^= (unsigned char)*s++; h *= 1099511628211UL; }
    return h;
}
static void sm_init(SMap *m, int cap) {
    m->cap = cap; m->size = 0;
    m->keys = (char **)calloc((size_t)cap, sizeof(char *));
    m->vals = (int *)malloc(sizeof(int) * (size_t)cap);
}
static int sm_slot(SMap *m, const char *k) {
    int i = (int)(sm_hash(k) % (unsigned long)m->cap);
    while (m->keys[i] && strcmp(m->keys[i], k) != 0) i = (i + 1) % m->cap;
    return i;
}
static void sm_put(SMap *m, const char *k, int v) {
    int i = sm_slot(m, k);
    if (!m->keys[i]) { m->keys[i] = strdup(k); m->size++; }
    m->vals[i] = v;
}
static int sm_get(SMap *m, const char *k) {  /* -1 if absent */
    int i = sm_slot(m, k);
    return m->keys[i] ? m->vals[i] : -1;
}

/* ---- max-heap of (score, name): higher score first, then smaller name ---- */
typedef struct { int score; char *name; } HNode;
typedef struct { HNode *a; int size, cap; } Heap;

static int h_better(HNode x, HNode y) {
    if (x.score != y.score) return x.score > y.score;
    return strcmp(x.name, y.name) < 0;
}
static void h_push(Heap *h, HNode v) {
    if (h->size == h->cap) {
        h->cap = h->cap ? h->cap * 2 : 4;
        h->a = (HNode *)realloc(h->a, sizeof(HNode) * (size_t)h->cap);
    }
    int i = h->size++;
    h->a[i] = v;
    while (i > 0) {
        int p = (i - 1) / 2;
        if (h_better(h->a[i], h->a[p])) { HNode t = h->a[i]; h->a[i] = h->a[p]; h->a[p] = t; i = p; }
        else break;
    }
}
static void h_pop(Heap *h) {
    h->a[0] = h->a[--h->size];
    int i = 0;
    while (1) {
        int l = 2 * i + 1, r = 2 * i + 2, best = i;
        if (l < h->size && h_better(h->a[l], h->a[best])) best = l;
        if (r < h->size && h_better(h->a[r], h->a[best])) best = r;
        if (best == i) break;
        HNode t = h->a[i]; h->a[i] = h->a[best]; h->a[best] = t; i = best;
    }
}

typedef struct {
    SMap score;   /* product name -> current score */
    SMap cat;     /* product name -> category id   */
    SMap catId;   /* category name -> id           */
    Heap *heaps;  /* one lazy max-heap per category */
    int numCat;
} CategoryLeaderboard;

CategoryLeaderboard *clCreate(char **products, int productsSize,
                              char **categories, int categoriesSize,
                              int *scores, int scoresSize) {
    CategoryLeaderboard *o = (CategoryLeaderboard *)malloc(sizeof(CategoryLeaderboard));
    int cap = productsSize * 2 + 16;               /* load factor <= 0.5 */
    sm_init(&o->score, cap);
    sm_init(&o->cat, cap);
    sm_init(&o->catId, cap);
    o->numCat = 0;
    o->heaps = (Heap *)calloc((size_t)productsSize + 1, sizeof(Heap));
    for (int i = 0; i < productsSize; i++) {
        int cid = sm_get(&o->catId, categories[i]);
        if (cid < 0) { cid = o->numCat++; sm_put(&o->catId, categories[i], cid); }
        sm_put(&o->score, products[i], scores[i]);
        sm_put(&o->cat, products[i], cid);
        int slot = sm_slot(&o->score, products[i]);
        HNode node = { scores[i], o->score.keys[slot] };  /* stable key pointer */
        h_push(&o->heaps[cid], node);
    }
    return o;
}

void clUpdateScore(CategoryLeaderboard *o, char *product, int newScore) {
    sm_put(&o->score, product, newScore);
    int cid = sm_get(&o->cat, product);
    int slot = sm_slot(&o->score, product);
    HNode node = { newScore, o->score.keys[slot] };
    h_push(&o->heaps[cid], node);
}

char *clTopProduct(CategoryLeaderboard *o, char *category) {
    int cid = sm_get(&o->catId, category);
    Heap *h = &o->heaps[cid];
    while (h->size > 0) {
        HNode t = h->a[0];
        if (sm_get(&o->score, t.name) == t.score) return t.name;  /* still valid */
        h_pop(h);                                                 /* stale -> drop */
    }
    return NULL;
}

void clFree(CategoryLeaderboard *o) {
    for (int i = 0; i < o->numCat; i++) free(o->heaps[i].a);
    free(o->heaps);
    for (int i = 0; i < o->score.cap; i++) if (o->score.keys[i]) free(o->score.keys[i]);
    for (int i = 0; i < o->cat.cap; i++)   if (o->cat.keys[i])   free(o->cat.keys[i]);
    for (int i = 0; i < o->catId.cap; i++) if (o->catId.keys[i]) free(o->catId.keys[i]);
    free(o->score.keys); free(o->score.vals);
    free(o->cat.keys);   free(o->cat.vals);
    free(o->catId.keys); free(o->catId.vals);
    free(o);
}

#### Follow-up
questions
Add a bottomProduct(category) that returns the lowest-scored item; maintain a second (min) heap per category or switch to an ordered set.
Support removing an item entirely from the leaderboard — extend the lazy-deletion validity check with a tombstone set.
If updateScore is extremely frequent and topProduct is rare, how does an ordered set (TreeMap/std::set) keyed by (score, name) compare, and when is the lazy heap actually better?
Change the tie-break rule to "highest score, then most recently updated" and describe how the stored key must change.

