---
type: "practice"
difficulty: "Other"
---

# Membership in a Single-Inheritance Type Hierarchy

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Inheritance Chain](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Inheritance_Chain.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are working with a simplified type system that uses single inheritance: every type has at most one direct parent type.

There are n types, numbered 0 to n - 1. You are given an integer array parent of length n, where parent[i] is the id of the direct parent of type i, or -1 if type i is a root type (it has no parent). Following the parent links repeatedly from any type walks up the hierarchy toward a root; the relationship is guaranteed to form a forest (there are no cycles).

An object in this system is described only by the id of the type it was created from. We say an object of type objType is an instance of targetType when targetType is equal to objType itself or is any ancestor of objType (i.e. targetType is reachable by starting at objType and repeatedly following parent links). In other words, an object is an instance of its own type and of every type above it in the chain.

Write a function that, given parent, an objType, and a targetType, returns true if the object is an instance of targetType, and false otherwise.

Handle "missing" inputs: either objType or targetType may be -1, representing an absent value with no type. If either one is -1, the answer is false.

Input

parent: integer array of length n describing the hierarchy (parent[i] == -1 for a root, otherwise a valid type id).
objType: the id of the object's type, or -1 for a missing object.
targetType: the id of the type being tested against, or -1 for a missing type.

Output

A boolean: whether the object is an instance of targetType.

#### Cons
traints

1 <= n <= 10^4
Each parent[i] is either -1 or an integer in [0, n - 1].
The parent links form a forest (no cycles); each type has at most one direct parent.
-1 <= objType < n and -1 <= targetType < n.

#### Example
s & test cases
TEXT
Assume the hierarchy `parent = [-1, 0, 1, 0, 3]` meaning: type 0 is a root; type 1's parent is 0; type 2's parent is 1; type 3's parent is 0; type 4's parent is 3. So the chains are 2 -> 1 -> 0 and 4 -> 3 -> 0.

- Input: parent=[-1,0,1,0,3], objType=2, targetType=0 -> Output: true
  Why: walking up from 2 gives 2, 1, 0; the target 0 is an ancestor, so it is an instance (normal ancestor hit).
- Input: parent=[-1,0,1,0,3], objType=2, targetType=2 -> Output: true
  Why: an object is always an instance of its own type; the walk matches on the very first step.
- Input: parent=[-1,0,1,0,3], objType=4, targetType=1 -> Output: false
  Why: the chain 4 -> 3 -> 0 never reaches type 1, so type 1 is on a different branch (negative case).
- Input: parent=[-1,0,1,0,3], objType=3, targetType=4 -> Output: false
  Why: the target is a descendant, not an ancestor; membership only counts self and ancestors.
- Input: parent=[-1,0,1,0,3], objType=-1, targetType=0 -> Output: false
  Why: a missing object (objType = -1) is never an instance of anything (edge case).
- Input: parent=[-1], objType=0, targetType=0 -> Output: true
  Why: a single root type; an object of it is an instance of itself (minimal hierarchy edge case).

#### Approach

This is a parent-pointer chain traversal (the same idea a language runtime uses to resolve instanceof by walking a prototype/superclass chain). Membership of an object in a type reduces to: "is the target reachable by moving upward from the object's type?" Start a cursor at objType and repeatedly jump to parent[cursor]. If the cursor ever equals targetType, the object is an instance (this naturally includes the object's own type on the first check). If the cursor reaches -1 (a root's parent) without a match, it is not. Missing inputs (-1) short-circuit to false up front. Because each type has a single parent, the path is a simple chain with no branching, so a plain loop suffices — no recursion or extra memory needed. For many repeated queries, the chain walk can be preprocessed away with depth + binary lifting or Euler/interval labeling.

Complexity

Time: O(d), where d is the depth of objType in the hierarchy (worst case O(n) for a single long chain), since we follow one parent link per step and stop on a match or at a root. Space: O(1) — only a single moving index is kept, no extra structures.

Python
PYTHON
from typing import List

def is_instance_of(parent: List[int], obj_type: int, target_type: int) -> bool:
    """Walk up the parent chain from obj_type; True if target_type is found."""
    if obj_type < 0 or target_type < 0:
        return False
    cur = obj_type
    while cur != -1:
        if cur == target_type:
            return True
        cur = parent[cur]
    return False
JavaScript
JS
/**
 * @param {number[]} parent - parent[i] is the direct parent of type i, or -1.
 * @param {number} objType
 * @param {number} targetType
 * @return {boolean}
 */
function isInstanceOf(parent, objType, targetType) {
    if (objType < 0 || targetType < 0) {
        return false;
    }
    let cur = objType;
    while (cur !== -1) {
        if (cur === targetType) {
            return true;
        }
        cur = parent[cur];
    }
    return false;
}
Java
JAVA
class Solution {
    // Returns true iff targetType equals objType or is an ancestor of it.
    public boolean isInstanceOf(int[] parent, int objType, int targetType) {
        if (objType < 0 || targetType < 0) {
            return false;
        }
        int cur = objType;
        while (cur != -1) {
            if (cur == targetType) {
                return true;
            }
            cur = parent[cur];
        }
        return false;
    }
}
C++
CPP
#include <vector>
using namespace std;

class Solution {
public:
    // Walk up the single-inheritance chain from objType looking for targetType.
    bool isInstanceOf(const vector<int>& parent, int objType, int targetType) {
        if (objType < 0 || targetType < 0) {
            return false;
        }
        int cur = objType;
        while (cur != -1) {
            if (cur == targetType) {
                return true;
            }
            cur = parent[cur];
        }
        return false;
    }
};
C
C
#include <stdbool.h>

/* Returns whether an object of type objType is an instance of targetType,
 * i.e. targetType is objType or one of its ancestors in the parent chain. */
bool isInstanceOf(const int* parent, int n, int objType, int targetType) {
    (void)n; /* n is available for validation; not needed by the walk itself */
    if (objType < 0 || targetType < 0) {
        return false;
    }
    int cur = objType;
    while (cur != -1) {
        if (cur == targetType) {
            return true;
        }
        cur = parent[cur];
    }
    return false;
}

#### Follow-up
questions
What changes if a type may have multiple direct parents (multiple inheritance / a DAG)? How does the membership check become a graph search, and how do you avoid revisiting shared ancestors?
How would you detect and guard against a malformed hierarchy that contains a cycle so the walk cannot loop forever?
Given millions of repeated membership queries on one fixed hierarchy, how would you preprocess it (depth + binary lifting, or interval/Euler labeling) so each query is answered in O(1) or O(log n)?
In a real language runtime, how does instanceof actually walk the prototype (or vtable/superclass) chain, and what edge cases arise for boxed primitives versus plain objects?

