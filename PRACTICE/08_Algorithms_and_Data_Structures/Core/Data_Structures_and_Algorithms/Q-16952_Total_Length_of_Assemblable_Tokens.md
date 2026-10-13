---
type: "practice"
difficulty: "Other"
---

# Total Length of Assemblable Tokens

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Hash Table](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Hash_Table.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a list of lowercase tokens tokens and a single string pool that represents a bag of available lowercase letters.

A token is called assemblable if it can be built entirely from the letters in pool, where each letter of pool may be used at most once per token. In other words, for a token to be assemblable, for every distinct letter it contains, the number of times that letter appears in the token must not exceed the number of times it appears in pool. The bag is conceptually refreshed for every token (spending letters on one token does not reduce what is available for another token).

Return the sum of the lengths of all assemblable tokens in tokens.

Input
tokens: an array of strings, each made of lowercase English letters.
pool: a string of lowercase English letters.
Output
A single integer: the total number of characters across all tokens that can be assembled from pool.

#### Cons
traints
1 <= tokens.length <= 1000
1 <= tokens[i].length <= 100
1 <= pool.length <= 100
tokens[i] and pool consist only of lowercase English letters (a–z).

#### Example
s & test cases
TEXT
Input: tokens = ["ace","bad","cab","dice"], pool = "abcde" -> Output: 9
Why: "ace", "bad", "cab" each use only letters present in the pool (3+3+3=9); "dice"
needs an 'i' that the pool lacks, so it is excluded.

Input: tokens = ["bee","cat","deed","ball"], pool = "abcdee" -> Output: 3
Why: "bee" needs two 'e's and the pool has exactly two, so it qualifies (length 3);
"cat" needs a 't', "deed" needs two 'd's (only one available), and "ball" needs an 'l',
so all three are rejected.

Input: tokens = ["ape","pan","rope","zebra"], pool = "orangep" -> Output: 10
Why: "ape"(3), "pan"(3), and "rope"(4) can all be formed; "zebra" needs a 'z' and 'b'
not in the pool. 3+3+4 = 10.

Input: tokens = ["xyz"], pool = "a" -> Output: 0
Why: A single token that cannot be formed at all; the answer is 0 (minimal case).

Input: tokens = ["a","aa","aaa"], pool = "aa" -> Output: 3
Why: "a" and "aa" fit within two available 'a's (1+2=3); "aaa" needs three 'a's and is
rejected — demonstrates the per-letter count check with duplicates.

#### Approach

This is a frequency-counting / hash-table problem. First build a fixed-size (26-letter) frequency table for pool. Then, for each token, build its own letter-frequency table and verify that no letter is required more times than the pool provides. A token passes exactly when every letter's needed count is <= the pool's count, at which point its length is added to the running total. Using a fixed 26-entry array instead of a hash map keeps the per-comparison cost constant.

Complexity

Time: O(P + S) where P is the length of pool and S is the total number of characters across all tokens — each character is scanned a constant number of times, and the fixed 26-letter comparison per token is O(1). Space: O(1) — only two fixed 26-slot frequency arrays are used regardless of input size.

Python
PYTHON
from collections import Counter

class Solution:
    def assemblableLength(self, tokens, pool):
        pool_count = Counter(pool)
        total = 0
        for w in tokens:
            need = Counter(w)
            if all(need[ch] <= pool_count[ch] for ch in need):
                total += len(w)
        return total
JavaScript
JAVASCRIPT
function assemblableLength(tokens, pool) {

#### cons
t poolCount = new Array(26).fill(0);

#### cons
t base = 'a'.charCodeAt(0);
    for (const ch of pool) poolCount[ch.charCodeAt(0) - base]++;

    let total = 0;
    for (const w of tokens) {

#### cons
t need = new Array(26).fill(0);
        for (const ch of w) need[ch.charCodeAt(0) - base]++;
        let ok = true;
        for (let c = 0; c < 26; c++) {
            if (need[c] > poolCount[c]) { ok = false; break; }
        }
        if (ok) total += w.length;
    }
    return total;
}
Java
JAVA
class Solution {
    public int assemblableLength(String[] tokens, String pool) {
        int[] poolCount = new int[26];
        for (int i = 0; i < pool.length(); i++)
            poolCount[pool.charAt(i) - 'a']++;

        int total = 0;
        for (String w : tokens) {
            int[] need = new int[26];
            for (int i = 0; i < w.length(); i++)
                need[w.charAt(i) - 'a']++;
            boolean ok = true;
            for (int c = 0; c < 26; c++) {
                if (need[c] > poolCount[c]) { ok = false; break; }
            }
            if (ok) total += w.length();
        }
        return total;
    }
}
C++
CPP
#include <string>
#include <vector>
using namespace std;

class Solution {
public:
    int assemblableLength(vector<string>& tokens, string pool) {
        vector<int> poolCount(26, 0);
        for (char ch : pool) poolCount[ch - 'a']++;

        int total = 0;
        for (const string& w : tokens) {
            vector<int> need(26, 0);
            for (char ch : w) need[ch - 'a']++;
            bool ok = true;
            for (int c = 0; c < 26 && ok; c++)
                if (need[c] > poolCount[c]) ok = false;
            if (ok) total += (int)w.size();
        }
        return total;
    }
};
C
C
#include <string.h>

int assemblableLength(char** tokens, int tokensSize, char* pool) {
    int poolCount[26] = {0};
    for (int i = 0; pool[i] != '\0'; i++) {
        poolCount[pool[i] - 'a']++;
    }

    int total = 0;
    for (int t = 0; t < tokensSize; t++) {
        int need[26] = {0};
        int len = 0;
        int ok = 1;
        char* w = tokens[t];
        for (int i = 0; w[i] != '\0'; i++) {
            need[w[i] - 'a']++;
            len++;
        }
        for (int c = 0; c < 26; c++) {
            if (need[c] > poolCount[c]) { ok = 0; break; }
        }
        if (ok) total += len;
    }
    return total;
}

#### Follow-up
questions
What changes if each pool letter can be reused an unlimited number of times per token (set-membership instead of counting)?
Instead of the total length, return the actual list of assemblable tokens in their original order.
How would you handle a case-insensitive or full-Unicode alphabet where a fixed 26-array no longer works?
If tokens arrives as a stream of billions of entries but pool is fixed, how do you keep the per-token check O(1) amortized and avoid re-scanning pool?

