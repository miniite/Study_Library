---
type: "practice"
difficulty: "Other"
---

# Time-Limited Key-Value Store

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Design](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Design a data structure ExpiringStore that holds integer key-value pairs, where every key has a limited lifetime. The structure has no access to an ambient clock, so the current time in milliseconds is supplied to every operation as an explicit argument now. Across the sequence of operations, now is non-decreasing.

Support three operations:

set(key, value, duration, now): store value under key with an expiry timestamp of now + duration. The entry stays readable while the current time is strictly less than its expiry timestamp; once time reaches (or passes) the expiry timestamp the key is considered expired and inaccessible. If key currently exists and has not yet expired at time now, overwrite its value, reset its lifetime to the new duration, and return true. Otherwise (the key is absent, or present but already expired), create/replace it and return false.
get(key, now): if key exists and is not expired at time now, return its stored value; otherwise return -1.
count(now): return how many keys are not expired at time now.

A key is live / unexpired at time t iff t < expiryTimestamp for that key. Consequently a duration of 0 expires the key immediately — it is already expired at the very same now at which it was set.

Input / Output. Implement the class. A driver applies a sequence of operations in time order: set returns a boolean, get returns an integer (the stored value or -1), and count returns an integer.

#### Cons
traints:

0 <= key, value <= 10^9
0 <= duration <= 1000
0 <= now <= 10^9, and now is non-decreasing across the operation sequence
at most 10^5 operations are performed
for a get/count on a never-set key, the result is -1 / 0 respectively

#### Example
s & test cases
TEXT
Input: set(7,42,100,now=0); get(7,now=50); count(now=50); get(7,now=150) -> Output: [false, 42, 1, -1]
Why: new key returns false on set; still live at t=50 so value and count reflect it; expires at t=100 so t=150 read returns -1 (normal lifecycle).

Input: set(3,10,50,now=0); set(3,20,100,now=40); get(3,now=50); get(3,now=120); get(3,now=200); count(now=250) -> Output: [false, true, 20, 20, -1, 0]
Why: overwriting a still-live key returns true and resets its lifetime (new expiry 140); reads succeed until 140, then -1 and count 0 (overwrite path).

Input: set(5,99,0,now=0); get(5,now=0) -> Output: [false, -1]
Why: duration 0 means expiry == now, and "live" requires now < expiry, so the key is expired the instant it is set (edge: zero duration).

Input: set(2,1,30,now=0); set(2,2,30,now=40); get(2,now=50) -> Output: [false, false, 2]
Why: at t=40 the first entry (expiry 30) is already expired, so the second set returns false even though the key was seen before; the fresh entry is then readable (re-set after expiry).

Input: get(9,now=10); count(now=10) -> Output: [-1, 0]
Why: querying a key that was never set returns -1 and the store is empty (empty/min case).

#### Approach

Store each key in a hash map together with an absolute expiry timestamp (now + duration) instead of a countdown or a real timer. A key is "live" exactly when the query time is strictly less than its expiry, so expiration is evaluated lazily on read rather than actively deleted. set decides its boolean return by first checking whether the existing entry is still live at now (before overwriting). This is the classic lazy-expiration TTL cache pattern; it removes any dependence on wall-clock timers and makes the structure fully deterministic given the passed-in timestamps.

Complexity

Time: set and get are O(1) average (single hash-map lookup); count(now) is O(k) where k is the number of stored keys, since it scans entries and tests each expiry against now. Space: O(k) for the stored keys. The design avoids background timers by storing an absolute expiry timestamp and comparing lazily at access time.

Python
PYTHON
class ExpiringStore:
    def __init__(self):
        self.data = {}  # key -> (value, absolute_expiry)

    def set(self, key, value, duration, now):
        entry = self.data.get(key)
        existed = entry is not None and entry[1] > now
        self.data[key] = (value, now + duration)
        return existed

    def get(self, key, now):
        entry = self.data.get(key)
        if entry is not None and entry[1] > now:
            return entry[0]
        return -1

    def count(self, now):
        return sum(1 for _, expiry in self.data.values() if expiry > now)
JavaScript
JAVASCRIPT
class ExpiringStore {

#### cons
tructor() {
    this.data = new Map(); // key -> { value, expiry }
  }

  // returns true if an un-expired key already existed at time `now`
  set(key, value, duration, now) {

#### cons
t e = this.data.get(key);

#### cons
t existed = e !== undefined && e.expiry > now;
    this.data.set(key, { value, expiry: now + duration });
    return existed;
  }

  get(key, now) {

#### cons
t e = this.data.get(key);
    if (e !== undefined && e.expiry > now) return e.value;
    return -1;
  }

  count(now) {
    let c = 0;
    for (const e of this.data.values()) if (e.expiry > now) c++;
    return c;
  }
}
Java
JAVA
import java.util.HashMap;
import java.util.Map;

class ExpiringStore {
    private static final class Entry {
        long value, expiry;
        Entry(long v, long e) { value = v; expiry = e; }
    }
    private final Map<Integer, Entry> data = new HashMap<>();

    /** true if an un-expired key already existed at time now */
    public boolean set(int key, long value, long duration, long now) {
        Entry e = data.get(key);
        boolean existed = (e != null && e.expiry > now);
        data.put(key, new Entry(value, now + duration));
        return existed;
    }

    public long get(int key, long now) {
        Entry e = data.get(key);
        if (e != null && e.expiry > now) return e.value;
        return -1;
    }

    public int count(long now) {
        int c = 0;
        for (Entry e : data.values()) if (e.expiry > now) c++;
        return c;
    }
}
C++
CPP
#include <unordered_map>
using namespace std;

class ExpiringStore {
    struct Entry { long value; long expiry; };
    unordered_map<int, Entry> data;   // key -> {value, absolute expiry}
public:
    // true if an un-expired key already existed at time `now`
    bool set(int key, long value, long duration, long now) {
        auto it = data.find(key);
        bool existed = (it != data.end() && it->second.expiry > now);
        data[key] = { value, now + duration };
        return existed;
    }
    long get(int key, long now) {
        auto it = data.find(key);
        if (it != data.end() && it->second.expiry > now) return it->second.value;
        return -1;
    }
    int count(long now) {
        int c = 0;
        for (auto &kv : data) if (kv.second.expiry > now) ++c;
        return c;
    }
};
C
C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct {
    int   cap;
    int  *key;
    char *used;
    long *value;
    long *expiry;   /* absolute expiry timestamp = setTime + duration */
} ExpiringStore;

ExpiringStore *storeCreate(void) {
    ExpiringStore *s = malloc(sizeof(ExpiringStore));
    s->cap    = 1 << 16;
    s->key    = malloc(sizeof(int)  * s->cap);
    s->used   = calloc(s->cap, 1);
    s->value  = malloc(sizeof(long) * s->cap);
    s->expiry = malloc(sizeof(long) * s->cap);
    return s;
}

/* returns the slot holding `key`, or the first empty slot (open addressing) */
static int storeSlot(ExpiringStore *s, int key) {
    unsigned long h = (unsigned long)key * 2654435761u;
    int idx = (int)(h & (s->cap - 1));
    while (s->used[idx] && s->key[idx] != key) idx = (idx + 1) & (s->cap - 1);
    return idx;
}

/* returns 1 (true) if an un-expired key already existed, else 0 */
int storeSet(ExpiringStore *s, int key, long value, long duration, long now) {
    int idx = storeSlot(s, key);
    int existed = s->used[idx] && s->expiry[idx] > now;
    s->used[idx]   = 1;
    s->key[idx]    = key;
    s->value[idx]  = value;
    s->expiry[idx] = now + duration;
    return existed;
}

long storeGet(ExpiringStore *s, int key, long now) {
    int idx = storeSlot(s, key);
    if (s->used[idx] && s->expiry[idx] > now) return s->value[idx];
    return -1;
}

int storeCount(ExpiringStore *s, long now) {
    int c = 0;
    for (int i = 0; i < s->cap; i++)
        if (s->used[i] && s->expiry[i] > now) c++;
    return c;
}

void storeFree(ExpiringStore *s) {
    free(s->key); free(s->used); free(s->value); free(s->expiry); free(s);
}

#### Follow-up
questions
Make count() O(1): keep a live-size counter plus a min-heap / ordered structure of expiry timestamps and purge expired entries lazily before answering.
Add a maximum capacity with LRU eviction on top of the TTL, so the store evicts the least-recently-used key when full.
Support a real wall-clock version that uses actual timers/setTimeout to auto-remove keys, and discuss the trade-offs versus lazy expiration.
Make the structure thread-safe for concurrent get/set/count from multiple threads.

