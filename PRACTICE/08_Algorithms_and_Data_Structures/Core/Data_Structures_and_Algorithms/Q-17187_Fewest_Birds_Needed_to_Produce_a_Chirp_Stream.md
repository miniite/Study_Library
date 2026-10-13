---
type: "practice"
difficulty: "Other"
---

# Fewest Birds Needed to Produce a Chirp Stream

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ String](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/String.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A group of birds is chirping. Each individual bird produces the word chirp by emitting its five distinct letters — c, h, i, r, p — one after another in that exact order. A bird must emit all five letters to complete one chirp, and only after finishing a chirp may it immediately begin another one.

Because several birds can chirp at the same time, the letters they produce arrive interleaved in a single stream. You are given that stream as a string stream, where every character is one of c, h, i, r, p.

Return the minimum number of distinct birds required to produce stream so that the whole string can be decomposed into complete, correctly ordered chirp sequences. If stream cannot be formed as a valid interleaving of complete chirps, return -1.

Any number of birds may be chirping concurrently, but a single bird can only work on one chirp at a time.

Input

stream: a non-empty string; each character is one of 'c', 'h', 'i', 'r', 'p'.

Output

An integer: the minimum number of birds needed, or -1 if the stream is not a valid combination of complete chirps.

#### Cons
traints

1 <= stream.length <= 10^5
Every character of stream is one of 'c', 'h', 'i', 'r', 'p'.

#### Example
s & test cases
TEXT
- Input: `stream = "chirpchirp"` -> Output: `1`
  Why: one bird can chirp twice back-to-back; no overlap is needed, so a single bird suffices.
- Input: `stream = "chcirphirp"` -> Output: `2`
  Why: a second `c` appears before the first chirp finishes, so two birds must overlap; each still emits a complete `chirp` in order.
- Input: `stream = "cchhiirrpp"` -> Output: `2`
  Why: two birds chirp perfectly in sync — at the peak, two birds are simultaneously active, so two are required.
- Input: `stream = "chirp"` -> Output: `1`
  Why: a single complete chirp needs exactly one bird (minimum-length valid case).
- Input: `stream = "chirpchiip"` -> Output: `-1`
  Why: the second chirp emits `i` twice with no matching `h` waiting, so the stream cannot decompose into valid chirps.
- Input: `stream = "chir"` -> Output: `-1`
  Why: the chirp is left unfinished (no `p`), so a bird is stuck mid-chirp and the stream is invalid.

#### Approach

Model the word chirp as a five-stage pipeline and simulate the stream once, tracking how many birds are waiting just after each stage. Reading 'c' opens a new chirp, so a bird becomes active; the running maximum of active birds is the answer, because that peak is the fewest birds that must coexist. Reading any later letter must "hand off" a bird from the immediately preceding stage: decrement the previous stage's waiting count, and if it goes negative there was no bird ready to advance, so the stream is malformed and we return -1. Reading the final letter 'p' completes a chirp and frees a bird (decrement active). After the scan, every bird must have finished its chirp — the active count must be back to 0; otherwise some bird is stuck mid-word and the answer is -1. This is a counting / greedy-simulation pattern: the peak concurrency equals the minimum resources needed.

Complexity

Time: O(n) where n = stream.length — a single left-to-right pass, with O(1) work per character (index lookup in the fixed 5-letter word). Space: O(1) — a fixed 5-slot counter plus two integers, independent of input size. Justification: each character advances or opens exactly one bird's pipeline stage, so no character is revisited.

Python
PYTHON
class Solution:
    def fewestChirpingBirds(self, stream: str) -> int:
        word = "chirp"
        counter = [0] * 5          # birds waiting after each stage
        active = peak = 0
        for ch in stream:
            idx = word.find(ch)
            if idx < 0:
                return -1
            counter[idx] += 1
            if idx == 0:            # 'c' opens a new chirp
                active += 1
                peak = max(peak, active)
            else:
                counter[idx - 1] -= 1        # consume the previous stage
                if counter[idx - 1] < 0:
                    return -1
                if idx == 4:                 # 'p' completes a chirp
                    active -= 1
        return peak if active == 0 else -1   # no bird left mid-chirp
JavaScript
JAVASCRIPT
function fewestChirpingBirds(stream) {

#### cons
t word = "chirp";

#### cons
t counter = new Array(5).fill(0);   // birds waiting after each stage
    let active = 0, peak = 0;
    for (const ch of stream) {

#### cons
t idx = word.indexOf(ch);
        if (idx < 0) return -1;
        counter[idx]++;
        if (idx === 0) {                     // 'c' opens a new chirp
            peak = Math.max(peak, ++active);
        } else {
            counter[idx - 1]--;              // consume the previous stage
            if (counter[idx - 1] < 0) return -1;
            if (idx === 4) active--;         // 'p' completes a chirp
        }
    }
    return active === 0 ? peak : -1;         // no bird left mid-chirp
}
Java
JAVA
class Solution {
  public int fewestChirpingBirds(String stream) {
    String word = "chirp";
    int[] counter = new int[5];             // birds waiting after each stage
    int active = 0, peak = 0;
    for (int i = 0; i < stream.length(); i++) {
      int idx = word.indexOf(stream.charAt(i));
      if (idx < 0) return -1;
      counter[idx]++;
      if (idx == 0) {                        // 'c' opens a new chirp
        peak = Math.max(peak, ++active);
      } else {
        counter[idx - 1]--;                  // consume the previous stage
        if (counter[idx - 1] < 0) return -1;
        if (idx == 4) active--;              // 'p' completes a chirp
      }
    }
    return active == 0 ? peak : -1;          // no bird left mid-chirp
  }
}
C++
CPP
class Solution {
 public:
  int fewestChirpingBirds(string stream) {

#### cons
t string word = "chirp";
    int counter[5] = {0, 0, 0, 0, 0};      // birds waiting after each stage
    int active = 0, peak = 0;
    for (char ch : stream) {
      size_t idx = word.find(ch);
      if (idx == string::npos) return -1;
      counter[idx]++;
      if (idx == 0) {                       // 'c' opens a new chirp
        peak = max(peak, ++active);
      } else {
        counter[idx - 1]--;                 // consume the previous stage
        if (counter[idx - 1] < 0) return -1;
        if (idx == 4) active--;             // 'p' completes a chirp
      }
    }
    return active == 0 ? peak : -1;         // no bird left mid-chirp
  }
};
C
C
#include <string.h>

int fewestChirpingBirds(char* stream) {

#### cons
t char* word = "chirp";
    int counter[5] = {0, 0, 0, 0, 0};   /* birds waiting after emitting each stage */
    int active = 0, peak = 0;
    for (int i = 0; stream[i] != '\0'; ++i) {

#### cons
t char* pos = strchr(word, stream[i]);
        if (pos == NULL) return -1;
        int idx = (int)(pos - word);
        counter[idx]++;
        if (idx == 0) {                 /* 'c' opens a new chirp -> a bird becomes active */
            if (++active > peak) peak = active;
        } else {
            counter[idx - 1]--;         /* consume a bird waiting at the previous stage */
            if (counter[idx - 1] < 0) return -1;
            if (idx == 4) active--;     /* 'p' finishes a chirp -> a bird is freed */
        }
    }
    return active == 0 ? peak : -1;     /* no bird may be left mid-chirp */
}

#### Follow-up
questions
Generalize the target word to any string of distinct characters supplied at runtime, not just chirp — how does the counter array change?
What if a bird may pause mid-chirp and resume later — does the minimum-birds answer change, and how would you detect invalid streams then?
Return not just the count of birds but an assignment of each character to a specific bird id.
Stream the input character-by-character with O(1) memory and report -1 at the earliest possible position — where exactly can you detect invalidity?

