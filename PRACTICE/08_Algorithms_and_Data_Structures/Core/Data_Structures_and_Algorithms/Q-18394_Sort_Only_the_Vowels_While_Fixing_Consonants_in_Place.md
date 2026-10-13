---
type: "practice"
difficulty: "Other"
---

# Sort Only the Vowels While Fixing Consonants in Place

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Sorting](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Sorting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a 0-indexed string text made up of English letters (both uppercase and lowercase). Build a new string result by rearranging only the vowels of text according to these rules:

Every consonant stays exactly where it is. Formally, if text[i] is a consonant, then result[i] = text[i].
The vowels are reordered so that, read left to right across the positions they occupy, they appear in non-decreasing order of their ASCII values. Formally, for any two vowel positions i < j, result[i] must not have a higher ASCII value than result[j].

The vowels are 'a', 'e', 'i', 'o', 'u' and their uppercase forms 'A', 'E', 'I', 'O', 'U'. Every other letter is a consonant. Because uppercase letters have smaller ASCII values than lowercase ones, in the sorted order all uppercase vowels come before all lowercase vowels ('A' < 'E' < 'I' < 'O' < 'U' < 'a' < 'e' < 'i' < 'o' < 'u').

Return the resulting string.

Input

text: a string of English letters, uppercase and/or lowercase.

Output

The string with consonants unchanged and vowels sorted by ASCII value into their original vowel slots.

#### Cons
traints

1 <= text.length <= 10^5
text consists only of uppercase and lowercase English letters.

#### Example
s & test cases
TEXT
Input: text = "cOdInG" -> Output: "cIdOnG"
Why: The vowels are 'O' (position 1) and 'I' (position 3). Sorted by ASCII, 'I' (73) comes before 'O' (79), so they swap into their slots while c, d, n, G stay put.

Input: text = "rhythm" -> Output: "rhythm"
Why: There are no vowels at all, so nothing changes.

Input: text = "programming" -> Output: "pragrimmong"
Why: The vowels 'o', 'a', 'i' (positions 2, 5, 8) sort to 'a', 'i', 'o' and drop back into those same three positions; all consonants keep their places.

Input: text = "aeiou" -> Output: "aeiou"
Why: The vowels are already in non-decreasing ASCII order, so the string is unchanged (already-sorted case).

Input: text = "EIOUA" -> Output: "AEIOU"
Why: All five uppercase vowels get reordered by ASCII into A, E, I, O, U.

Input: text = "aA" -> Output: "Aa"
Why: 'A' (65) has a smaller ASCII value than 'a' (97), so the uppercase vowel must come first — confirms uppercase-before-lowercase ordering.

#### Approach

Only the characters sitting in vowel positions ever move, and they must end up sorted by ASCII value. Because there are just 10 distinct vowel characters, use counting sort: scan once and tally how many of each vowel occur. Then scan again; whenever the current position holds a vowel, overwrite it with the smallest remaining vowel (advancing a bucket pointer through the fixed order A E I O U a e i o u) and decrement that bucket's count. Consonants are copied through untouched. This is the "fixed-alphabet counting sort applied to selected positions" pattern — it beats a general sort because the key domain is tiny and constant, giving linear time.

Complexity

Time: O(n) — one pass to tally the vowels and one pass to rewrite them, since the vowel alphabet has a fixed size of 10 (counting sort, not comparison sort). Space: O(n) for the output buffer plus O(1) extra for the fixed 10-element count array. A comparison-sort variant (extract vowels, sort, reinsert) would be O(n log n).

Python
PYTHON
def sort_vowels(text: str) -> str:
    vowels = "AEIOUaeiou"                # already in ascending ASCII order
    idx = {c: i for i, c in enumerate(vowels)}

    count = [0] * 10
    for ch in text:                      # counting sort over the 10 vowels
        if ch in idx:
            count[idx[ch]] += 1

    res = list(text)
    cur = 0                              # next vowel bucket to emit
    for i, ch in enumerate(res):
        if ch in idx:
            while count[cur] == 0:
                cur += 1
            res[i] = vowels[cur]
            count[cur] -= 1
    return "".join(res)
JavaScript
JAVASCRIPT
function sortVowels(text) {

#### cons
t vowels = "AEIOUaeiou";          // ascending ASCII order

#### cons
t idx = {};
    for (let i = 0; i < vowels.length; i++) idx[vowels[i]] = i;

#### cons
t count = new Array(10).fill(0);
    for (const ch of text) {              // counting sort over the 10 vowels
        if (ch in idx) count[idx[ch]]++;
    }

#### cons
t res = text.split('');
    let cur = 0;                          // next vowel bucket to emit
    for (let i = 0; i < res.length; i++) {
        if (res[i] in idx) {
            while (count[cur] === 0) cur++;
            res[i] = vowels[cur];
            count[cur]--;
        }
    }
    return res.join('');
}
Java
JAVA
class Solution {
    private int vowelIndex(char c) {
        return "AEIOUaeiou".indexOf(c);   // -1 if not a vowel
    }

    public String sortVowels(String text) {
        final String vowels = "AEIOUaeiou";
        int[] count = new int[10];
        for (int i = 0; i < text.length(); i++) {   // counting sort over the 10 vowels
            int vi = vowelIndex(text.charAt(i));
            if (vi >= 0) count[vi]++;
        }

        char[] res = text.toCharArray();
        int cur = 0;                                 // next vowel bucket to emit
        for (int i = 0; i < res.length; i++) {
            if (vowelIndex(res[i]) >= 0) {
                while (count[cur] == 0) cur++;
                res[i] = vowels.charAt(cur);
                count[cur]--;
            }
        }
        return new String(res);
    }
}
C++
CPP
#include <string>
using namespace std;

class Solution {
    static int vowelIndex(char c) {
        static const string v = "AEIOUaeiou";
        size_t p = v.find(c);
        return p == string::npos ? -1 : (int)p;
    }
public:
    string sortVowels(string text) {

#### cons
t string vowels = "AEIOUaeiou";
        int count[10] = {0};
        for (char c : text) {                 // counting sort over the 10 vowels
            int vi = vowelIndex(c);
            if (vi >= 0) count[vi]++;
        }
        int cur = 0;                          // next vowel bucket to emit
        for (char& c : text) {
            if (vowelIndex(c) >= 0) {
                while (count[cur] == 0) cur++;
                c = vowels[cur];
                count[cur]--;
            }
        }
        return text;
    }
};
C
C
#include <string.h>
#include <stdlib.h>

static int vowelIndex(char c) {
    switch (c) {
        case 'A': return 0; case 'E': return 1; case 'I': return 2;
        case 'O': return 3; case 'U': return 4;
        case 'a': return 5; case 'e': return 6; case 'i': return 7;
        case 'o': return 8; case 'u': return 9;
        default:  return -1;
    }
}

char* sortVowels(char* text) {
    static const char vowels[10] = {'A','E','I','O','U','a','e','i','o','u'};
    int count[10] = {0};
    int n = (int)strlen(text);

    for (int i = 0; i < n; i++) {           /* counting sort over the 10 vowels */
        int vi = vowelIndex(text[i]);
        if (vi >= 0) count[vi]++;
    }

    char* result = (char*)malloc(n + 1);
    int cur = 0;                            /* next vowel bucket to emit */
    for (int i = 0; i < n; i++) {
        if (vowelIndex(text[i]) >= 0) {
            while (count[cur] == 0) cur++;
            result[i] = vowels[cur];
            count[cur]--;
        } else {
            result[i] = text[i];
        }
    }
    result[n] = '\0';
    return result;
}

#### Follow-up
questions
What if you also had to sort the consonants among themselves while keeping the vowels fixed in their positions?
How would you adapt this to a Unicode alphabet with accented vowels, where a fixed 10-bucket counting scheme no longer covers every case?
Can you do the rewrite fully in place (aside from the small counting array) rather than allocating a separate result buffer?
What if the letter 'y' should sometimes count as a vowel depending on its position in a word?

