---
type: "practice"
difficulty: "Other"
---

# Uniform Random Value from a Singly Linked List

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Linked List](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Linked_List.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given the head of a singly linked list whose nodes each hold an integer value. Build a class that can repeatedly return the value of a uniformly random node — every node in the list must have exactly the same chance of being chosen on each query.

Implement RandomListSampler:

RandomListSampler(ListNode head) — initialize the object with the head of a non-empty singly linked list.
int pick() — return the value stored in one node picked uniformly at random. Across many independent calls, each node should be selected with probability 1 / n, where n is the number of nodes in the list.

The interesting constraint: assume you may not know the list's length ahead of time and cannot afford to store all values in an auxiliary array. You should be able to answer each query in a single forward pass using only O(1) extra memory.

Input: the head of a singly linked list, followed by a series of pick() calls.

Output: for each pick() call, one node value, drawn uniformly at random over the nodes.

#### Cons
traints:

The number of nodes n is in the range [1, 10^4].
Each node value v satisfies -10^4 <= v <= 10^4.
At most 10^4 calls will be made to pick().
Each pick() must give every node probability exactly 1/n and use only O(1) additional space.

#### Example
s & test cases
TEXT
Input: list = 5 -> 9 -> 2 ; pick() -> Output: one of {5, 9, 2}, each with probability 1/3
Why: with three nodes, a uniform pick returns any of the three values one-third of the time.

Input: list = 42 ; pick(), pick(), pick() -> Output: 42, 42, 42
Why: a single-node list can only ever return that node's value, no matter how many times pick is called.

Input: list = -4 -> -4 -> 7 ; pick() -> Output: -4 with probability 2/3 or 7 with probability 1/3
Why: probability is over nodes, not distinct values; two nodes carry -4 so that value surfaces twice as often.

Input: list = 1 -> 2 ; pick() repeated many times -> Output: roughly half 1's and half 2's
Why: over a large number of independent calls each of the two nodes is chosen about 50% of the time.

Input: list = 3 -> 6 -> 9 -> 12 ; pick() -> Output: one of {3, 6, 9, 12}, each with probability 1/4
Why: reservoir sampling keeps every node equally likely even though later nodes are seen last.

#### Approach

Use reservoir sampling with a reservoir of size one. Walk the list while keeping a single "currently chosen" value. When visiting the i-th node (1-indexed), replace the chosen value with this node's value with probability 1/i, and keep the old choice with probability (i-1)/i. By induction, after processing i nodes each of those i nodes is the current choice with probability exactly 1/i, so when the walk ends at node n every node holds probability 1/n. This needs no knowledge of the length in advance and only O(1) memory, which is exactly why it beats the naive "count first, then index" or "copy to an array" methods when the length is unknown or the list is huge. The underlying technique is reservoir sampling over a stream.

Complexity

Time: O(n) per pick() call, where n is the number of nodes — each query walks the whole list once. Space: O(1) extra — only a running chosen value, a counter, and a node pointer are kept, regardless of list length, so nothing needs to be materialized up front.

Python
PYTHON
import random

class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

class RandomListSampler:
    def __init__(self, head: 'ListNode'):
        self.head = head

    def pick(self) -> int:
        result = 0
        i = 1
        node = self.head
        while node:
            if random.randrange(i) == 0:      # keep current node with probability 1/i
                result = node.val
            node = node.next
            i += 1
        return result
JavaScript
JAVASCRIPT
function ListNode(val, next = null) {
    this.val = val;
    this.next = next;
}

class RandomListSampler {

#### cons
tructor(head) {
        this.head = head;
    }

    pick() {
        let result = 0, i = 1;
        for (let cur = this.head; cur !== null; cur = cur.next, i++) {
            if (Math.floor(Math.random() * i) === 0)  // keep current node with prob 1/i
                result = cur.val;
        }
        return result;
    }
}
Java
JAVA
import java.util.Random;

class ListNode {
    int val;
    ListNode next;
    ListNode(int x) { val = x; }
}

class RandomListSampler {
    private final ListNode head;
    private final Random rand = new Random();

    public RandomListSampler(ListNode head) {
        this.head = head;
    }

    public int pick() {
        int result = 0, i = 1;
        for (ListNode cur = head; cur != null; cur = cur.next, i++)
            if (rand.nextInt(i) == 0)      // keep current node with probability 1/i
                result = cur.val;
        return result;
    }
}
C++
CPP
#include <cstdlib>

struct ListNode {
    int val;
    ListNode *next;
    ListNode(int x) : val(x), next(nullptr) {}
};

class RandomListSampler {
public:
    RandomListSampler(ListNode *head) : head(head) {}

    int pick() {
        int result = 0, i = 1;
        for (ListNode *cur = head; cur; cur = cur->next, ++i)
            if (rand() % i == 0)          // keep current node with probability 1/i
                result = cur->val;
        return result;
    }

private:
    ListNode *head;
};
C
C
#include <stdlib.h>
#include <time.h>

struct ListNode {
    int val;
    struct ListNode *next;
};

typedef struct {
    struct ListNode *head;
} RandomListSampler;

RandomListSampler *samplerCreate(struct ListNode *head) {
    RandomListSampler *o = (RandomListSampler *)malloc(sizeof(RandomListSampler));
    o->head = head;
    srand((unsigned)time(NULL));
    return o;
}

int samplerPick(RandomListSampler *o) {
    int result = 0, i = 1;
    for (struct ListNode *cur = o->head; cur; cur = cur->next, ++i)
        if (rand() % i == 0)          /* keep current node with probability 1/i */
            result = cur->val;
    return result;
}

void samplerFree(RandomListSampler *o) {
    free(o);
}

#### Follow-up
questions
The list is so large its length is unknown and it may not fit in memory — show that your one-pass method still gives a uniform pick (this is the reservoir-sampling argument).
Generalize to return k distinct random nodes in one pass using a reservoir of size k.
Support weighted sampling where each node's chance is proportional to its value or an attached weight.
If pick() is called millions of times on a fixed list, when is it worth paying O(n) space to copy values into an array for O(1) queries, and what is the crossover?

