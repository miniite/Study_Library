---
type: "practice"
difficulty: "Other"
---

# Design a Peer-to-Peer Block Registry

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Heap (Priority Queue)](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Heap_(Priority_Queue).md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are building the coordinator for a peer-to-peer distribution service that shares one large file. The file is split into k equal blocks, numbered from 1 to k. Peers connect to the coordinator, exchange blocks with one another, and later disconnect.

Each connected peer must have a unique numeric handle. When a peer connects, the coordinator assigns it the smallest positive integer that is not currently in use by any connected peer. When a peer disconnects, its handle becomes free and may be handed out again to a future peer.

A peer may ask the coordinator which currently-connected peers hold a particular block. If at least one peer holds it, the download succeeds and the requesting peer now also holds that block (so a later query will list it as an owner too). If nobody holds it, the download fails and the requesting peer does not gain the block.

Implement a class supporting the following operations:

#### Cons
tructor(k) — initialize the coordinator for a file of k blocks.
connect(ownedBlocks) — a new peer joins holding the blocks listed in ownedBlocks (the list may be empty). Assign and return its handle: the smallest positive integer not currently in use.
disconnect(handle) — the peer with the given handle leaves; its blocks are no longer available to anyone, and its handle is freed so it can be reassigned later.
fetch(handle, blockId) — the peer identified by handle requests blockId. Return the list of handles of all currently-connected peers that hold that block, sorted in ascending order. If that list is non-empty, the requesting peer also acquires the block.

You may assume every handle passed to disconnect/fetch belongs to a peer that is currently connected (i.e., the caller uses the handles you returned), and every disconnect is paired with an earlier connect.

#### Cons
traints:

1 <= k <= 10^5
0 <= ownedBlocks.length <= min(100, k)
1 <= ownedBlocks[i] <= k, and the values in ownedBlocks are distinct.
1 <= blockId <= k
At most 10^4 calls in total are made to connect, disconnect, and fetch.

#### Example
s & test cases

Input:

CODE
BlockRegistry(4)
connect([1,2])  -> 1        (smallest free handle is 1)
connect([2,3])  -> 2        (1 is taken, so 2)
connect([4])    -> 3
fetch(1, 3)     -> [2]      (only peer 2 holds block 3; peer 1 now holds it too)
fetch(2, 2)     -> [1,2]    (peers 1 and 2 both hold block 2)
disconnect(1)               (handle 1 is freed; peer 1's blocks gone)
fetch(2, 1)     -> []       (nobody holds block 1 now -> download fails)
disconnect(2)               (handle 2 is freed)
connect([])     -> 1        (handles 1 and 2 are free; smallest is 1)

Output: [null, 1, 2, 3, [2], [1,2], null, [], null, 1] Why: exercises handle reuse (lowest freed handle wins), successful fetch granting ownership, and a failed fetch returning an empty list.

Input:

CODE
BlockRegistry(3)
connect([])   -> 1
fetch(1, 2)   -> []

Output: [null, 1, []] Why: edge case — a peer connecting with no blocks still gets a handle; a fetch for a block nobody owns returns an empty list and the requester gains nothing.

Input:

CODE
BlockRegistry(2)
connect([1])  -> 1
connect([1])  -> 2
fetch(2, 1)   -> [1,2]

Output: [null, 1, 2, [1,2]] Why: multiple owners of the same block are all returned in ascending order; peer 2 already owned block 1, so it appears in its own result.

Input:

CODE
BlockRegistry(2)
connect([2])  -> 1
disconnect(1)
connect([])   -> 1
fetch(1, 2)   -> []

Output: [null, 1, null, 1, []] Why: after the only owner disconnects, the block is unavailable; the freed handle 1 is reused by the next peer.

#### Approach

This is a bookkeeping / design problem combining three data structures. (1) Handle allocation must always return the smallest unused positive integer with O(log) reuse: keep a counter nextHandle for never-yet-issued handles and a min-heap of handles freed by disconnect; on connect, pop the heap if it is non-empty, otherwise take the counter — this always gives the smallest available. (2) Two hash maps form a bidirectional index: blocksOfPeer[handle] (so disconnect can find every block a leaving peer held) and ownersOfBlock[blockId] (so fetch can list owners). Keeping ownersOfBlock values as an ordered set (TreeSet / std::set / sorted-on-read) makes fetch's ascending output free. (3) A successful fetch mutates state — the requesting peer becomes an owner of that block — so both indexes are updated before returning. The pattern is classic "maintain inverse indexes for O(1)/O(log) membership + a free-list (min-heap) for id recycling."

Complexity

Time: connect is O(c log F) where c = number of blocks the peer brings and F = size of the freed-handle heap (one heap pop plus c ordered-set inserts); disconnect is O(c') where c' = blocks the peer held, plus O(log F) to push the freed handle; fetch is O(o) using an ordered set that already yields owners in ascending order (or O(o log o) if you sort a plain list), where o = number of owners of that block. Space: O(P + T) where P = number of connected peers and T = total number of (peer, block) ownership pairs currently recorded — one entry per ownership, indexed both by peer and by block.

Python
PYTHON
import heapq
from typing import List

class BlockRegistry:
    def __init__(self, k: int):
        self.total_blocks = k
        self.next_handle = 1
        self.freed = []                      # min-heap of reusable handles
        self.owners_of_block = {}            # blockId -> set of owner handles
        self.blocks_of_peer = {}             # handle  -> set of blocks held

    def connect(self, owned_blocks: List[int]) -> int:
        handle = heapq.heappop(self.freed) if self.freed else self._take_next()
        self.blocks_of_peer[handle] = set()
        for b in owned_blocks:
            self.owners_of_block.setdefault(b, set()).add(handle)
            self.blocks_of_peer[handle].add(b)
        return handle

    def _take_next(self) -> int:
        h = self.next_handle
        self.next_handle += 1
        return h

    def disconnect(self, handle: int) -> None:
        blocks = self.blocks_of_peer.pop(handle, None)
        if blocks is None:
            return
        for b in blocks:
            owners = self.owners_of_block.get(b)
            if owners is not None:
                owners.discard(handle)
        heapq.heappush(self.freed, handle)

    def fetch(self, handle: int, block_id: int) -> List[int]:
        owners = self.owners_of_block.get(block_id)
        if not owners:
            return []
        result = sorted(owners)              # ascending list of current owners
        owners.add(handle)                   # requester acquires the block
        self.blocks_of_peer.setdefault(handle, set()).add(block_id)
        return result
JavaScript
JAVASCRIPT
class MinHeap {

#### cons
tructor() { this.a = []; }
    get size() { return this.a.length; }
    push(x) {

#### cons
t a = this.a; a.push(x); let i = a.length - 1;
        while (i > 0) { const p = (i - 1) >> 1; if (a[p] <= a[i]) break;
            [a[p], a[i]] = [a[i], a[p]]; i = p; }
    }
    pop() {

#### cons
t a = this.a, top = a[0], last = a.pop();
        if (a.length) { a[0] = last; let i = 0; const n = a.length;
            while (true) { let l = 2*i+1, r = 2*i+2, s = i;
                if (l < n && a[l] < a[s]) s = l;
                if (r < n && a[r] < a[s]) s = r;
                if (s === i) break; [a[s], a[i]] = [a[i], a[s]]; i = s; } }
        return top;
    }
}

class BlockRegistry {

#### cons
tructor(k) {
        this.totalBlocks = k;
        this.nextHandle = 1;
        this.freed = new MinHeap();                 // reusable handles
        this.ownersOfBlock = new Map();             // blockId -> Set of owner handles
        this.blocksOfPeer = new Map();              // handle  -> Set of blocks held
    }
    connect(ownedBlocks) {

#### cons
t handle = this.freed.size ? this.freed.pop() : this.nextHandle++;
        this.blocksOfPeer.set(handle, new Set());
        for (const b of ownedBlocks) {
            if (!this.ownersOfBlock.has(b)) this.ownersOfBlock.set(b, new Set());
            this.ownersOfBlock.get(b).add(handle);
            this.blocksOfPeer.get(handle).add(b);
        }
        return handle;
    }
    disconnect(handle) {

#### cons
t blocks = this.blocksOfPeer.get(handle);
        if (blocks === undefined) return;
        for (const b of blocks) {

#### cons
t owners = this.ownersOfBlock.get(b);
            if (owners) owners.delete(handle);
        }
        this.blocksOfPeer.delete(handle);
        this.freed.push(handle);
    }
    fetch(handle, blockId) {

#### cons
t owners = this.ownersOfBlock.get(blockId);
        if (!owners || owners.size === 0) return [];

#### cons
t result = [...owners].sort((x, y) => x - y);   // ascending
        owners.add(handle);                                 // requester acquires block
        if (!this.blocksOfPeer.has(handle)) this.blocksOfPeer.set(handle, new Set());
        this.blocksOfPeer.get(handle).add(blockId);
        return result;
    }
}
Java
JAVA
import java.util.*;

class BlockRegistry {
    private final int totalBlocks;
    private int nextHandle = 1;
    private final PriorityQueue<Integer> freed = new PriorityQueue<>();      // reusable handles
    private final Map<Integer, TreeSet<Integer>> ownersOfBlock = new HashMap<>();
    private final Map<Integer, Set<Integer>> blocksOfPeer = new HashMap<>();

    public BlockRegistry(int k) { this.totalBlocks = k; }

    public int connect(int[] ownedBlocks) {
        int handle = freed.isEmpty() ? nextHandle++ : freed.poll();
        blocksOfPeer.putIfAbsent(handle, new HashSet<>());
        for (int b : ownedBlocks) {
            ownersOfBlock.computeIfAbsent(b, x -> new TreeSet<>()).add(handle);
            blocksOfPeer.get(handle).add(b);
        }
        return handle;
    }
    public void disconnect(int handle) {
        Set<Integer> blocks = blocksOfPeer.remove(handle);
        if (blocks == null) return;
        for (int b : blocks) {
            TreeSet<Integer> owners = ownersOfBlock.get(b);
            if (owners != null) owners.remove(handle);
        }
        freed.offer(handle);
    }
    public int[] fetch(int handle, int blockId) {
        TreeSet<Integer> owners = ownersOfBlock.get(blockId);
        if (owners == null || owners.isEmpty()) return new int[0];
        int[] res = new int[owners.size()];
        int i = 0;
        for (int h : owners) res[i++] = h;                 // TreeSet iterates ascending
        owners.add(handle);                                // requester acquires block
        blocksOfPeer.computeIfAbsent(handle, x -> new HashSet<>()).add(blockId);
        return res;
    }
}
C++
CPP
#include <vector>
#include <unordered_map>
#include <unordered_set>
#include <set>
#include <queue>
using namespace std;

class BlockRegistry {
    int totalBlocks;
    int nextHandle = 1;
    priority_queue<int, vector<int>, greater<int>> freed;   // min-heap of reusable handles
    unordered_map<int, set<int>> ownersOfBlock;             // blockId -> sorted owner handles
    unordered_map<int, unordered_set<int>> blocksOfPeer;    // handle  -> blocks held
public:
    BlockRegistry(int k) : totalBlocks(k) {}

    int connect(vector<int>& ownedBlocks) {
        int handle;
        if (freed.empty()) handle = nextHandle++;
        else { handle = freed.top(); freed.pop(); }
        blocksOfPeer[handle];                                // ensure the entry exists
        for (int b : ownedBlocks) {
            ownersOfBlock[b].insert(handle);
            blocksOfPeer[handle].insert(b);
        }
        return handle;
    }
    void disconnect(int handle) {
        auto it = blocksOfPeer.find(handle);
        if (it == blocksOfPeer.end()) return;
        for (int b : it->second) ownersOfBlock[b].erase(handle);
        blocksOfPeer.erase(it);
        freed.push(handle);
    }
    vector<int> fetch(int handle, int blockId) {
        auto it = ownersOfBlock.find(blockId);
        if (it == ownersOfBlock.end() || it->second.empty()) return {};
        vector<int> res(it->second.begin(), it->second.end());   // already ascending
        it->second.insert(handle);                               // requester acquires block
        blocksOfPeer[handle].insert(blockId);
        return res;
    }
};
C
C
#include <stdio.h>
#include <stdlib.h>

/* growable int vector */
typedef struct { int *data; int size, cap; } Vec;
static void vecInit(Vec *v) { v->data = NULL; v->size = 0; v->cap = 0; }
static void vecPush(Vec *v, int x) {
    if (v->size == v->cap) { v->cap = v->cap ? v->cap * 2 : 4;
        v->data = realloc(v->data, v->cap * sizeof(int)); }
    v->data[v->size++] = x;
}
static void vecRemoveValue(Vec *v, int x) {                 /* swap-remove */
    for (int i = 0; i < v->size; i++)
        if (v->data[i] == x) { v->data[i] = v->data[--v->size]; return; }
}
static int vecContains(Vec *v, int x) {
    for (int i = 0; i < v->size; i++) if (v->data[i] == x) return 1;
    return 0;
}

/* min-heap of freed handles */
typedef struct { int *h; int size, cap; } MinHeap;
static void heapInit(MinHeap *hp) { hp->h = NULL; hp->size = 0; hp->cap = 0; }
static void heapPush(MinHeap *hp, int x) {
    if (hp->size == hp->cap) { hp->cap = hp->cap ? hp->cap * 2 : 4;
        hp->h = realloc(hp->h, hp->cap * sizeof(int)); }
    int i = hp->size++; hp->h[i] = x;
    while (i > 0) { int p = (i - 1) / 2; if (hp->h[p] <= hp->h[i]) break;
        int t = hp->h[p]; hp->h[p] = hp->h[i]; hp->h[i] = t; i = p; }
}
static int heapPop(MinHeap *hp) {
    int top = hp->h[0]; hp->h[0] = hp->h[--hp->size]; int i = 0;
    while (1) { int l = 2*i+1, r = 2*i+2, s = i;
        if (l < hp->size && hp->h[l] < hp->h[s]) s = l;
        if (r < hp->size && hp->h[r] < hp->h[s]) s = r;
        if (s == i) break;
        int t = hp->h[s]; hp->h[s] = hp->h[i]; hp->h[i] = t; i = s; }
    return top;
}

typedef struct {
    int totalBlocks, nextHandle, peerCap;
    MinHeap freed;
    Vec *ownersOfBlock;   /* index by blockId, 1..totalBlocks */
    Vec *blocksOfPeer;    /* index by handle */
} BlockRegistry;

BlockRegistry *registryCreate(int k) {
    BlockRegistry *r = malloc(sizeof(BlockRegistry));
    r->totalBlocks = k; r->nextHandle = 1; heapInit(&r->freed);
    r->ownersOfBlock = malloc((k + 1) * sizeof(Vec));
    for (int i = 0; i <= k; i++) vecInit(&r->ownersOfBlock[i]);
    r->peerCap = 16;
    r->blocksOfPeer = malloc(r->peerCap * sizeof(Vec));
    for (int i = 0; i < r->peerCap; i++) vecInit(&r->blocksOfPeer[i]);
    return r;
}
static void ensurePeerCap(BlockRegistry *r, int handle) {
    if (handle < r->peerCap) return;
    int old = r->peerCap;
    while (handle >= r->peerCap) r->peerCap *= 2;
    r->blocksOfPeer = realloc(r->blocksOfPeer, r->peerCap * sizeof(Vec));
    for (int i = old; i < r->peerCap; i++) vecInit(&r->blocksOfPeer[i]);
}

int registryConnect(BlockRegistry *r, int *ownedBlocks, int ownedSize) {
    int handle = r->freed.size ? heapPop(&r->freed) : r->nextHandle++;
    ensurePeerCap(r, handle);
    r->blocksOfPeer[handle].size = 0;                 /* reuse any prior storage */
    for (int i = 0; i < ownedSize; i++) {
        int b = ownedBlocks[i];
        vecPush(&r->ownersOfBlock[b], handle);
        vecPush(&r->blocksOfPeer[handle], b);
    }
    return handle;
}
void registryDisconnect(BlockRegistry *r, int handle) {
    Vec *blocks = &r->blocksOfPeer[handle];
    for (int i = 0; i < blocks->size; i++)
        vecRemoveValue(&r->ownersOfBlock[blocks->data[i]], handle);
    blocks->size = 0;
    heapPush(&r->freed, handle);
}
static int cmpInt(const void *a, const void *b) { return (*(int*)a) - (*(int*)b); }

int *registryFetch(BlockRegistry *r, int handle, int blockId, int *returnSize) {
    Vec *owners = &r->ownersOfBlock[blockId];
    if (owners->size == 0) { *returnSize = 0; return NULL; }
    int n = owners->size;
    int *res = malloc(n * sizeof(int));
    for (int i = 0; i < n; i++) res[i] = owners->data[i];
    qsort(res, n, sizeof(int), cmpInt);
    if (!vecContains(owners, handle)) {               /* requester acquires block */
        vecPush(owners, handle);
        ensurePeerCap(r, handle);
        vecPush(&r->blocksOfPeer[handle], blockId);
    }
    *returnSize = n;
    return res;
}

#### Follow-up
questions
What if peers are identified by their network address instead of an assigned handle, and the same address disconnects and reconnects repeatedly?
If peers connect and disconnect frequently but never call fetch, is your design still efficient? (Yes — no owner scans happen; only heap and map updates.)
If every peer connects once, fetches every block, then disconnects, does it stay efficient, and where is the cost concentrated?
Extend the coordinator to serve n distinct files, where file i has k[i] blocks. What changes — key blocks by (fileId, blockId), or run one registry per file?

