---
type: "practice"
difficulty: "Other"
---

# All Possible Values of a Fully Parenthesized Expression

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Divide and Conquer](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Divide_and_Conquer.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a string formula that represents an arithmetic expression made up of non-negative integers and the binary operators +, -, and *. The expression is written without any parentheses.

By inserting parentheses in every syntactically valid way, you can force the operators to be applied in different orders, and each order can produce a different numeric value. Your task is to compute every value that can be obtained by fully parenthesizing the expression in all possible ways.

Return the list of all such values. The list may be returned in any order, and it may contain duplicate values (two different groupings can legitimately yield the same number).

Notes:

The expression contains only the digits 0–9 and the operators +, -, *.
Numbers may have more than one digit; parentheses may only be placed around whole sub-expressions, never in the middle of a number.
The inputs are chosen so that every intermediate and final value fits in a signed 32-bit integer and the number of distinct results does not exceed 10,000.

#### Cons
traints

1 <= formula.length <= 20
formula consists only of digits and the operators +, -, *.
Every integer literal in formula lies in the range [0, 99].
No integer literal carries an explicit leading sign (+ / -).

#### Example
s & test cases
TEXT
Input: formula = "3+2*4" -> Output: [20, 11]
  Why: ((3+2)*4)=20 and (3+(2*4))=11. Two operators give two groupings (order may differ).

Input: formula = "5-2-1" -> Output: [2, 4]
  Why: ((5-2)-1)=2 and (5-(2-1))=4. Subtraction is not associative, so grouping matters.

Input: formula = "7" -> Output: [7]
  Why: A single number with no operator has exactly one grouping — itself.

Input: formula = "2*3-4*5" -> Output: [-34, -14, -10, -10, 10]
  Why: The five distinct parenthesizations of three operators; note the duplicate -10 is kept
  because two different groupings both evaluate to -10.

Input: formula = "10-20*3" -> Output: [-30, -50]
  Why: Multi-digit operands; ((10-20)*3)=-30 and (10-(20*3))=-50.

Input: formula = "0*0" -> Output: [0]
  Why: Single operator, one grouping; verifies zero handling.

#### Approach

This is a divide-and-conquer problem. Every top-level operator can be the last operation applied to the whole expression. Pick each operator position as the split point: recursively compute all values the left substring can produce and all values the right substring can produce, then combine every left value with every right value using that operator. The union over all split points is the answer. A single number (no operator) is the base case and returns itself. Because many sub-expressions repeat, memoizing on the substring (or its index range) turns exponential recomputation into a polynomial number of distinct subproblems — the same optimal-substructure idea behind interval dynamic programming.

Complexity

Time: O(C(n)) where n is the number of operators and C is the Catalan number — the count of distinct parenthesizations — but memoization collapses overlapping sub-expressions so each distinct substring is evaluated once; effectively bounded by O(number-of-results * length). Space: O(n^2) distinct substrings cached, each holding up to O(results) values. For the given bound (length <= 20, <= 10^4 results) this is comfortably fast.

Python
PYTHON
from functools import lru_cache
from typing import List

class Solution:
    def computeAllResults(self, formula: str) -> List[int]:
        @lru_cache(maxsize=None)
        def solve(s: str) -> tuple:
            results = []
            for i, c in enumerate(s):
                if c in '+-*':
                    for a in solve(s[:i]):
                        for b in solve(s[i + 1:]):
                            if c == '+':
                                results.append(a + b)
                            elif c == '-':
                                results.append(a - b)
                            else:
                                results.append(a * b)
            if not results:          # base case: a single number
                results.append(int(s))
            return tuple(results)

        return list(solve(formula))
JavaScript
JAVASCRIPT
function computeAllResults(formula) {

#### cons
t memo = new Map();

    function solve(s) {
        if (memo.has(s)) return memo.get(s);

#### cons
t results = [];
        for (let i = 0; i < s.length; i++) {

#### cons
t c = s[i];
            if (c === '+' || c === '-' || c === '*') {

#### cons
t left = solve(s.slice(0, i));

#### cons
t right = solve(s.slice(i + 1));
                for (const a of left) {
                    for (const b of right) {
                        if (c === '+') results.push(a + b);
                        else if (c === '-') results.push(a - b);
                        else results.push(a * b);
                    }
                }
            }
        }
        if (results.length === 0) results.push(parseInt(s, 10)); // single number
        memo.set(s, results);
        return results;
    }

    return solve(formula);
}
Java
JAVA
import java.util.*;

class Solution {
    public List<Integer> computeAllResults(String formula) {
        return solve(formula, new HashMap<>());
    }

    private List<Integer> solve(String s, Map<String, List<Integer>> memo) {
        if (memo.containsKey(s)) return memo.get(s);

        List<Integer> ans = new ArrayList<>();
        for (int i = 0; i < s.length(); i++) {
            char c = s.charAt(i);
            if (c == '+' || c == '-' || c == '*') {
                List<Integer> left = solve(s.substring(0, i), memo);
                List<Integer> right = solve(s.substring(i + 1), memo);
                for (int a : left)
                    for (int b : right) {
                        if (c == '+')      ans.add(a + b);
                        else if (c == '-') ans.add(a - b);
                        else               ans.add(a * b);
                    }
            }
        }
        if (ans.isEmpty()) ans.add(Integer.parseInt(s)); // base case: a single number
        memo.put(s, ans);
        return ans;
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
  vector<int> computeAllResults(string formula) {
    unordered_map<string, vector<int>> memo;
    return solve(formula, memo);
  }

 private:
  vector<int> solve(const string& s, unordered_map<string, vector<int>>& memo) {
    auto it = memo.find(s);
    if (it != memo.end()) return it->second;

    vector<int> ans;
    for (int i = 0; i < (int)s.size(); ++i) {
      char c = s[i];
      if (c == '+' || c == '-' || c == '*') {
        vector<int> left = solve(s.substr(0, i), memo);
        vector<int> right = solve(s.substr(i + 1), memo);
        for (int a : left)
          for (int b : right) {
            if (c == '+')      ans.push_back(a + b);
            else if (c == '-') ans.push_back(a - b);
            else               ans.push_back(a * b);
          }
      }
    }
    if (ans.empty()) ans.push_back(stoi(s));  // base case: a single number
    return memo[s] = ans;
  }
};
C
C
#include <stdlib.h>
#include <string.h>

/* Returns a freshly allocated array of every value obtainable by parenthesizing
 * the substring formula[lo, hi). *outCnt receives the number of values.
 * memo/memoCnt cache results keyed by (lo, hi) to avoid recomputation. */
static int* solve(const char* s, int lo, int hi, int** memo, int* memoCnt,
                  int stride, int* outCnt) {
    int key = lo * stride + hi;
    if (memoCnt[key] != -1) {
        *outCnt = memoCnt[key];
        return memo[key];
    }

    int hasOp = 0;
    for (int i = lo; i < hi; ++i) {
        char c = s[i];
        if (c == '+' || c == '-' || c == '*') { hasOp = 1; break; }
    }

    int cap = 0, cnt = 0;
    int* res = NULL;

    if (!hasOp) {                 /* base case: a plain number */
        int val = 0;
        for (int i = lo; i < hi; ++i) val = val * 10 + (s[i] - '0');
        res = (int*)malloc(sizeof(int));
        res[0] = val;
        cnt = 1;
    } else {                      /* split on each operator */
        for (int i = lo; i < hi; ++i) {
            char c = s[i];
            if (c != '+' && c != '-' && c != '*') continue;
            int lc, rc;
            int* left  = solve(s, lo, i, memo, memoCnt, stride, &lc);
            int* right = solve(s, i + 1, hi, memo, memoCnt, stride, &rc);
            for (int a = 0; a < lc; ++a)
                for (int b = 0; b < rc; ++b) {
                    int v;
                    if (c == '+')      v = left[a] + right[b];
                    else if (c == '-') v = left[a] - right[b];
                    else               v = left[a] * right[b];
                    if (cnt == cap) {
                        cap = cap ? cap * 2 : 4;
                        res = (int*)realloc(res, cap * sizeof(int));
                    }
                    res[cnt++] = v;
                }
        }
    }

    memo[key] = res;
    memoCnt[key] = cnt;
    *outCnt = cnt;
    return res;
}

int* computeAllResults(char* formula, int* returnSize) {
    int n = (int)strlen(formula);
    int stride = n + 1;
    int total = n * stride;
    int** memo = (int**)malloc(sizeof(int*) * total);
    int* memoCnt = (int*)malloc(sizeof(int) * total);
    for (int i = 0; i < total; ++i) { memo[i] = NULL; memoCnt[i] = -1; }

    int cnt = 0;
    int* cached = solve(formula, 0, n, memo, memoCnt, stride, &cnt);

    int* out = (int*)malloc(sizeof(int) * (cnt ? cnt : 1));
    memcpy(out, cached, sizeof(int) * cnt);
    *returnSize = cnt;

    for (int i = 0; i < total; ++i) free(memo[i]);
    free(memo);
    free(memoCnt);
    return out;
}

#### Follow-up
questions
Instead of listing every value, return only the count of distinct results — how does that change memoization?
Extend the operator set to include integer division /; how do you guard against divide-by-zero across groupings?
Return the actual parenthesization string that yields the maximum value, not just the value.
If the same expression is queried many times, how would you precompute or cache across queries?

