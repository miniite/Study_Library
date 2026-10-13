---
type: "practice"
difficulty: "Other"
---

# Three-Tier Docking Bay Allocator

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Design](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A logistics depot has docking bays of three fixed sizes: heavy, standard, and light. When the depot opens, it has a known, fixed number of bays of each size, and that capacity never grows.

Design a class BayAllocator that manages this depot:

BayAllocator(int heavy, int standard, int light) — Initializes the allocator with the number of bays available for each size.

boolean dock(int tier) — A vehicle of category tier arrives and requests a bay. The category is encoded as an integer:

1 = heavy
2 = standard
3 = light

A vehicle can only occupy a bay of exactly its own size (a heavy vehicle cannot use a standard bay, and so on). If at least one bay of that size is still free, occupy one of them and return true. Otherwise, leave the depot unchanged and return false.

Once a bay is occupied it stays occupied for the lifetime of the object (there is no "leave" operation), so each successful dock permanently reduces the remaining capacity of that size by one.

#### Cons
traints:

0 <= heavy, standard, light <= 1000
tier is always one of 1, 2, or 3.
At most 1000 calls will be made to dock.

Return type of dock is a boolean. The constructor performs no I/O.

#### Example
s & test cases
TEXT
Input: `BayAllocator(1, 1, 0)`, then `dock(1)` -> Output: `true`
Why: one heavy bay exists and is free, so the heavy vehicle parks.

Input: (same object) then `dock(2)` -> Output: `true`
Why: one standard bay is free; the standard vehicle parks.

Input: (same object) then `dock(3)` -> Output: `false`
Why: zero light bays were configured, so a light vehicle cannot dock.

Input: (same object) then `dock(1)` -> Output: `false`
Why: the single heavy bay is already occupied by the first vehicle, so no heavy space remains.

Input: `BayAllocator(0, 0, 0)`, then `dock(2)` -> Output: `false`
Why: minimum-capacity edge case — every size has zero bays, so any request fails immediately.

Input: `BayAllocator(2, 0, 1)`, then `dock(1)`, `dock(1)`, `dock(1)` -> Output: `true, true, false`
Why: two heavy bays accept the first two vehicles; the third heavy request finds no room.

#### Approach

This is a pure counting/simulation design problem. Keep one integer counter per bay size (a fixed-length array indexed by the category code makes the mapping direct and branch-free). Each dock(tier) checks whether that size's counter is positive; if so, decrement it and report success, otherwise report failure. There is no need to track individual bays or vehicles because they are interchangeable within a size class — only the remaining count matters. The pattern is "maintain aggregate state with O(1) updates," the same idea behind many rate-limiter and resource-pool designs.

Complexity

Time: O(1) per dock call and O(1) for construction — each request only reads and decrements a single counter. Space: O(1) — exactly three counters are stored regardless of how many calls are made.

Python
PYTHON
class BayAllocator:
    def __init__(self, heavy: int, standard: int, light: int):
        # index 1 = heavy, 2 = standard, 3 = light
        self.slots = [0, heavy, standard, light]

    def dock(self, tier: int) -> bool:
        if self.slots[tier] > 0:
            self.slots[tier] -= 1
            return True
        return False
JavaScript
JAVASCRIPT
class BayAllocator {

#### cons
tructor(heavy, standard, light) {
        // index 1 = heavy, 2 = standard, 3 = light
        this.slots = [0, heavy, standard, light];
    }

    dock(tier) {
        if (this.slots[tier] > 0) {
            this.slots[tier]--;
            return true;
        }
        return false;
    }
}
Java
JAVA
class BayAllocator {
    private final int[] slots;

    public BayAllocator(int heavy, int standard, int light) {
        // index 1 = heavy, 2 = standard, 3 = light
        slots = new int[]{0, heavy, standard, light};
    }

    public boolean dock(int tier) {
        if (slots[tier] > 0) {
            slots[tier]--;
            return true;
        }
        return false;
    }
}
C++
CPP
class BayAllocator {
    int slots[4]; // 1 = heavy, 2 = standard, 3 = light
public:
    BayAllocator(int heavy, int standard, int light) {
        slots[1] = heavy;
        slots[2] = standard;
        slots[3] = light;
    }

    bool dock(int tier) {
        if (slots[tier] > 0) {
            slots[tier]--;
            return true;
        }
        return false;
    }
};
C
C
#include <stdbool.h>
#include <stdlib.h>

typedef struct {
    int slots[4]; /* index 1 = heavy, 2 = standard, 3 = light; index 0 unused */
} BayAllocator;

BayAllocator* bayAllocatorCreate(int heavy, int standard, int light) {
    BayAllocator* obj = (BayAllocator*)malloc(sizeof(BayAllocator));
    obj->slots[0] = 0;
    obj->slots[1] = heavy;
    obj->slots[2] = standard;
    obj->slots[3] = light;
    return obj;
}

bool bayAllocatorDock(BayAllocator* obj, int tier) {
    if (obj->slots[tier] > 0) {
        obj->slots[tier]--;
        return true;
    }
    return false;
}

void bayAllocatorFree(BayAllocator* obj) {
    free(obj);
}

#### Follow-up
questions
Add a release(tier) method so bays can be freed when a vehicle leaves; how do you prevent the count from exceeding the original capacity?
Support arbitrary size categories supplied as a map at construction rather than exactly three fixed sizes.
Allow a vehicle to fall back to a larger free bay when its exact size is full; return which size it was placed in.
Make the allocator thread-safe for concurrent dock calls without a global lock.

