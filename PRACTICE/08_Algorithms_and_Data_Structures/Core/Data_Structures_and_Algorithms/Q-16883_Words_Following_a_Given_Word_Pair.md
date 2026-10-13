---
type: "practice"
difficulty: "Other"
---

# Words Following a Given Word Pair

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Sliding Window](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Sliding_Window.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a sentence as a single string made up of lowercase words separated by single spaces, together with two words lead and mid.

Scan the sentence for every position where the word lead is immediately followed by the word mid. For each such position, look at the word that comes immediately after mid — call it the successor — and collect it. In other words, for every occurrence of the three-word run "lead mid successor", record successor.

Return the list of all such successor words, in the order they appear in the sentence. Matches may overlap: the successor of one match can itself be the lead of a later match. If lead mid occurs at the very end of the sentence with no word after it, it produces nothing.

Input

sentence: a string of lowercase words separated by single spaces, with no leading or trailing spaces.
lead: a lowercase word.
mid: a lowercase word.

Output

A list of the successor words, in order of occurrence. The list is empty if the pair lead mid never has a word after it.

#### Cons
traints

1 <= length of sentence <= 1000
sentence consists of lowercase English letters and single spaces only.
All words in sentence are separated by exactly one space.
1 <= length of lead, length of mid <= 10
lead and mid consist of lowercase English letters.

#### Example
s & test cases
TEXT
- Input: `sentence = "tom is a kind boy and jane is a kind teacher"`, `lead = "a"`, `mid = "kind"` -> Output: `["boy", "teacher"]`
  Why: the pair `a kind` occurs twice and each occurrence has a following word (normal case).
- Input: `sentence = "go left go left then right"`, `lead = "go"`, `mid = "left"` -> Output: `["go", "then"]`
  Why: overlapping matches — the word after the first pair (`go`) is itself the `lead` of the second pair.
- Input: `sentence = "one two three"`, `lead = "one"`, `mid = "two"` -> Output: `["three"]`
  Why: a single occurrence in the middle of the sentence.
- Input: `sentence = "alpha beta"`, `lead = "alpha"`, `mid = "beta"` -> Output: `[]`
  Why: the pair appears but sits at the very end with no following word (edge case).
- Input: `sentence = "dog cat dog fish"`, `lead = "dog"`, `mid = "bird"` -> Output: `[]`
  Why: the pair `dog bird` never appears, so there are no successors.
- Input: `sentence = "a a a a"`, `lead = "a"`, `mid = "a"` -> Output: `["a", "a"]`
  Why: repeated identical words — pairs starting at indices 0 and 1 each have a following word (duplicates edge case).

#### Approach

Split the sentence into its words and slide a fixed window of size three across them. At each index i, if words[i] == lead and words[i+1] == mid, then words[i+2] is a successor of the pair, so append it to the answer. Because the window advances by one each step, overlapping matches are handled naturally — the successor of one match can start the next pair. The C++ solution expresses the same idea as a streaming scan that remembers only the two most recent words (prev2, prev), checking them against lead/mid before absorbing the current word. The underlying technique is a fixed-size sliding-window / consecutive-triple pattern match over a token sequence.

Complexity

Time: O(n) where n is the number of words in the sentence — a single left-to-right pass inspects each word (and its two predecessors) once. Space: O(n) for the tokenized words plus the output list; the streaming C++ version keeps only two previous words, so its auxiliary space beyond the output is O(1). Justification: no word is examined more than a constant number of times.

Python
PYTHON
from typing import List

def words_after_pair(sentence: str, lead: str, mid: str) -> List[str]:
    words = sentence.split()
    result = []
    for i in range(len(words) - 2):
        if words[i] == lead and words[i + 1] == mid:
            result.append(words[i + 2])
    return result
JavaScript
JAVASCRIPT
function wordsAfterPair(sentence, lead, mid) {

#### cons
t words = sentence.split(" ");

#### cons
t result = [];
    for (let i = 0; i + 2 < words.length; i++) {
        if (words[i] === lead && words[i + 1] === mid) {
            result.push(words[i + 2]);
        }
    }
    return result;
}
Java
JAVA
import java.util.*;

class Solution {
    public String[] wordsAfterPair(String sentence, String lead, String mid) {
        String[] words = sentence.split(" ");
        List<String> result = new ArrayList<>();
        for (int i = 0; i + 2 < words.length; i++) {
            if (words[i].equals(lead) && words[i + 1].equals(mid)) {
                result.add(words[i + 2]);
            }
        }
        return result.toArray(new String[0]);
    }
}
C++
CPP
#include <sstream>
#include <string>
#include <vector>
using namespace std;

class Solution {
 public:
  vector<string> wordsAfterPair(string sentence, string lead, string mid) {
    vector<string> result;
    istringstream ss(sentence);
    string prev2, prev, word;
    while (ss >> word) {
      if (prev2 == lead && prev == mid) result.push_back(word);
      prev2 = prev;
      prev = word;
    }
    return result;
  }
};
C
C
#include <stdlib.h>
#include <string.h>

/* Returns, via *returnSize, a malloc'd array of the words that immediately
   follow every occurrence of "lead mid" in `sentence`. */
char** wordsAfterPair(const char* sentence, const char* lead, const char* mid,
                      int* returnSize) {
    char* buf = strdup(sentence);            /* strtok mutates its input */
    int cap = 8, n = 0;
    char** words = malloc(sizeof(char*) * cap);
    for (char* tok = strtok(buf, " "); tok; tok = strtok(NULL, " ")) {
        if (n == cap) { cap *= 2; words = realloc(words, sizeof(char*) * cap); }
        words[n++] = tok;
    }
    char** result = malloc(sizeof(char*) * (n > 0 ? n : 1));
    int m = 0;
    for (int i = 0; i + 2 < n; ++i) {
        if (strcmp(words[i], lead) == 0 && strcmp(words[i + 1], mid) == 0)
            result[m++] = strdup(words[i + 2]);  /* copy out before buf is freed */
    }
    free(words);
    free(buf);
    *returnSize = m;
    return result;
}

#### Follow-up
questions
Generalize to an arbitrary k-word prefix: return the word that follows every occurrence of a given sequence of k words.
Instead of the successor words, return the starting indices of the matches, or count overlapping versus non-overlapping occurrences.
What changes if words may be separated by arbitrary whitespace or punctuation rather than exactly one space?
Process the text as a stream, word by word, using only O(1) extra memory instead of first splitting the entire string.

