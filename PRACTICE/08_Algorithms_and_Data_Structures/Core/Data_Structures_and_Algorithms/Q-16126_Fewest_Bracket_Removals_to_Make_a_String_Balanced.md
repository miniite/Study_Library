---
type: "practice"
difficulty: "Other"
---

# Fewest Bracket Removals to Make a String Balanced

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Backtracking](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Backtracking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a string expr composed of lowercase English letters and the two round-bracket characters ( and ).

A string is balanced when every opening bracket ( has a matching closing bracket ) that appears somewhere after it, and there are no unmatched closing brackets. Letters are always allowed and never affect balance.

Remove the minimum possible number of bracket characters (never a letter) so that the remaining string becomes balanced. Because several different removals may all reach a balanced string using that same minimum count, return every distinct balanced string that can be produced with the minimum number of removals.

Return the results in any order. The empty string "" counts as balanced, so if all brackets must be removed and no letters remain, the answer is [""].

Input

expr: a string of lowercase letters and the characters ( and ).

Output

A list of unique strings, each obtainable by deleting the fewest bracket characters needed to make expr balanced.

#### Cons
traints

1 <= expr.length <= 25
expr consists only of lowercase English letters and the characters ( and ).
There are at most 20 bracket characters in expr.

#### Example
s & test cases
TEXT
- Input: expr = "(w)(v))" -> Output: ["(w)(v)","(w(v))"]  — one extra closing bracket; two different single removals give two distinct balanced strings.
- Input: expr = ")x(" -> Output: ["x"]  — both brackets are unmatched, so both are removed, leaving just the letter.
- Input: expr = "abc" -> Output: ["abc"]  — no brackets at all; already balanced, zero removals.
- Input: expr = "(((" -> Output: [""]  — every opening bracket is unmatched; all three are removed and nothing remains.
- Input: expr = "(a)((b)" -> Output: ["(a)(b)"]  — one redundant opening bracket; removing either of the two adjacent `(` yields the same single balanced string (duplicates collapse).

#### Approach

First, one linear pass computes exactly how many opening brackets l and closing brackets r are surplus (a running balance: every unmatchable ) bumps r, and whatever ( are left unmatched at the end are l). That is the provable minimum number of removals. A depth-first search then deletes brackets one at a time, decrementing l when it drops a ( and r when it drops a ), and only records a string once the budget hits zero and a validity check passes. Two tricks keep it clean and duplicate-free: recursion always resumes from the current index (so each bracket is considered for deletion once), and within a run of identical characters only the first is ever deleted (if (i > start && s[i] == s[i-1]) continue), which prevents generating the same string by two different deletion orders. Pattern: bounded backtracking / DFS with a precomputed removal budget.

Complexity

Time: O(2^n) in the worst case, where n is the string length — in principle every bracket can be kept or removed — but the exact removal budget (l opens, r closes) plus the "skip a bracket identical to its predecessor" rule prune the search dramatically, so real inputs explore far fewer branches. Space: O(n) for recursion depth and the working copies, excluding the output list itself.

Python
PYTHON
from typing import List

class Solution:
    def removeInvalidParentheses(self, expr: str) -> List[str]:
        def counts_to_remove(s: str):
            l = r = 0
            for c in s:
                if c == '(':
                    l += 1
                elif c == ')':
                    if l == 0:
                        r += 1
                    else:
                        l -= 1
            return l, r

        def is_valid(s: str) -> bool:
            count = 0
            for c in s:
                if c == '(':
                    count += 1
                elif c == ')':
                    count -= 1
                if count < 0:
                    return False
            return count == 0

        ans: List[str] = []

        def dfs(s: str, start: int, l: int, r: int) -> None:
            if l == 0 and r == 0:
                if is_valid(s):
                    ans.append(s)
                return
            for i in range(start, len(s)):
                if i > start and s[i] == s[i - 1]:
                    continue  # skip duplicate removals
                if l > 0 and s[i] == '(':
                    dfs(s[:i] + s[i + 1:], i, l - 1, r)
                elif r > 0 and s[i] == ')':
                    dfs(s[:i] + s[i + 1:], i, l, r - 1)

        l, r = counts_to_remove(expr)
        dfs(expr, 0, l, r)
        return ans
JavaScript
JAVASCRIPT
function removeInvalidParentheses(expr) {

#### cons
t ans = [];

#### cons
t isValid = (s) => {
        let count = 0;
        for (const c of s) {
            if (c === '(') count++;
            else if (c === ')') { if (--count < 0) return false; }
        }
        return count === 0;
    };

#### cons
t dfs = (s, start, l, r) => {
        if (l === 0 && r === 0) {
            if (isValid(s)) ans.push(s);
            return;
        }
        for (let i = start; i < s.length; i++) {
            if (i > start && s[i] === s[i - 1]) continue;  // skip duplicate removals
            if (l > 0 && s[i] === '(')
                dfs(s.slice(0, i) + s.slice(i + 1), i, l - 1, r);
            else if (r > 0 && s[i] === ')')
                dfs(s.slice(0, i) + s.slice(i + 1), i, l, r - 1);
        }
    };

    let l = 0, r = 0;
    for (const c of expr) {
        if (c === '(') l++;
        else if (c === ')') { if (l === 0) r++; else l--; }
    }
    dfs(expr, 0, l, r);
    return ans;
}
Java
JAVA
import java.util.ArrayList;
import java.util.List;

class Solution {
  public List<String> removeInvalidParentheses(String expr) {
    int l = 0, r = 0;   // '(' to delete, ')' to delete
    for (final char c : expr.toCharArray()) {
      if (c == '(') ++l;
      else if (c == ')') { if (l == 0) ++r; else --l; }
    }
    List<String> ans = new ArrayList<>();
    dfs(expr, 0, l, r, ans);
    return ans;
  }

  private void dfs(final String s, int start, int l, int r, List<String> ans) {
    if (l == 0 && r == 0) {
      if (isValid(s)) ans.add(s);
      return;
    }
    for (int i = start; i < s.length(); ++i) {
      if (i > start && s.charAt(i) == s.charAt(i - 1)) continue;  // skip dup removals
      if (l > 0 && s.charAt(i) == '(')
        dfs(s.substring(0, i) + s.substring(i + 1), i, l - 1, r, ans);
      else if (r > 0 && s.charAt(i) == ')')
        dfs(s.substring(0, i) + s.substring(i + 1), i, l, r - 1, ans);
    }
  }

  private boolean isValid(final String s) {
    int count = 0;
    for (final char c : s.toCharArray()) {
      if (c == '(') ++count;
      else if (c == ')') --count;
      if (count < 0) return false;
    }
    return count == 0;
  }
}
C++
CPP
#include <string>
#include <vector>
using namespace std;

class Solution {
 public:
  vector<string> removeInvalidParentheses(string expr) {
    int l = 0, r = 0;   // '(' to delete, ')' to delete
    for (const char c : expr)
      if (c == '(') ++l;
      else if (c == ')') { if (l == 0) ++r; else --l; }

    vector<string> ans;
    dfs(expr, 0, l, r, ans);
    return ans;
  }

 private:
  bool isValid(const string& s) {
    int count = 0;
    for (const char c : s) {
      if (c == '(') ++count;
      else if (c == ')') --count;
      if (count < 0) return false;
    }
    return count == 0;
  }

  void dfs(const string& s, int start, int l, int r, vector<string>& ans) {
    if (l == 0 && r == 0) {
      if (isValid(s)) ans.push_back(s);
      return;
    }
    for (int i = start; i < (int)s.length(); ++i) {
      if (i > start && s[i] == s[i - 1]) continue;   // skip duplicate removals
      if (l > 0 && s[i] == '(')
        dfs(s.substr(0, i) + s.substr(i + 1), i, l - 1, r, ans);
      else if (r > 0 && s[i] == ')')
        dfs(s.substr(0, i) + s.substr(i + 1), i, l, r - 1, ans);
    }
  }
};
C
C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

static int isValid(const char* s) {
    int count = 0;
    for (int i = 0; s[i]; ++i) {
        if (s[i] == '(') ++count;
        else if (s[i] == ')') { if (--count < 0) return 0; }
    }
    return count == 0;
}

static void dfs(const char* s, int start, int l, int r,
                char*** ans, int* ansSize, int* ansCap) {
    if (l == 0 && r == 0) {
        if (isValid(s)) {
            if (*ansSize == *ansCap) {
                *ansCap = (*ansCap == 0) ? 8 : (*ansCap * 2);
                *ans = (char**)realloc(*ans, (*ansCap) * sizeof(char*));
            }
            (*ans)[(*ansSize)++] = strdup(s);
        }
        return;
    }
    int n = (int)strlen(s);
    for (int i = start; i < n; ++i) {
        if (i > start && s[i] == s[i - 1]) continue;   /* skip duplicate removals */
        if ((l > 0 && s[i] == '(') || (r > 0 && s[i] == ')')) {
            char* next = (char*)malloc(n);             /* (n-1) chars + '\0' */
            memcpy(next, s, i);
            strcpy(next + i, s + i + 1);
            if (s[i] == '(') dfs(next, i, l - 1, r, ans, ansSize, ansCap);
            else             dfs(next, i, l, r - 1, ans, ansSize, ansCap);
            free(next);
        }
    }
}

char** removeInvalidParentheses(char* expr, int* returnSize) {
    int l = 0, r = 0;
    for (int i = 0; expr[i]; ++i) {
        if (expr[i] == '(') ++l;
        else if (expr[i] == ')') { if (l == 0) ++r; else --l; }
    }
    char** ans = NULL;
    int ansSize = 0, ansCap = 0;
    dfs(expr, 0, l, r, &ans, &ansSize, &ansCap);
    *returnSize = ansSize;
    return ans;
}

#### Follow-up
questions
What if you only need the minimum number of removals, not the actual strings? (Just return l + r from the counting pass.)
Solve it with level-by-level BFS instead: why does the first BFS level that yields any valid string guarantee the minimum removals?
Generalize to multiple bracket types ()[]{} where matching must respect type and nesting.
How would you cap or stream the output if the number of valid results could be large?

