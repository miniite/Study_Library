---
type: "practice"
difficulty: "Intermediate"
---

# Given a matrix, rotate it 90 degrees in place and explain the index mapping.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Matrix Search](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Matrix_Search.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked: Rotate an n x n matrix by 90 degrees clockwise (the default; I'll note anti-clockwise too), in place — meaning O(1) extra space, no separate output matrix. And critically, explain the index mapping: exactly which source cell lands in which destination cell.

What the interviewer is probing for:

Do you know the two-step trick (transpose + reverse), and can you also do the direct 4-way cyclic swap?
Can you derive the index formula from first principles, not just memorize it?
Do you handle the in-place constraint correctly (no accidental O(n²) allocation)?
Do you get the boundary/loop ranges right so you don't double-rotate cells?

Clarifying questions I'd ask:

Is the matrix guaranteed square (n x n)? True in-place 90° rotation is only possible for square matrices — a rectangular m x n becomes n x m, so it cannot be done in place with the same buffer. I'll assume square.
Clockwise or counter-clockwise? I'll default to clockwise.
Any memory constraint beyond in-place? I'll aim for O(1) auxiliary space.

Plan: Derive the index mapping. Then present two correct approaches — (A) transpose-then-reverse-rows, and (B) the direct 4-cell cyclic swap. Dry-run a 3x3, state complexity, and note the common bugs.

Deriving the index mapping (the heart of the question)

Take an n x n matrix. A cell at row i, column j — write it (i, j) — moves to a new position after a 90° clockwise rotation.

Think physically: the top row of the original becomes the rightmost column of the result. So element (0, j) (top row) goes to column n-1 and row j. Generalizing, the row index i becomes the distance from the right edge, i.e. new column = n - 1 - i; and the column index j becomes the new row.

Clockwise mapping:

CODE
(i, j)  ->  (j, n-1-i)

Equivalently, the value that ends up at destination (i, j) came from source (n-1-j, i).

Counter-clockwise (for contrast):

CODE
(i, j)  ->  (n-1-j, i)

Sanity check on a 3x3 (n = 3), clockwise: (0,0) -> (0,2), (0,2) -> (2,2), (2,2) -> (2,0), (2,0) -> (0,0). Those four form one cycle — a group of 4 cells that just rotate among themselves. That is exactly why the direct method works with a single temp variable: each swap group is a 4-cycle.

#### Approach
A — Transpose, then reverse each row

Two clean, easy-to-verify passes:

Transpose (mirror across the main diagonal): swap M[i][j] with M[j][i]. After transpose, (i, j) holds what was at (j, i).
Reverse each row: (i, j) swaps with (i, n-1-j).

Why this equals a clockwise rotation — compose the two: transpose sends (i,j) -> (j,i); reversing rows sends (i,k) -> (i, n-1-k). Start with the original value at (a,b). Transpose moves it to (b,a). Row-reverse moves it to (b, n-1-a). That is precisely the clockwise formula (a,b) -> (b, n-1-a). QED.

For counter-clockwise, do transpose then reverse each column (or reverse rows first, then transpose).

The transpose loop must use j starting at i+1 (upper triangle only). If you loop j over all columns you swap every pair twice and end up transposing back to the original — a classic bug.

Complexity: O(n²) time, O(1) extra space. Simple, readable, interview-safe.

#### Approach
B — Direct 4-way cyclic swap (single pass, one temp)

Rotate in concentric "rings/layers" from outside in. For each ring, rotate 4 elements at a time using the cycle we found. With n rows, there are n/2 layers; within a layer indexed by layer, the elements to touch run from first = layer to last = n-1-layer. For each offset within the ring:

CODE
top    = M[first][first + offset]
M[first][first + offset]        = M[last - offset][first]        // left  -> top
M[last - offset][first]         = M[last][last - offset]         // bottom-> left
M[last][last - offset]          = M[first + offset][last]        // right -> bottom
M[first + offset][last]         = top                            // top   -> right

