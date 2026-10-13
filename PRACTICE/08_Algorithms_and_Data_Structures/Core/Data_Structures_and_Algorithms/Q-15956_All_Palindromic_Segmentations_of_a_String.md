---
type: "practice"
difficulty: "Other"
---

# All Palindromic Segmentations of a String

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Backtracking](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Backtracking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a string text made up of lowercase English letters. A segmentation of text is a way of splitting it into a sequence of consecutive, non-empty pieces such that concatenating the pieces in order reproduces text exactly. A piece is called symmetric if it reads the same forwards and backwards (that is, it is a palindrome).

Return every possible segmentation of text in which all of its pieces are symmetric. Each segmentation should be returned as an ordered list of its pieces (left to right). The collection of segmentations may be returned in any order.

Note that every single character is trivially symmetric, so the segmentation that cuts text into individual characters is always valid; there may be additional, coarser segmentations as well.

Input

text: a string of lowercase English letters.

Output

A list of segmentations; each segmentation is a list of strings whose pieces are all palindromes and whose concatenation equals text.

#### Cons
traints

1 <= text.length <= 16
text consists only of lowercase English letters (a–z).

#### Example
s & test cases
TEXT
Input: text = "boob" -> Output: [["b","o","o","b"], ["b","oo","b"], ["boob"]]
Why: three ways — all singles, the middle "oo" merged, or the whole word as one palindrome.

Input: text = "wow" -> Output: [["w","o","w"], ["wow"]]
Why: the singles split always works, and the entire string is itself a palindrome.

Input: text = "q" -> Output: [["q"]]
Why: a single character is a palindrome, so the only segmentation is itself. (Minimum-length edge case.)

Input: text = "abc" -> Output: [["a","b","c"]]
Why: no substring of length >= 2 is symmetric, so only the all-singles segmentation is valid.

Input: text = "aaa" -> Output: [["a","a","a"], ["a","aa"], ["aa","a"], ["aaa"]]
Why: repeated characters create many palindromic substrings, so several segmentations exist (duplicates edge case).

#### Approach

This is a classic backtracking (DFS) enumeration. Walk left to right: at position start, try every prefix text[start..end]; if that prefix is a palindrome, fix it as the next piece and recurse from end+1. When start reaches the end of the string, the current path is a complete valid segmentation, so record a copy of it. Undo the last choice (pop) before trying a longer prefix — this is the backtracking step. The palindrome test is a simple two-pointer scan from both ends. The pattern is "generate all partitions via choose-recurse-unchoose," pruned by the palindrome constraint. For very hot inputs one can precompute an n×n isPalindrome table in O(n^2) so each check is O(1), but the exponential number of outputs dominates regardless.

Complexity

Time: O(n · 2^n) — there are up to 2^(n-1) ways to place cut points, and reconstructing/copying each valid segmentation costs O(n). Space: O(n) auxiliary for the recursion stack and the current path (excluding the output list itself), where n = text.length.

Python
PYTHON
from typing import List

class Solution:
    def allPalindromeSegmentations(self, text: str) -> List[List[str]]:
        ans: List[List[str]] = []

        def is_palindrome(piece: str) -> bool:
            return piece == piece[::-1]

        def dfs(start: int, path: List[str]) -> None:
            if start == len(text):
                ans.append(path[:])
                return
            for end in range(start, len(text)):
                piece = text[start:end + 1]
                if is_palindrome(piece):
                    path.append(piece)
                    dfs(end + 1, path)
                    path.pop()

        dfs(0, [])
        return ans
JavaScript
JAVASCRIPT
function allPalindromeSegmentations(text) {

#### cons
t ans = [];

    function isPalindrome(l, r) {
        while (l < r) {
            if (text[l++] !== text[r--]) return false;
        }
        return true;
    }

    function dfs(start, path) {
        if (start === text.length) {
            ans.push([...path]);
            return;
        }
        for (let end = start; end < text.length; end++) {
            if (isPalindrome(start, end)) {
                path.push(text.slice(start, end + 1));
                dfs(end + 1, path);
                path.pop();
            }
        }
    }

    dfs(0, []);
    return ans;
}
Java
JAVA
import java.util.ArrayList;
import java.util.List;

class Solution {
  public List<List<String>> allPalindromeSegmentations(String text) {
    List<List<String>> ans = new ArrayList<>();
    dfs(text, 0, new ArrayList<>(), ans);
    return ans;
  }

  private void dfs(String s, int start, List<String> path, List<List<String>> ans) {
    if (start == s.length()) {
      ans.add(new ArrayList<>(path));
      return;
    }
    for (int end = start; end < s.length(); ++end)
      if (isPalindrome(s, start, end)) {
        path.add(s.substring(start, end + 1));
        dfs(s, end + 1, path, ans);
        path.remove(path.size() - 1);
      }
  }

  private boolean isPalindrome(String s, int l, int r) {
    while (l < r)
      if (s.charAt(l++) != s.charAt(r--))
        return false;
    return true;
  }
}
C++
CPP
#include <string>
#include <vector>
using namespace std;

class Solution {
 public:
  vector<vector<string>> allPalindromeSegmentations(const string& text) {
    vector<vector<string>> ans;
    vector<string> path;
    dfs(text, 0, path, ans);
    return ans;
  }

 private:
  void dfs(const string& s, int start, vector<string>& path,
           vector<vector<string>>& ans) {
    if (start == (int)s.length()) {
      ans.push_back(path);
      return;
    }
    for (int end = start; end < (int)s.length(); ++end)
      if (isPalindrome(s, start, end)) {
        path.push_back(s.substr(start, end - start + 1));
        dfs(s, end + 1, path, ans);
        path.pop_back();
      }
  }

  bool isPalindrome(const string& s, int l, int r) {
    while (l < r)
      if (s[l++] != s[r--])
        return false;
    return true;
  }
};
C
C
#include <stdio.h>
#include <string.h>
#include <stdbool.h>

static bool isPalindrome(const char* s, int l, int r) {
    while (l < r)
        if (s[l++] != s[r--])
            return false;
    return true;
}

/* Enumerate and print every all-palindrome segmentation of s[start..n-1]. */
static void dfs(const char* s, int n, int start, char parts[][32], int depth) {
    if (start == n) {
        for (int k = 0; k < depth; ++k)
            printf(k ? " %s" : "[%s", parts[k]);
        printf("]\n");
        return;
    }
    for (int end = start; end < n; ++end) {
        if (isPalindrome(s, start, end)) {
            int len = end - start + 1;
            memcpy(parts[depth], s + start, len);
            parts[depth][len] = '\0';
            dfs(s, n, end + 1, parts, depth + 1);
        }
    }
}

void allPalindromeSegmentations(const char* s) {
    int n = (int)strlen(s);
    char parts[32][32];
    dfs(s, n, 0, parts, 0);
}

int main(void) {
    allPalindromeSegmentations("boob");
    return 0;
}

#### Follow-up
questions
Instead of listing every segmentation, return only the count of valid all-palindrome segmentations — can you do it without materializing them all?
Return just one segmentation that uses the fewest pieces (equivalently, the minimum number of cuts).
Add an O(n^2) precomputed palindrome table and explain how it changes the per-check cost versus the overall complexity.
How would memoization help if you only needed to know whether any all-palindrome segmentation exists under some extra constraint (e.g., each piece length >= 2)?

