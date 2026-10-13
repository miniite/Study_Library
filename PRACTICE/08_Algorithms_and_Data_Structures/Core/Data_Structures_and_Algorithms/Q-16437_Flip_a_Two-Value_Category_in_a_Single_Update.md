---
type: "practice"
difficulty: "Other"
---

# Flip a Two-Value Category in a Single Update

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Database](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Database.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A registry stores one row per player, and each player belongs to exactly one of two groups.

Table: Roster

Column Name	Type
id	int
name	varchar
team	enum
score	int

id is the primary key (unique value) of this table. The team column is an ENUM (category) whose value is always exactly one of two strings: 'A' or 'B'. Each row records one player's id, name, team, and score.

Swap all 'A' and 'B' values in the team column — that is, change every 'A' to 'B' and every 'B' to 'A' — using a single update statement, with no intermediate/temporary table.

You must write a single update statement, and you must not write any select statement for this problem.

#### Cons
traints

team is always exactly one of 'A' or 'B' (never NULL, never any other value).
The transformation must be done with one update statement (no two-step overwrite, no temp table).
1 <= number of rows <= 10^5.
Only the team column changes; id, name, and score stay the same.

#### Example
s & test cases
TEXT
Input:
Roster:
| id | name  | team | score |
| 1  | Pat   | A    | 2500  |
| 2  | Quinn | B    | 1500  |
| 3  | Ravi  | A    | 5500  |
| 4  | Sami  | B    | 500   |
-> Output:
| id | name  | team | score |
| 1  | Pat   | B    | 2500  |
| 2  | Quinn | A    | 1500  |
| 3  | Ravi  | B    | 5500  |
| 4  | Sami  | A    | 500   |
Why: Every `A` becomes `B` and every `B` becomes `A`; all other columns are unchanged.

Input:
Roster: | 1 | Ana | A | 100 | ; | 2 | Bo | A | 200 |
-> Output: | 1 | Ana | B | 100 | ; | 2 | Bo | B | 200 |
Why: Edge case — a column that is uniformly one value flips entirely to the other; this is exactly what a naive two-statement approach would get wrong.

Input:
Roster: | 7 | Uma | B | 9000 |
-> Output: | 7 | Uma | A | 9000 |
Why: A single `B` row toggles to `A`.

Input:
Roster: (no rows)
-> Output: (no rows)
Why: An empty table has nothing to flip; the update affects zero rows.

#### Approach

The key idea is a single-pass conditional toggle of a two-value field. For a binary category, "swap A and B" is just "map each value to the other one," which can be computed independently per row — no lookup of other rows and no scratch storage is needed. Each language walks the rows once and rewrites team with a conditional expression.

In SQL this is one statement using a CASE expression:

SQL
UPDATE Roster
SET team = CASE team WHEN 'A' THEN 'B' ELSE 'A' END;

The single UPDATE is essential. The tempting but wrong approach — UPDATE ... SET team='B' WHERE team='A'; followed by UPDATE ... SET team='A' WHERE team='B'; — fails: after the first statement, the rows that were originally 'B' are indistinguishable from the ones just changed to 'B', so the second statement flips everything back to 'A'. Because the CASE is evaluated against each row's original value within one atomic statement, both directions of the swap happen simultaneously and correctly. (Databases that support IF() allow the equivalent SET team = IF(team = 'A', 'B', 'A').)

Complexity

Time: O(n) — one pass over the rows, doing O(1) work per row. Space: O(1) extra — every update is in place, so nothing grows with the number of rows. The reference SQL is a single UPDATE that scans the table once and rewrites the team value per row.

Python
PYTHON
from typing import List, Dict

def swap_teams(rows: List[Dict]) -> List[Dict]:
    # Flip each row's team between "A" and "B" in one pass, in place.
    for r in rows:
        r["team"] = "B" if r["team"] == "A" else "A"
    return rows
JavaScript
JAVASCRIPT
// Toggles each row's team between "A" and "B" in a single pass, in place.
function swapTeams(rows) {
    for (const r of rows) {
        r.team = r.team === "A" ? "B" : "A";
    }
    return rows;
}
Java
JAVA
import java.util.*;

class Solution {
    static class Row {
        int id; String name; String team; int score;
        Row(int id, String name, String team, int score) {
            this.id = id; this.name = name; this.team = team; this.score = score;
        }
    }

    // Toggles each row's team between "A" and "B" in a single pass.
    public List<Row> swapTeams(List<Row> rows) {
        for (Row r : rows)
            r.team = r.team.equals("A") ? "B" : "A";
        return rows;
    }
}
C++
CPP
#include <vector>
#include <string>
using namespace std;

struct Row { int id; string name; string team; int score; };

// Flips each row's team ('A' <-> 'B') in one pass, in place.
void swapTeams(vector<Row>& rows) {
    for (auto& r : rows)
        r.team = (r.team == "A") ? "B" : "A";
}
C
C
#include <stddef.h>

typedef struct {
    int  id;
    char name[64];
    char team[2];   /* "A" or "B" (single char + terminator) */
    int  score;
} Row;

/* Toggles every row's team between 'A' and 'B' in a single pass, in place. */
void swapTeams(Row* rows, size_t n) {
    for (size_t i = 0; i < n; i++)
        rows[i].team[0] = (rows[i].team[0] == 'A') ? 'B' : 'A';
}

#### Follow-up
questions
Why does the two-statement approach (flip A->B, then flip B->A) fail, and how does the single CASE update avoid it?
Do the swap without a CASE expression — e.g. using a character/ASCII arithmetic trick or a lookup that maps each value to its opposite.
Generalize to rotating a three-value enum (A->B->C->A) in a single update.
If the column could contain NULLs or a stray third value, how would you make the update leave those rows untouched?

