---
type: "practice"
difficulty: "Other"
---

# Samples Reachable Within Two Character Changes

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ String](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/String.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given two lists of strings, samples and catalog. Every string in both lists has the same fixed length n and is made up only of lowercase English letters.

A single change lets you pick one position of a string and replace its letter with any other lowercase letter. A sample string is considered acceptable if, using at most two changes, it can be transformed so that it becomes exactly equal to some string in catalog.

Return the list of all strings from samples that are acceptable, in the same order in which they appear in samples. Because every string has the same length, the smallest number of changes needed to turn one string into another is simply the number of positions at which they differ (their Hamming distance); a sample is acceptable when that count is <= 2 for at least one catalog string.

Input

samples: a list of equal-length lowercase strings.
catalog: a list of lowercase strings, each of the same length as the samples.

Output

A list containing every string from samples (preserving order and duplicates) that lies within two changes of at least one string in catalog.

#### Cons
traints

1 <= samples.length, catalog.length <= 100
n == samples[i].length == catalog[j].length
1 <= n <= 100
All strings consist only of lowercase English letters.

#### Example
s & test cases
TEXT
- Input: `samples = ["bake","song","rice"], catalog = ["cake","done"]` -> Output: `["bake","song"]`
  Why: "bake"→"cake" is 1 change; "song"→"done" is 2 changes (s→d, g→e); "rice" differs from both catalog words in 3 positions, so it is excluded.
- Input: `samples = ["yes"], catalog = ["not"]` -> Output: `[]`
  Why: "yes" differs from "not" at all three positions, which is more than two changes, so nothing qualifies (empty-result edge case).
- Input: `samples = ["abcd"], catalog = ["abcd"]` -> Output: `["abcd"]`
  Why: an identical catalog word needs 0 changes, which is within the limit.
- Input: `samples = ["aaaa","bbbb"], catalog = ["aacc"]` -> Output: `["aaaa"]`
  Why: "aaaa" vs "aacc" differ in 2 positions (accepted); "bbbb" vs "aacc" differ in 4 positions (rejected).
- Input: `samples = ["mm","mm"], catalog = ["mm"]` -> Output: `["mm","mm"]`
  Why: duplicates in `samples` are each judged independently and both are kept when acceptable.
- Input: `samples = ["z"], catalog = ["a","z"]` -> Output: `["z"]`
  Why: single-character strings; "z" matches the catalog word "z" with 0 changes (minimum length edge case).

#### Approach

Two equal-length strings need exactly (number of differing positions) single-character changes to be made equal — that is their Hamming distance. So a sample is acceptable iff its Hamming distance to some catalog word is at most 2. The direct optimal solution for these bounds is a triple loop: for each sample, scan each catalog word and count mismatches, breaking early the moment the mismatch count exceeds 2 (never comparing beyond what matters). Keep the sample as soon as any catalog word satisfies the bound, preserving input order. This is a straightforward brute-force / Hamming-distance pattern; the early exit keeps constant per-pair work small.

Complexity

Time: O(S * C * n) where S = samples.length, C = catalog.length, and n is the common string length — for each sample we compare against each catalog word position by position, with an early cutoff once the difference count passes 2. Space: O(1) extra beyond the output list (only a running counter per comparison). Given the constraints (S, C <= 100, n <= 100) this is at most ~10^6 character comparisons, comfortably fast.

Python
PYTHON
from typing import List

class Solution:
    def acceptableSamples(self, samples: List[str], catalog: List[str]) -> List[str]:
        def within_two(a: str, b: str) -> bool:
            diff = 0
            for x, y in zip(a, b):
                if x != y:
                    diff += 1
                    if diff > 2:
                        return False
            return True

        result = []
        for s in samples:
            if any(within_two(s, ref) for ref in catalog):
                result.append(s)
        return result
JavaScript
JAVASCRIPT
function acceptableSamples(samples, catalog) {

#### cons
t withinTwo = (a, b) => {
        let diff = 0;
        for (let i = 0; i < a.length; ++i) {
            if (a[i] !== b[i] && ++diff > 2) return false;
        }
        return true;
    };

#### cons
t result = [];
    for (const s of samples) {
        if (catalog.some(ref => withinTwo(s, ref))) {
            result.push(s);
        }
    }
    return result;
}
Java
JAVA
import java.util.ArrayList;
import java.util.List;

class Solution {
  public List<String> acceptableSamples(String[] samples, String[] catalog) {
    List<String> result = new ArrayList<>();
    for (String s : samples) {
      for (String ref : catalog) {
        int diff = 0;
        for (int i = 0; i < s.length() && diff <= 2; ++i)
          if (s.charAt(i) != ref.charAt(i)) ++diff;
        if (diff <= 2) {
          result.add(s);
          break;
        }
      }
    }
    return result;
  }
}
C++
CPP
#include <string>
#include <vector>
using namespace std;

class Solution {
 public:
  vector<string> acceptableSamples(vector<string>& samples, vector<string>& catalog) {
    vector<string> result;
    for (const string& s : samples) {
      for (const string& ref : catalog) {
        int diff = 0;
        for (int i = 0; i < (int)s.size() && diff <= 2; ++i)
          if (s[i] != ref[i]) ++diff;
        if (diff <= 2) {            // within two changes of this catalog word
          result.push_back(s);
          break;                    // one match is enough; keep order
        }
      }
    }
    return result;
  }
};
C
C
#include <stdlib.h>

static int hammingWithinTwo(const char* a, const char* b) {
    int diff = 0;
    for (int i = 0; a[i] != '\0'; ++i) {   /* both strings share the same length */
        if (a[i] != b[i]) {
            if (++diff > 2) return 3;       /* early exit once it exceeds the limit */
        }
    }
    return diff;
}

char** acceptableSamples(char** samples, int samplesSize,
                         char** catalog, int catalogSize, int* returnSize) {
    char** result = (char**)malloc(sizeof(char*) * samplesSize);
    int cnt = 0;
    for (int i = 0; i < samplesSize; ++i) {
        for (int j = 0; j < catalogSize; ++j) {
            if (hammingWithinTwo(samples[i], catalog[j]) <= 2) {
                result[cnt++] = samples[i];
                break;
            }
        }
    }
    *returnSize = cnt;
    return result;
}

#### Follow-up
questions
If the catalog contained tens of thousands of words, how would a trie built over the catalog with a bounded-mismatch (allow-up-to-2) DFS reduce the per-sample cost versus scanning every word?
Generalize the allowed edit budget from 2 to an arbitrary k; does the brute-force still work and what is its complexity?
What if strings could have different lengths and true edit (insert/delete/replace) distance were required instead of Hamming distance?
How would you return, for each acceptable sample, the actual minimum number of changes and which catalog word it matched?

