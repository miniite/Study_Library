---
type: "practice"
difficulty: "Other"
---

# Read N Characters Through a Fixed Four-Character Reader

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Array](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Array.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are working with an opaque, sequential character source. You may not touch the source directly; the only way to pull data out of it is a single low-level primitive:

readBlock(buf4) — copies up to four consecutive characters from the source into the caller-provided array buf4 (which has length 4) and returns how many characters it actually copied. The primitive keeps its own internal cursor into the source, so each call resumes exactly where the previous one stopped. When fewer than four characters remain it copies only those and returns that smaller count; once the source is exhausted it returns 0.

Using only readBlock, implement a function readChars(dest, n) that reads at most n characters from the source and stores them, in original order, into the destination array dest (which is guaranteed to have room for n characters). The function must return the number of characters actually written into dest. That number equals n unless the source runs out of characters first, in which case it equals the total number of characters the source held.

For this variant, readChars is called exactly once — you do not have to remember anything between calls.

Input / Output

Input: the underlying source content (available only through readBlock), a destination buffer dest, and an integer n.
Output: the count of characters written to dest; dest[0..count-1] holds those characters.

#### Cons
traints

1 <= source length <= 500
The source consists of English letters and digits only.
1 <= n <= 1000
Each readBlock call copies at most 4 characters and returns a value in 0..4.

#### Example
s & test cases
TEXT
- Input: source = "wolf", n = 6 -> Output: 4
  Why: the source has only 4 characters, so we hit end-of-source before reaching `n=6`.
- Input: source = "kite9", n = 5 -> Output: 5
  Why: exactly `n` characters are available; one full block (4) plus a partial block (1).
- Input: source = "map3Kd7Qz18b", n = 12 -> Output: 12
  Why: length is a multiple of 4; three full `readBlock` calls fill `dest` completely.
- Input: source = "yo", n = 1 -> Output: 1
  Why: caller wants fewer characters than the block yields; we stop after copying 1.
- Input: source = "a", n = 1 -> Output: 1
  Why: minimal case — a single character read in one partial block.

#### Approach

This is a buffered-reading / producer-consumer pattern. The primitive hands over data in fixed blocks of up to four characters, but the caller wants an arbitrary count n. Keep a small 4-slot staging buffer together with two markers: cnt4 (how many valid characters the last refill produced) and idx4 (how many of them have already been consumed). Copy characters one at a time from the staging buffer into dest; whenever the staging buffer is fully consumed (idx4 == cnt4), refill it with one readBlock call. If a refill yields 0 characters the source is exhausted, so return the number written so far. The loop naturally stops either at n characters or at end-of-source, and no character is ever read twice.

Complexity

Time: O(n) — every character requested is copied exactly once, and readBlock is called roughly n/4 times, each O(1). Space: O(1) — only a fixed 4-slot staging buffer plus three integer cursors, independent of n.

Python
PYTHON
from typing import List

# Provided by the runtime:
#   def readBlock(buf4: List[str]) -> int:  # copies up to 4 chars into buf4, returns count (0..4)

class Reader:
    def read_chars(self, dest: List[str], n: int) -> int:
        buf4 = [''] * 4
        idx4 = 0   # next unread slot inside buf4
        cnt4 = 0   # number of valid chars in buf4
        i = 0      # next write position in dest

        while i < n:
            if idx4 == cnt4:               # staging buffer drained -> refill
                idx4 = 0
                cnt4 = readBlock(buf4)
                if cnt4 == 0:              # end of source
                    return i
            dest[i] = buf4[idx4]
            i += 1
            idx4 += 1
        return i
JavaScript
JAVASCRIPT
// Provided by the runtime:
//   readBlock(buf4)  // copies up to 4 chars into buf4, returns count (0..4)

function readChars(dest, n) {

#### cons
t buf4 = new Array(4);
    let idx4 = 0;   // next unread slot inside buf4
    let cnt4 = 0;   // number of valid chars in buf4
    let i = 0;      // next write position in dest

    while (i < n) {
        if (idx4 === cnt4) {              // staging buffer drained -> refill
            idx4 = 0;
            cnt4 = readBlock(buf4);
            if (cnt4 === 0)               // end of source
                return i;
        }
        dest[i++] = buf4[idx4++];
    }
    return i;
}
Java
JAVA
// Provided by the runtime: int readBlock(char[] buf4)
// copies up to 4 chars into buf4 and returns how many (0..4).
class Reader {
    public int readChars(char[] dest, int n) {
        char[] buf4 = new char[4];
        int idx4 = 0;   // next unread slot inside buf4
        int cnt4 = 0;   // number of valid chars in buf4
        int i = 0;      // next write position in dest

        while (i < n) {
            if (idx4 == cnt4) {            // staging buffer drained -> refill
                idx4 = 0;
                cnt4 = readBlock(buf4);
                if (cnt4 == 0)             // end of source
                    return i;
            }
            dest[i++] = buf4[idx4++];
        }
        return i;
    }

    // Placeholder for the environment-supplied primitive.
    private int readBlock(char[] buf4) { return 0; }
}
C++
CPP
// Provided by the runtime:
//   int readBlock(char* buf4);  // copies up to 4 chars into buf4, returns count (0..4)
int readBlock(char* buf4);

class Reader {
public:
    int readChars(char* dest, int n) {
        char buf4[4];
        int idx4 = 0;   // next unread slot inside buf4
        int cnt4 = 0;   // number of valid chars in buf4
        int i = 0;      // next write position in dest

        while (i < n) {
            if (idx4 == cnt4) {           // staging buffer drained -> refill
                idx4 = 0;
                cnt4 = readBlock(buf4);
                if (cnt4 == 0)            // end of source
                    return i;
            }
            dest[i++] = buf4[idx4++];
        }
        return i;
    }
};
C
C
/* Provided by the runtime:
 *   int readBlock(char *buf4);   // copies up to 4 chars into buf4, returns count (0..4)
 */
int readBlock(char *buf4);

int readChars(char *dest, int n) {
    char buf4[4];
    int idx4 = 0;   /* next unread slot inside buf4 */
    int cnt4 = 0;   /* number of valid chars in buf4 */
    int i = 0;      /* next write position in dest   */

    while (i < n) {
        if (idx4 == cnt4) {            /* staging buffer drained -> refill */
            idx4 = 0;
            cnt4 = readBlock(buf4);
            if (cnt4 == 0)             /* end of source */
                return i;
        }
        dest[i++] = buf4[idx4++];
    }
    return i;
}

#### Follow-up
questions
What must change if readChars can be called multiple times on the same source? (You must
persist the unconsumed staging-buffer characters between calls — the harder variant.)
Generalize the primitive to read up to k characters per call instead of 4 — does your logic
still hold with only the constant changed?
How would you make the reader safe if two threads call readChars concurrently?
If the source could contain embedded end-of-line markers, how would you read exactly one line?

