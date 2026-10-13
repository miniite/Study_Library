---
type: "practice"
difficulty: "Other"
---

# Does the Rover Stay Within Some Circle Forever

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Math](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Math.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A small rover moves on an infinite flat grid. It starts at the origin (0, 0) and initially faces north. Directions are defined as:

North is the positive y direction.
South is the negative y direction.
East is the positive x direction.
West is the negative x direction.

You are given a string commands. Each character is one of three instructions:

'F': step forward exactly 1 unit in the direction the rover currently faces.
'L': rotate the rover 90 degrees to the left (counter-clockwise). Its position does not change.
'R': rotate the rover 90 degrees to the right (clockwise). Its position does not change.

The rover executes the whole commands string once in order, then repeats the entire string again, and again, forever.

Return true if and only if there exists some circle in the plane (of finite radius, centered anywhere) such that the rover's trajectory never leaves that circle; otherwise return false.

Intuition to keep in mind: after one full pass through commands, the rover is bounded forever exactly when either it has returned to the origin, or its facing direction is no longer north. If it ends a pass having moved but still facing north, each subsequent pass shifts it by the same nonzero vector, so it drifts to infinity.

#### Cons
traints:

1 <= commands.length <= 100
Each character of commands is 'F', 'L', or 'R'.

#### Example
s & test cases
TEXT
Input: commands = "FFRRFF" -> Output: true
Why: One pass: forward to (0,2), two right turns to face south, forward back to (0,0). It returns to the origin, so it loops forever within a bounded region.

Input: commands = "FF" -> Output: false
Why: Ends at (0,2) still facing north; every repeat adds (0,2), so the rover marches north without bound.

Input: commands = "FL" -> Output: true
Why: Ends at (0,1) but facing west (net 90-degree turn). A nonzero net rotation forces the path back to the origin within at most four passes, so it stays bounded.

Input: commands = "FRFRFRFR" -> Output: true
Why: The rover traces a unit square and finishes at (0,0) facing north again — a perfect closed cycle, clearly bounded.

Input: commands = "L" -> Output: true
Why: The rover only turns and never moves, so it stays fixed at the origin forever (trivially bounded).

#### Approach

Simulate a single execution of the command string, tracking position (x, y) and a facing index d over the four unit direction vectors (N, E, S, W). A left turn is d = (d + 3) % 4 and a right turn is d = (d + 1) % 4. After one full pass, the motion of every future pass is described by the same rigid transformation (a translation composed with the net rotation). The rover is bounded forever exactly when the net displacement after one pass is zero, OR the net rotation is nonzero (facing not north): a nonzero rotation guarantees that after at most four passes the accumulated displacement cancels and the rover returns to the origin, producing a closed loop. Only a pure nonzero translation with no rotation escapes to infinity. This is the "detect the cycle from one period of a repeating rigid motion" simulation pattern.

Complexity

Time: O(n) where n = commands.length — a single pass over the string. Space: O(1) — only the position, a direction index, and fixed direction tables. Justification: boundedness is decided by simulating exactly one pass; no repetition or extra storage is required.

Python
PYTHON
class Solution:
    def staysBounded(self, commands: str) -> bool:
        x = y = 0
        d = 0  # 0=N, 1=E, 2=S, 3=W
        dirs = [(0, 1), (1, 0), (0, -1), (-1, 0)]
        for c in commands:
            if c == 'F':
                x += dirs[d][0]
                y += dirs[d][1]
            elif c == 'L':
                d = (d + 3) % 4   # counter-clockwise
            else:
                d = (d + 1) % 4   # 'R', clockwise
        return (x == 0 and y == 0) or d != 0
JavaScript
JAVASCRIPT
function staysBounded(commands) {
    let x = 0, y = 0, d = 0;          // d: 0=N, 1=E, 2=S, 3=W

#### cons
t dx = [0, 1, 0, -1];

#### cons
t dy = [1, 0, -1, 0];
    for (const c of commands) {
        if (c === 'F') { x += dx[d]; y += dy[d]; }
        else if (c === 'L') d = (d + 3) % 4;   // counter-clockwise
        else d = (d + 1) % 4;                  // 'R', clockwise
    }
    return (x === 0 && y === 0) || d !== 0;
}
Java
JAVA
class Solution {
    public boolean staysBounded(String commands) {
        int x = 0, y = 0, d = 0;          // d: 0=N, 1=E, 2=S, 3=W
        int[] dx = {0, 1, 0, -1};
        int[] dy = {1, 0, -1, 0};
        for (char c : commands.toCharArray()) {
            if (c == 'F') { x += dx[d]; y += dy[d]; }
            else if (c == 'L') d = (d + 3) % 4;   // counter-clockwise
            else d = (d + 1) % 4;                 // 'R', clockwise
        }
        return (x == 0 && y == 0) || d != 0;
    }
}
C++
CPP
#include <string>
using namespace std;

class Solution {
public:
    bool staysBounded(const string& commands) {
        int x = 0, y = 0, d = 0;          // d: 0=N, 1=E, 2=S, 3=W
        int dx[4] = {0, 1, 0, -1};
        int dy[4] = {1, 0, -1, 0};
        for (char c : commands) {
            if (c == 'F') { x += dx[d]; y += dy[d]; }
            else if (c == 'L') d = (d + 3) % 4;   // counter-clockwise
            else d = (d + 1) % 4;                 // 'R', clockwise
        }
        return (x == 0 && y == 0) || d != 0;
    }
};
C
C
#include <stdbool.h>

bool staysBounded(const char* commands) {
    int x = 0, y = 0, d = 0;          /* d: 0=N, 1=E, 2=S, 3=W */
    int dx[4] = {0, 1, 0, -1};
    int dy[4] = {1, 0, -1, 0};
    for (int i = 0; commands[i] != '\0'; ++i) {
        char c = commands[i];
        if (c == 'F') { x += dx[d]; y += dy[d]; }
        else if (c == 'L') d = (d + 3) % 4;   /* counter-clockwise */
        else d = (d + 1) % 4;                 /* 'R', clockwise */
    }
    return (x == 0 && y == 0) || d != 0;
}

#### Follow-up
questions
Prove rigorously why exactly one pass suffices, and why a nonzero net rotation always returns the rover to the origin within four passes.
If bounded, return the radius (or center) of a smallest circle that contains the whole trajectory.
What changes if 'F' can move a variable number of units, or if backward motion is allowed?
Generalize the turn to an arbitrary angle (say 60 degrees) — for which command strings is the path bounded?

