---
type: "practice"
difficulty: "Intermediate"
---

# Design an in-memory LRU cache. What data structures give O(1) get and put?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Two-level Cache Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Compensating_Transactions/Two-level_Cache_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked? Design a fixed-capacity, in-memory Least Recently Used cache exposing two operations — get(key) and put(key, value) — both in O(1) average time. When the cache is full and a new key is inserted, the least recently used entry must be evicted. "Recently used" means: any successful get or any put counts as a use and makes that key the most-recently-used.

What the interviewer is probing for:

Do you know that a single data structure won't give you both O(1) lookup and O(1) recency ordering, and that the trick is to combine two — a hash map + a doubly linked list?
Can you articulate why each structure is chosen (why not a singly linked list, why not an array, why not a plain queue)?
Can you write clean, bug-free node-splicing code (the classic off-by-one / dangling-pointer trap)?
Do you handle the edge cases: capacity 1, updating an existing key, get on a miss, eviction ordering.

Clarifying questions I would ask:

Is capacity fixed at construction? (Assume yes.)
Single-threaded or concurrent access? (Assume single-threaded here; thread-safety is a follow-up.)
Are keys/values arbitrary objects, and can values be null? (Assume non-null values; I use a sentinel for misses.)
Do we need TTL/expiry too? (Assume no — pure LRU.)

Plan: Establish why the two-structure combo is necessary, define the invariant, walk through get/put step by step, then present the code and a dry-run.

First principles: why two structures

Break the requirement into the two things the cache must do fast:

Find a key's value quickly → this screams hash map (O(1) average lookup).
Know and mutate the recency order quickly → we must, on every access, move a key to "most recent," and on eviction, drop the "least recent." That means fast reordering: O(1) removal from the middle and O(1) insertion at an end.

A hash map alone gives (1) but has no ordering. An array/list gives ordering but moving an element to the front is O(n) (shifting). A queue gives O(1) at the ends but you can't cheaply remove an element from the middle when it's re-accessed. The structure that supports O(1) removal of an arbitrary node and O(1) insertion at an end is a doubly linked list, provided you already hold a pointer to the node.

So the design is: hash map + doubly linked list, where the map stores key → node pointer and the list keeps nodes ordered from most-recently-used (head) to least-recently-used (tail).

The map gives O(1) "does this key exist, and where is its node?"
The doubly linked list gives O(1) "unlink this node from wherever it is and move it to the head," and O(1) "evict the tail."

Why doubly and not singly linked? To unlink an arbitrary node in O(1) you need its predecessor. A singly linked list would force an O(n) scan to find the predecessor; a doubly linked list stores prev, so unlinking is constant time.

The invariant and sentinels

I use two sentinel (dummy) nodes, head and tail, that always exist and never hold real data. Real nodes live between them:

CODE
head <-> mostRecent <-> ... <-> leastRecent <-> tail

Sentinels remove all the "is this the first/last node?" branching — every insert/remove touches two real neighbors that are guaranteed to exist. Most recent is right after head; least recent is right before tail.

The operations, step by step

get(key)

Look up the node in the map. Miss → return "not found" (I return a sentinel/-1/null per contract).
Hit → the node was just used, so unlink it from its current position and move it to the front (right after head).
Return its value. Both steps are O(1).

put(key, value)

If the key already exists: update the node's value and move it to the front. Done (no size change).
If it's new: create a node, insert it at the front, add key → node to the map.
If size now exceeds capacity: the node before tail is the LRU victim — unlink it and remove its key from the map (forgetting this is the #1 bug: a stale map entry leaks memory and returns a dangling node later).

Every path is O(1) average: one or two hash operations plus a constant number of pointer splices.

Complexity
Time: get = O(1) average, put = O(1) average (hash-map operations are amortized O(1); linked-list splices are worst-case O(1)).
Space: O(capacity) — one node per stored entry plus its map slot.
Edge cases I explicitly handle
Capacity 1: a second distinct put evicts the first — the sentinel design handles this without special-casing.
Updating an existing key: value changes, size stays, key becomes most recent (not treated as a new insert).
get miss: returns the miss sentinel and does not modify order.
Eviction: always removes tail.prev, and crucially deletes that key from the map.
Real-world framing

This is exactly how Java's LinkedHashMap with accessOrder=true works under the hood — you can override removeEldestEntry to get an LRU in a few lines; interviewers often accept that as the "I know the library" answer but still want the hand-rolled version to prove you understand the mechanics. Production caches (Caffeine, Guava, Redis' allkeys-lru) approximate or extend this — e.g., Redis samples a few keys rather than keeping a strict list, trading exactness for lower memory/overhead, and Caffeine uses the W-TinyLFU admission policy because pure LRU is fooled by a one-time scan that floods the cache with keys never read again ("cache pollution"). Knowing LRU's weakness (scan resistance) is a strong signal of depth.

