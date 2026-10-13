---
type: "practice"
difficulty: "Other"
---

# Range Queries for Words Bounded by Vowels

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Prefix Sum](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Prefix_Sum.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a 0-indexed array of strings tokens, and a list of range queries queries, where each query is a pair [left, right] with 0 <= left <= right < tokens.length.

For each query [left, right], you must count how many strings among tokens[left], tokens[left+1], ..., tokens[right] (both endpoints inclusive) both begin and end with a vowel. The vowel letters are 'a', 'e', 'i', 'o', and 'u'. All strings consist only of lowercase English letters.

A single-character string that is a vowel qualifies, because its first and last character are the same vowel (for example "e" counts).

Return an array answer of the same length as queries, where answer[i] is the count for the i-th query.

Input

tokens: an array of non-empty lowercase strings.
queries: an array of [left, right] index pairs.

Output

An integer array, one count per query, in the same order as the queries.

#### Cons
traints

1 <= tokens.length <= 10^5
1 <= tokens[i].length <= 40
tokens[i] consists only of lowercase English letters.
sum(tokens[i].length) <= 3 * 10^5
1 <= queries.length <= 10^5
0 <= left <= right < tokens.length

Because there can be up to 10^5 queries over up to 10^5 strings, an approach that re-scans the range for every query is too slow; precompute so each query is answered in O(1).

#### Example
s & test cases
TEXT
Input: tokens = ["oreo","bit","ivy","ai","u"], queries = [[0,2],[1,4],[0,4]] -> Output: [1,2,3]
Why: Qualifying strings (start and end with a vowel) are "oreo" (o..o) at index 0, "ai" (a..i) at index 3, and "u" (u..u) at index 4. "bit" starts with b, "ivy" ends with y, so neither qualifies. Query [0,2] covers indices 0,1,2 -> only index 0 -> 1. Query [1,4] covers 1..4 -> indices 3 and 4 -> 2. Query [0,4] covers all -> indices 0,3,4 -> 3.

Input: tokens = ["a","e","i"], queries = [[0,2],[0,1],[2,2]] -> Output: [3,2,1]
Why: Every string is a single vowel, so all qualify. The three queries count 3, 2, and 1 qualifying strings respectively. Tests the single-character-vowel edge case.

Input: tokens = ["sky","dry","gym"], queries = [[0,2]] -> Output: [0]
Why: None of the strings start or end with a vowel, so the count over the whole range is 0. Tests the all-negative case.

Input: tokens = ["ohio"], queries = [[0,0]] -> Output: [1]
Why: Single element that starts with 'o' and ends with 'o'; the only query is a point range [0,0]. Tests minimal array size and a point query.

Input: tokens = ["apple","banana","ok","xyz"], queries = [[0,0],[1,3],[0,3]] -> Output: [1,1,2]
Why: "apple" (a..e) qualifies, "banana" (b..a) no, "ok" (o..k) no, "xyz" no. Wait recheck: qualifying are only "apple" (index 0). "ok" starts with vowel 'o' but ends with 'k', so it does not qualify. So indices qualifying: only 0. Query [0,0] -> 1, query [1,3] -> 0, query [0,3] -> 1. Output: [1,0,1].
Why: Confirms that BOTH ends must be vowels, not just one; "ok" and "banana" are excluded.

#### Approach

The qualifying test for each string depends only on its first and last characters, so mark each string with 1 if it both starts and ends with a vowel, else 0. Build a prefix-sum array prefix where prefix[i] is the number of qualifying strings among the first i strings. Any range count [left, right] is then prefix[right + 1] - prefix[left], answered in constant time. The pattern is classic prefix sums for immutable range-sum queries: precompute cumulative counts once, then serve each query in O(1) instead of re-scanning the range.

Complexity

Time: O(n + Q), where n is the number of strings and Q is the number of queries — one linear pass builds the prefix-sum array (checking only the first and last character of each string, which is O(1) each), and every query is answered in O(1). Space: O(n) for the prefix-sum array (plus O(Q) for the output). This replaces the naive O(Q * n) range re-scan.

Python
PYTHON
from typing import List

def count_vowel_bounded_in_ranges(tokens: List[str],
                                  queries: List[List[int]]) -> List[int]:
    vowels = set("aeiou")
    n = len(tokens)
    prefix = [0] * (n + 1)
    for i, s in enumerate(tokens):
        good = 1 if (s[0] in vowels and s[-1] in vowels) else 0
        prefix[i + 1] = prefix[i] + good
    return [prefix[r + 1] - prefix[l] for l, r in queries]
JavaScript
JAVASCRIPT
/**
 * @param {string[]} tokens
 * @param {number[][]} queries
 * @return {number[]}
 */
function countVowelBoundedInRanges(tokens, queries) {

#### cons
t isVowel = (c) => c === 'a' || c === 'e' || c === 'i' || c === 'o' || c === 'u';

#### cons
t n = tokens.length;

#### cons
t prefix = new Int32Array(n + 1);
    for (let i = 0; i < n; i++) {

#### cons
t s = tokens[i];

#### cons
t good = isVowel(s[0]) && isVowel(s[s.length - 1]) ? 1 : 0;
        prefix[i + 1] = prefix[i] + good;
    }

#### cons
t ans = new Array(queries.length);
    for (let q = 0; q < queries.length; q++) {

#### cons
t [l, r] = queries[q];
        ans[q] = prefix[r + 1] - prefix[l];
    }
    return ans;
}
Java
JAVA
class Solution {
    public int[] countVowelBoundedInRanges(String[] tokens, int[][] queries) {
        int n = tokens.length;
        int[] prefix = new int[n + 1];
        for (int i = 0; i < n; i++) {
            String s = tokens[i];
            boolean good = isVowel(s.charAt(0)) && isVowel(s.charAt(s.length() - 1));
            prefix[i + 1] = prefix[i] + (good ? 1 : 0);
        }
        int[] ans = new int[queries.length];
        for (int i = 0; i < queries.length; i++) {
            int l = queries[i][0], r = queries[i][1];
            ans[i] = prefix[r + 1] - prefix[l];
        }
        return ans;
    }

    private boolean isVowel(char c) {
        return c == 'a' || c == 'e' || c == 'i' || c == 'o' || c == 'u';
    }
}
C++
CPP
#include <vector>
#include <string>
using namespace std;

class Solution {
public:
    vector<int> countVowelBoundedInRanges(vector<string>& tokens,
                                          vector<vector<int>>& queries) {
        int n = tokens.size();
        vector<int> prefix(n + 1, 0);
        for (int i = 0; i < n; ++i) {

#### cons
t string& s = tokens[i];
            int good = isVowel(s.front()) && isVowel(s.back());
            prefix[i + 1] = prefix[i] + good;
        }
        vector<int> ans;
        ans.reserve(queries.size());
        for (const auto& q : queries)
            ans.push_back(prefix[q[1] + 1] - prefix[q[0]]);
        return ans;
    }
private:
    bool isVowel(char c) {
        return c == 'a' || c == 'e' || c == 'i' || c == 'o' || c == 'u';
    }
};
C
C
#include <stdlib.h>
#include <string.h>

static int isVowel(char c) {
    return c == 'a' || c == 'e' || c == 'i' || c == 'o' || c == 'u';
}

/* queries is a flattened array of size queriesSize*2: [l0,r0,l1,r1,...].
   Returns a malloc'd array of length queriesSize; caller frees. */
int* countVowelBoundedInRanges(char** tokens, int tokensSize,
                               int* queries, int queriesSize) {
    /* prefix[i] = number of qualifying strings among tokens[0..i-1] */
    int* prefix = (int*)malloc(sizeof(int) * (tokensSize + 1));
    prefix[0] = 0;
    for (int i = 0; i < tokensSize; i++) {

#### cons
t char* s = tokens[i];
        int len = (int)strlen(s);
        int good = isVowel(s[0]) && isVowel(s[len - 1]);
        prefix[i + 1] = prefix[i] + good;
    }

    int* ans = (int*)malloc(sizeof(int) * queriesSize);
    for (int q = 0; q < queriesSize; q++) {
        int l = queries[2 * q], r = queries[2 * q + 1];
        ans[q] = prefix[r + 1] - prefix[l];
    }
    free(prefix);
    return ans;
}

#### Follow-up
questions
What changes if the strings could be updated between queries (point updates)? (Answer: switch from a static prefix-sum array to a Fenwick/Binary Indexed Tree for O(log n) update and query.)
How would you extend it to count strings whose first and last characters are equal, or that both start and end with the same specific vowel?
If queries were given offline all at once, could you exploit that ordering, and does it change the asymptotic cost versus the online prefix-sum solution?
How would you generalize to a 2D grid of strings with rectangular range queries?

