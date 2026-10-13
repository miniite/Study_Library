---
type: "practice"
difficulty: "Other"
---

# Merge Multiple Sorted Linked Chains into One

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Heap (Priority Queue)](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Heap_(Priority_Queue).md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given an array chains of length k. Each element chains[i] is the head node of a singly linked list whose values are stored in non-decreasing (ascending) order. Some of the chains may be empty (represented by a null head).

Combine every node from all of the chains into a single linked list that is also sorted in non-decreasing order, and return the head of that merged list. You must splice the existing nodes together (rearrange pointers) rather than allocating a brand-new value for every node.

Input

chains: an array of k sorted linked-list heads. Any head may be null (an empty chain), and the whole array may be empty.

Output

The head of one linked list containing all nodes from all input chains, sorted in ascending order. Return null if there are no nodes at all.

#### Cons
traints

k == chains.length
0 <= k <= 10^4
0 <= chains[i].length <= 500
-10^4 <= node value <= 10^4
Each chains[i] is individually sorted in ascending order.
The total number of nodes across all chains does not exceed 10^4.

#### Example
s & test cases
TEXT
Input: chains = [[2,7,9],[2,5,7],[3,8]] -> Output: [2,2,3,5,7,7,8,9]
Why: Standard case; values interleave and duplicates appear across different chains.

Input: chains = [] -> Output: []
Why: No chains at all, so the merged list is empty (null head).

Input: chains = [[]] -> Output: []
Why: A single chain that is itself empty must still yield an empty result.

Input: chains = [[-4,-1,0],[-5,2]] -> Output: [-5,-4,-1,0,2]
Why: Negative values must be ordered correctly alongside non-negatives.

Input: chains = [[6,10],[],[1,4,4]] -> Output: [1,4,4,6,10]
Why: An empty chain sits in the middle and must be skipped without breaking the merge.

Input: chains = [[1],[1],[1]] -> Output: [1,1,1]
Why: Many single-node chains with equal values; the heap must handle ties cleanly.

#### Approach

This is a k-way merge. Keep a min-heap holding the current front node of each chain (initially the k heads). Repeatedly pop the globally smallest node, append it to the result tail, and push that node's successor (if any) back into the heap. Because the heap always contains at most one node per chain, its size stays bounded by k, giving O(log k) per extraction. An equivalent optimal alternative is pairwise divide-and-conquer merging (merge chains two at a time in log k rounds), which also runs in O(N log k); the heap version is used here for a single consistent implementation across languages.

Complexity

Time: O(N log k), where N is the total number of nodes and k is the number of chains. Each of the N nodes is pushed and popped from a heap that never holds more than k elements, so each heap operation costs O(log k). Space: O(k) for the heap (one live node per non-empty chain); the output reuses the input nodes, so no extra list allocation.

Python
PYTHON
import heapq

class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

def mergeKLists(chains):
    heap = []  # entries: (value, insertion_index, node) — index breaks val ties
    for i, node in enumerate(chains):
        if node:
            heapq.heappush(heap, (node.val, i, node))

    dummy = ListNode(0)
    curr = dummy
    while heap:
        _, i, node = heapq.heappop(heap)
        if node.next:
            heapq.heappush(heap, (node.next.val, i, node.next))
        curr.next = node
        curr = node
    return dummy.next
JavaScript
JAVASCRIPT
class ListNode {

#### cons
tructor(val = 0, next = null) {
        this.val = val;
        this.next = next;
    }
}

function mergeKLists(chains) {

#### cons
t heap = []; // binary min-heap of ListNode, keyed by val

#### cons
t swap = (i, j) => { const t = heap[i]; heap[i] = heap[j]; heap[j] = t; };

#### cons
t push = (node) => {
        heap.push(node);
        let i = heap.length - 1;
        while (i > 0) {

#### cons
t p = (i - 1) >> 1;
            if (heap[p].val <= heap[i].val) break;
            swap(p, i);
            i = p;
        }
    };

#### cons
t pop = () => {

#### cons
t top = heap[0];

#### cons
t last = heap.pop();
        if (heap.length) {
            heap[0] = last;
            let i = 0;
            while (true) {

#### cons
t l = 2 * i + 1, r = 2 * i + 2;
                let s = i;
                if (l < heap.length && heap[l].val < heap[s].val) s = l;
                if (r < heap.length && heap[r].val < heap[s].val) s = r;
                if (s === i) break;
                swap(s, i);
                i = s;
            }
        }
        return top;
    };

    for (const node of chains) if (node) push(node);

#### cons
t dummy = new ListNode(0);
    let curr = dummy;
    while (heap.length) {

#### cons
t node = pop();
        if (node.next) push(node.next);
        curr.next = node;
        curr = node;
    }
    return dummy.next;
}
Java
JAVA
import java.util.PriorityQueue;

class ListNode {
    int val;
    ListNode next;
    ListNode(int val) { this.val = val; }
}

class Solution {
    public ListNode mergeKLists(ListNode[] chains) {
        PriorityQueue<ListNode> minHeap = new PriorityQueue<>((a, b) -> a.val - b.val);
        for (ListNode node : chains)
            if (node != null) minHeap.offer(node);

        ListNode dummy = new ListNode(0);
        ListNode curr = dummy;
        while (!minHeap.isEmpty()) {
            ListNode node = minHeap.poll();
            if (node.next != null) minHeap.offer(node.next);
            curr.next = node;
            curr = node;
        }
        return dummy.next;
    }
}
C++
CPP
#include <queue>
#include <vector>
using namespace std;

struct ListNode {
    int val;
    ListNode *next;
    ListNode(int x) : val(x), next(nullptr) {}
};

class Solution {
public:
    ListNode *mergeKLists(vector<ListNode *> &chains) {
        auto greater = [] (ListNode *a, ListNode *b) { return a->val > b->val; };
        priority_queue<ListNode *, vector<ListNode *>, decltype(greater)> minHeap(greater);

        for (ListNode *node : chains)
            if (node) minHeap.push(node);

        ListNode dummy(0);
        ListNode *curr = &dummy;
        while (!minHeap.empty()) {
            ListNode *node = minHeap.top();
            minHeap.pop();
            if (node->next) minHeap.push(node->next);
            curr->next = node;
            curr = node;
        }
        return dummy.next;
    }
};
C
C
#include <stdlib.h>

struct ListNode {
    int val;
    struct ListNode *next;
};

/* Binary min-heap of ListNode* keyed by node value. */
typedef struct {
    struct ListNode **data;
    int size;
} Heap;

static void heapPush(Heap *h, struct ListNode *node) {
    int i = h->size++;
    h->data[i] = node;
    while (i > 0) {
        int parent = (i - 1) / 2;
        if (h->data[parent]->val <= h->data[i]->val) break;
        struct ListNode *tmp = h->data[parent];
        h->data[parent] = h->data[i];
        h->data[i] = tmp;
        i = parent;
    }
}

static struct ListNode *heapPop(Heap *h) {
    struct ListNode *top = h->data[0];
    h->data[0] = h->data[--h->size];
    int i = 0;
    while (1) {
        int l = 2 * i + 1, r = 2 * i + 2, smallest = i;
        if (l < h->size && h->data[l]->val < h->data[smallest]->val) smallest = l;
        if (r < h->size && h->data[r]->val < h->data[smallest]->val) smallest = r;
        if (smallest == i) break;
        struct ListNode *tmp = h->data[smallest];
        h->data[smallest] = h->data[i];
        h->data[i] = tmp;
        i = smallest;
    }
    return top;
}

struct ListNode *mergeKLists(struct ListNode **chains, int chainsSize) {
    Heap h;
    h.data = malloc(sizeof(struct ListNode *) * (chainsSize > 0 ? chainsSize : 1));
    h.size = 0;
    for (int i = 0; i < chainsSize; i++)
        if (chains[i]) heapPush(&h, chains[i]);

    struct ListNode dummy;
    dummy.next = NULL;
    struct ListNode *curr = &dummy;
    while (h.size > 0) {
        struct ListNode *node = heapPop(&h);
        if (node->next) heapPush(&h, node->next);
        curr->next = node;
        curr = node;
    }
    curr->next = NULL;
    free(h.data);
    return dummy.next;
}

#### Follow-up
questions
Compare the min-heap solution with pairwise divide-and-conquer merging: both are O(N log k), so what differs in constants, cache behavior, and code complexity?
What if the chains are so large they cannot all be held in memory (external / streaming merge from disk)?
Generalize to merging k sorted arrays or k sorted input streams instead of linked lists.
How would you merge into descending order, or drop duplicate values as you merge?