Summary — what a strong answer emphasizes
The core insight: hash map for O(1) lookup + doubly linked list for O(1) recency reordering; neither alone suffices.
Sentinel head/tail nodes to kill edge-case branching.
On eviction you must remove the key from both the list and the map.
State the complexity (O(1) average both ops, O(capacity) space) and the recency rule (get and put both count as a use).
Bonus depth: mention LinkedHashMap, and LRU's scan-resistance weakness vs. LFU/TinyLFU.

#### Common mistakes
/ follow-up probes: forgetting to delete the evicted key from the map; using a singly linked list (breaks O(1) unlink); treating get as read-only (it must reorder); mishandling the update-existing-key case; and the interviewer's usual next question — "now make it thread-safe" (a separate question).

**Comparison:**

```
Candidate structure	O(1) lookup by key	O(1) move-to-front on access	O(1) evict LRU	Verdict
Hash map only	Yes	No ordering at all	No	Insufficient alone
Array / ArrayList (ordered)	No (O(n) find)	No (O(n) shift)	O(1) at end only	Too slow
Singly linked list + map	Yes (map)	No (O(n) to find prev to unlink)	O(1)	Unlink is O(n)
Hash map + doubly linked list	Yes	Yes	Yes	Chosen
LinkedHashMap(accessOrder=true)	Yes	Yes (built-in)	Yes (removeEldestEntry)	Library shortcut
Operation	Time	Space
get	O(1) average	—
put	O(1) average	—
Whole cache	—	O(capacity)
```

**Code:**

```

import java.util.HashMap; import java.util.Map;

public class LRUCache { // Doubly linked list node private static class Node { int key, value; Node prev, next; Node(int key, int value) { this.key = key; this.value = value; } }

CODE
Copy
private final int capacity;
private final Map<Integer, Node> map = new HashMap<>();
private final Node head = new Node(0, 0); // sentinel: most-recent side
private final Node tail = new Node(0, 0); // sentinel: least-recent side

public LRUCache(int capacity) {
    if (capacity <= 0) throw new IllegalArgumentException("capacity must be > 0");
    this.capacity = capacity;
    head.next = tail;
    tail.prev = head;
}

// Unlink a node from wherever it currently is. O(1).
private void remove(Node node) {
    node.prev.next = node.next;
    node.next.prev = node.prev;
}

// Insert node right after head (== most recently used). O(1).
private void addFront(Node node) {
    node.next = head.next;
    node.prev = head;
    head.next.prev = node;
    head.next = node;
}

public int get(int key) {
    Node node = map.get(key);
    if (node == null) return -1;   // miss sentinel
    remove(node);                  // it was just used ...
    addFront(node);                // ... so make it most recent
    return node.value;
}

public void put(int key, int value) {
    Node existing = map.get(key);
    if (existing != null) {        // update-in-place, no size change
        existing.value = value;
        remove(existing);
        addFront(existing);
        return;
    }
    Node node = new Node(key, value);
    map.put(key, node);
    addFront(node);
    if (map.size() > capacity) {   // evict least recently used
        Node lru = tail.prev;
        remove(lru);
        map.remove(lru.key);       // <-- must clear the map too
    }
}

}

CODE
Copy

```java
// The library shortcut, for completeness:
Map<Integer,Integer> lru = new LinkedHashMap<>(16, 0.75f, true) { // accessOrder=true
    protected boolean removeEldestEntry(Map.Entry<Integer,Integer> e) {
        return size() > CAPACITY;
    }
};
```

**Example input / output:**

```

Dry run with capacity = 2:

CODE
Copy
LRUCache c = new LRUCache(2);

c.put(1, 10);   // list: head <-> (1,10) <-> tail          map={1}
c.put(2, 20);   // list: head <-> (2,20) <-> (1,10) <-> tail   map={1,2}
c.get(1);  -> 10 // 1 used -> move front
                // list: head <-> (1,10) <-> (2,20) <-> tail   map={1,2}
c.put(3, 30);   // new key, over capacity -> evict tail.prev = key 2
                // list: head <-> (3,30) <-> (1,10) <-> tail   map={1,3}
c.get(2);  -> -1 // 2 was evicted (miss)
c.put(1, 11);   // existing key -> update value, move front
                // list: head <-> (1,11) <-> (3,30) <-> tail   map={1,3}
c.get(3);  -> 30

Recency order after the sequence (most -> least recent): 3, 1. Key 2 correctly evicted as the least recently used at the time of the put(3,30).
```

**Flow:**

1. B is LRU, evicted first
2. yes
3. no
4. yes
5. no
6. no
7. yes
8. Structures
9. HashMap: key -> Node*
10. Doubly Linked List (recencyorder)
11. head (sentinel)
12. Node A (MRU)
13. Node B
14. tail (sentinel)
15. put(k,v)
16. key in map?
17. update value + move to front
18. new node -> addFront ->map.put
19. size > capacity?
20. evict tail.prev: remove node +map.remove(key)
21. done
22. get(k)
23. key in map?
24. return -1
25. move node to front, returnvalue

