---
type: "practice"
difficulty: "Other"
---

# The Kth Token That Appears Exactly Once

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Hash Table](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Hash_Table.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a list of lowercase text tokens tokens and a positive integer k.

A token is called unique if it appears exactly once in the entire list. Tokens that occur two or more times are not unique.

#### Cons
ider the unique tokens in the order in which they first appear while scanning the list from left to right. Return the k-th unique token in that order.

If the list contains fewer than k unique tokens, return the empty string "".

Input

tokens: an array of strings, each made of lowercase English letters.
k: a positive integer (1-based rank of the unique token to return).

Output

The k-th unique token as a string, or "" if fewer than k unique tokens exist.

#### Cons
traints

1 <= k <= tokens.length <= 1000
1 <= tokens[i].length <= 5
tokens[i] consists only of lowercase English letters.

#### Example
s & test cases
TEXT
Input: tokens = ["fig", "sun", "sky", "sun", "sky", "owl"], k = 2 -> Output: "owl"
Why: "sun" and "sky" each appear twice, so the only unique tokens are "fig" (1st) and "owl" (2nd). k=2 selects "owl".

Input: tokens = ["aaa", "aa", "a"], k = 1 -> Output: "aaa"
Why: every token is distinct, so all are unique; the 1st in order of appearance is "aaa".

Input: tokens = ["red", "sky", "red"], k = 3 -> Output: ""
Why: only "sky" is unique (count 1). There are fewer than 3 unique tokens, so return "".

Input: tokens = ["hi"], k = 1 -> Output: "hi"
Why: minimum-size list with one token; it appears once, so it is the 1st unique token.

Input: tokens = ["ab", "cd", "ab", "cd", "ef", "gh"], k = 2 -> Output: "gh"
Why: "ab" and "cd" repeat; the unique tokens in order are "ef" (1st) and "gh" (2nd), so k=2 gives "gh".

Input: tokens = ["one", "one", "two", "two"], k = 1 -> Output: ""
Why: no token appears exactly once, so even the 1st unique token does not exist -> "".

#### Approach

This is a frequency-counting problem. First tally how many times each token occurs using a hash map. A token is "unique" exactly when its count equals 1. Then scan the list a second time in original order, decrementing k every time a count-1 token is seen; the token that drives k to 0 is the answer. Scanning in the original order preserves the required first-appearance ordering without any sorting. If the second pass ends without k reaching 0, there are fewer than k unique tokens, so return the empty string. The core pattern is a two-pass hash-map count-then-select.

Complexity

Time: O(n) where n is the number of tokens — one pass to build the frequency map and one pass to find the k-th unique token; each hash operation on a bounded-length string is O(1). Space: O(n) for the frequency map storing at most n distinct tokens.

Python
PYTHON
from collections import Counter
from typing import List

class Solution:
    def kthUniqueToken(self, tokens: List[str], k: int) -> str:
        count = Counter(tokens)
        for t in tokens:
            if count[t] == 1:
                k -= 1
                if k == 0:
                    return t
        return ""
JavaScript
JAVASCRIPT
/**
 * @param {string[]} tokens
 * @param {number} k
 * @return {string}
 */
function kthUniqueToken(tokens, k) {

#### cons
t count = new Map();
    for (const t of tokens) count.set(t, (count.get(t) || 0) + 1);
    for (const t of tokens) {
        if (count.get(t) === 1 && --k === 0) return t;
    }
    return "";
}
Java
JAVA
import java.util.HashMap;
import java.util.Map;

class Solution {
    public String kthUniqueToken(String[] tokens, int k) {
        Map<String, Integer> count = new HashMap<>();
        for (String t : tokens) count.merge(t, 1, Integer::sum);
        for (String t : tokens) {
            if (count.get(t) == 1 && --k == 0) return t;
        }
        return "";
    }
}
C++
CPP
#include <string>
#include <vector>
#include <unordered_map>
using namespace std;

class Solution {
public:
    string kthUniqueToken(vector<string>& tokens, int k) {
        unordered_map<string, int> count;
        for (const string& t : tokens) count[t]++;
        for (const string& t : tokens) {
            if (count[t] == 1 && --k == 0) return t;
        }
        return "";
    }
};
C
C
#include <stdlib.h>
#include <string.h>

/* Open-addressing hash table mapping a token string -> its occurrence count. */
typedef struct { const char *key; int count; char used; } Slot;

static unsigned int hashStr(const char *s, unsigned int mask) {
    unsigned long long h = 1469598103934665603ULL; /* FNV-1a */
    for (; *s; ++s) { h ^= (unsigned char)*s; h *= 1099511628211ULL; }
    return (unsigned int)h & mask;
}

/* Returns the k-th unique token, or a heap copy of "" if none. */
char* kthUniqueToken(char** tokens, int tokensSize, int k) {
    unsigned int cap = 1;
    while (cap < (unsigned int)tokensSize * 2u) cap <<= 1;
    if (cap < 2) cap = 2;
    unsigned int mask = cap - 1;
    Slot* table = (Slot*)calloc(cap, sizeof(Slot));

    /* Pass 1: count every token. */
    for (int i = 0; i < tokensSize; i++) {

#### cons
t char* key = tokens[i];
        unsigned int h = hashStr(key, mask);
        while (table[h].used && strcmp(table[h].key, key) != 0) h = (h + 1) & mask;
        if (!table[h].used) { table[h].used = 1; table[h].key = key; table[h].count = 1; }
        else table[h].count++;
    }

    /* Pass 2: walk in appearance order, pick the k-th token whose count is 1. */
    char* answer = NULL;
    int need = k;
    for (int i = 0; i < tokensSize && answer == NULL; i++) {

#### cons
t char* key = tokens[i];
        unsigned int h = hashStr(key, mask);
        while (strcmp(table[h].key, key) != 0) h = (h + 1) & mask;
        if (table[h].count == 1 && --need == 0) {
            answer = (char*)malloc(strlen(key) + 1);
            strcpy(answer, key);
        }
    }
    free(table);
    if (answer == NULL) { answer = (char*)malloc(1); answer[0] = '\0'; }
    return answer;
}

#### Follow-up
questions
What if you must return all unique tokens in order rather than just the k-th one?
How would you handle a streaming input where tokens arrive one at a time and memory is limited?
What changes if "unique" is redefined as "appears at most twice" instead of exactly once?
Can you return the k-th unique token counting from the end of the list instead of the start?