This moves each cell exactly once, uses a single temporary, and never allocates a second matrix. Same O(n²) / O(1). It's the most "in-place" answer and shows you truly understand the cycle. Approach A is what I'd write for clarity; Approach B is what I'd offer if asked to avoid the two-pass overhead.

Dry run (3x3, clockwise), Approach A

Original:

CODE
1 2 3
4 5 6
7 8 9

After transpose (swap across diagonal):

CODE
1 4 7
2 5 8
3 6 9

After reversing each row:

CODE
7 4 1
8 5 2
9 6 3

That is the correct 90° clockwise result (top row 1 2 3 became right column 1 2 3 reading top-to-bottom). ✔

Summary — what a strong answer emphasizes
State the mapping crisply: clockwise (i,j) -> (j, n-1-i), counter-clockwise (i,j) -> (n-1-j, i).
Give the transpose + reverse-rows method and prove it composes to that mapping — don't just recite it.
Mention the 4-way cyclic swap as the true single-pass in-place method.
Nail O(n²) time, O(1) space, and note that only square matrices rotate in place.

#### Common mistakes
/ follow-ups:

Transpose loop starting j = 0 (double-swaps → no-op). Must be j = i+1.
Confusing clockwise vs counter-clockwise (reverse rows vs reverse columns).
Claiming rectangular matrices can rotate in place — they can't (dimensions change).

#### Follow-up
probes: rotate 180° (reverse both dimensions) or 270° (transpose + reverse columns); rotate a rectangular matrix (needs a new buffer).

**Comparison:**

```
Method	Extra space	Passes over data	Best for
Transpose + reverse rows	O(1)	2	Readability, interview default
4-way cyclic swap	O(1)	1	True single-pass in-place
Copy into new matrix using (i,j)->(j,n-1-i)	O(n²)	1	When in-place not required / rectangular
Rotation	Index mapping (source (i,j) goes to)	Recipe
90° clockwise	(j, n-1-i)	transpose, then reverse each row
90° counter-clockwise	(n-1-j, i)	transpose, then reverse each column
180°	(n-1-i, n-1-j)	reverse rows and reverse columns
```

**Code:**

```python
def rotate_clockwise(M):
    """Rotate an n x n matrix 90 degrees clockwise, in place. O(n^2) time, O(1) space."""
    n = len(M)
    # 1) Transpose: swap upper triangle with lower triangle
    for i in range(n):
        for j in range(i + 1, n):          # j starts at i+1, not 0
            M[i][j], M[j][i] = M[j][i], M[i][j]
    # 2) Reverse each row
    for i in range(n):
        M[i].reverse()
    return M

def rotate_clockwise_cyclic(M):
    """Direct 4-way ring rotation, one temp var. O(n^2) time, O(1) space."""
    n = len(M)
    for layer in range(n // 2):
        first, last = layer, n - 1 - layer
        for off in range(first, last):
            i = off - first
            top = M[first][off]
            M[first][off]        = M[last - i][first]    # left  -> top
            M[last - i][first]   = M[last][last - i]      # bottom-> left
            M[last][last - i]    = M[first + i][last]     # right -> bottom
            M[first + i][last]   = top                    # top   -> right
    return M
```

**Example input / output:**

```

Input (n = 3):

CODE
Copy
1 2 3
4 5 6
7 8 9

rotate_clockwise -> Output:

CODE
Copy
7 4 1
8 5 2
9 6 3

Trace of rotate_clockwise_cyclic on the same input (single layer, first=0 last=2):

off=0: top=M[0][0]=1; M[0][0]=M[2][0]=7; M[2][0]=M[2][2]=9; M[2][2]=M[0][2]=3; M[0][2]=1
off=1: top=M[0][1]=2; M[0][1]=M[1][0]=4; M[1][0]=M[2][1]=8; M[2][1]=M[1][2]=6; M[1][2]=2
center M[1][1]=5 untouched Result identical to above. ✔
```

**Flow:**

1. Original n x n matrix
2. Transpose: swap M[i][j] &M[j][i] for j>i
3. Reverse each row
4. 90 deg clockwise result

