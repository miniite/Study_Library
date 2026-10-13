---
type: "practice"
difficulty: "Other"
---

# Members Not Invited by a Specific Member

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Database](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Database.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You maintain a directory of members of a community. Each member is described by three fields:

id: a unique positive integer identifying the member.
name: the member's name.
inviter: the id of the member who invited (referred) this member. A member who joined on their own has no inviter, represented by the sentinel value 0.

Given the directory and a target id t, return the names of every member who was NOT invited by the member whose id equals t.

Concretely, a member's name should be included in the result when its inviter is not equal to t. Members with no inviter (inviter == 0) are always included, because they were not invited by t (or by anyone). Members whose inviter equals t are excluded.

Return the names in any order.

Input
A list of members, each (id, name, inviter) with inviter == 0 meaning "no inviter".
An integer t, the target inviter id to exclude.
Output
A list of names (strings), in any order, of the members not invited by member t.

#### Cons
traints
0 <= number of members <= 10^5.
Each id is a unique integer with 1 <= id <= 10^9.
inviter is either 0 (no inviter) or the id of some member.
1 <= t <= 10^9.
name is a non-empty string of length up to 100.

#### Example
s & test cases
TEXT
Input: members = [(1,"Noah",0),(2,"Mia",0),(3,"Leo",2),(4,"Ada",0),(5,"Ravi",1),(6,"Omar",2)], t = 2 -> Output: ["Noah","Mia","Ada","Ravi"]
Why: Leo and Omar have inviter == 2, so they are excluded; everyone else — including the no-inviter members Noah, Mia, Ada and the member Ravi invited by 1 — is kept. (Typical mixed case.)

Input: members = [], t = 2 -> Output: []
Why: The directory is empty, so nothing can be returned. (Empty-input edge case.)

Input: members = [(10,"Kai",2),(11,"Zoe",2)], t = 2 -> Output: []
Why: Every member was invited by the target member 2, so all are excluded. (All-excluded case.)

Input: members = [(1,"Sam",0)], t = 2 -> Output: ["Sam"]
Why: A single self-joined member (inviter 0) was not invited by 2, so it is kept. (Single-member / no-inviter case.)

Input: members = [(1,"Uma",0),(2,"Ivy",1)], t = 2 -> Output: ["Uma","Ivy"]
Why: No member has inviter == 2 here, so everyone qualifies. (No-match-to-target case.)

#### Approach

This is a straightforward filtering / selection problem. Scan the directory once and keep every member whose inviter field is different from the target id t. The only subtlety is the "no inviter" case: a member who joined independently was not invited by t, so it must be retained — modelling "no inviter" as the sentinel 0 (or SQL NULL) makes the single predicate inviter != t handle it correctly, since 0 (or NULL under the IS NULL OR <> form) is never equal to a valid target id. No sorting, grouping, or auxiliary data structure is required; it is a pure linear predicate filter, the code analogue of a SQL WHERE clause with a null-safe inequality.

Complexity

Time: O(n), where n is the number of members — a single pass tests each member's inviter once. Space: O(k) for the k qualifying names in the result (O(1) auxiliary beyond the output). Equivalent SQL: SELECT name FROM Member WHERE inviter IS NULL OR inviter <> t;.

Python
PYTHON
from typing import List, Tuple

def not_invited_by(members: List[Tuple[int, str, int]], target: int) -> List[str]:
    """Return names of members whose inviter is not `target`.

    Each member is (id, name, inviter); inviter == 0 means "no inviter",
    which is never equal to a valid target, so those members are kept.
    """
    return [name for (_id, name, inviter) in members if inviter != target]
JavaScript
JAVASCRIPT
/**
 * Return names of members whose inviter is not `target`.
 * @param {{id:number,name:string,inviter:number}[]} members  inviter === 0 means "no inviter"
 * @param {number} target
 * @return {string[]}
 */
function notInvitedBy(members, target) {
    return members
        .filter(m => m.inviter !== target)
        .map(m => m.name);
}
Java
JAVA
import java.util.ArrayList;
import java.util.List;

class Solution {
    static class Member {
        int id;
        String name;
        int inviter;   // 0 means "no inviter"
        Member(int id, String name, int inviter) {
            this.id = id; this.name = name; this.inviter = inviter;
        }
    }

    public List<String> notInvitedBy(List<Member> members, int target) {
        List<String> result = new ArrayList<>();
        for (Member m : members) {
            if (m.inviter != target) {
                result.add(m.name);
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

struct Member {
    int id;
    string name;
    int inviter;   // 0 means "no inviter"
};

class Solution {
public:
    vector<string> notInvitedBy(const vector<Member>& members, int target) {
        vector<string> result;
        for (const Member& m : members) {
            if (m.inviter != target) {
                result.push_back(m.name);
            }
        }
        return result;
    }
};
C
C
#include <stddef.h>

// Fills `out` (caller-allocated, size >= n) with pointers to the names of members
// whose inviter != target. `inviter[i] == 0` means the member has no inviter.
// Returns the number of names written.
int notInvitedBy(const char** names, const int* inviter, int n, int target,

#### cons
t char** out) {
    int k = 0;
    for (int i = 0; i < n; i++) {
        if (inviter[i] != target) {
            out[k++] = names[i];
        }
    }
    return k;
}

#### Follow-up
questions
Exclude members invited by any id in a given set of targets rather than a single t.
Also return, for each kept member, the name of their inviter (requires an id-to-name lookup / self-join).
Handle the "no inviter" case explicitly with proper NULL semantics and discuss the null-safe inequality in SQL.
Return the count of qualifying members instead of the list, and do it without materializing the names.

