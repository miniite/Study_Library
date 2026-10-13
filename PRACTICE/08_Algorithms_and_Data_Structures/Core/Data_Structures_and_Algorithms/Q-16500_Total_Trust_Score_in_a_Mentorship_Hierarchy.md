---
type: "practice"
difficulty: "Other"
---

# Total Trust Score in a Mentorship Hierarchy

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Depth-First Search](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Depth-First_Search.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

An online community organizes its members into a mentorship hierarchy. Every member has:

a unique integer id,
an integer trust score (which may be negative), and
a list of the ids of the members they directly mentor (their direct mentees).

Each member is directly mentored by at most one other member, so the relationships form a forest of trees (there are no cycles). A member's indirect mentees are the mentees of their mentees, and so on, all the way down.

You are given the full list of members and a single query id. Return the total trust score obtained by adding together the trust score of the queried member and the trust scores of every one of their direct and indirect mentees.

Input:

members: a list where each entry describes one member as [id, trust, [menteeId1, menteeId2, ...]].
queryId: the id of the member whose subtree total you must compute.

Output:

A single integer: the sum of the trust score of the queried member plus the trust scores of all members reachable by following the mentee links downward from that member.

#### Cons
traints:

1 <= members.length <= 2000
1 <= id <= 2000
All ids are unique.
-100 <= trust <= 100
Every member has at most one direct mentor and may have any number of direct mentees.
Every id listed in a member's mentee list is a valid member id.
The queried id is guaranteed to be a valid member id.

#### Example
s & test cases
TEXT
Input: members = [[10,4,[20,30]],[20,6,[]],[30,6,[]]], queryId = 10 -> Output: 16
Why: root 10 (4) plus both mentees 20 (6) and 30 (6) gives 4+6+6 = 16.

Input: members = [[7,2,[15]],[15,-8,[]]], queryId = 15 -> Output: -8
Why: member 15 is a leaf with a negative score and no mentees, so the total is just -8 (edge case: negative value, query at a leaf).

Input: members = [[1,50,[]]], queryId = 1 -> Output: 50
Why: single member with no mentees; the total is its own score.

Input: members = [[1,3,[2]],[2,4,[3]],[3,5,[]]], queryId = 1 -> Output: 12
Why: a deep chain 1 -> 2 -> 3; indirect mentees still count: 3+4+5 = 12.

Input: members = [[1,3,[2]],[2,4,[3]],[3,5,[]]], queryId = 2 -> Output: 9
Why: querying a middle member counts only its own subtree: 4+5 = 9 (member 1 above is excluded).

Input: members = [[5,-2,[6,7]],[6,-3,[]],[7,10,[]]], queryId = 5 -> Output: 5
Why: mix of positive and negative scores: -2 + (-3) + 10 = 5.

#### Approach

Build a hash map from member id to the member record so any member can be located in O(1). Then run a depth-first search starting at the queried id: the answer for a member is its own trust score plus the recursively computed totals of each of its direct mentees. Because the relationships form a forest (no cycles), a plain DFS/recursion visits every node in the queried subtree exactly once with no need for a visited set. The underlying pattern is tree/graph subtree aggregation via DFS with an id lookup table.

Complexity

Time: O(N) — each member is visited exactly once during the traversal (N = number of members). Space: O(N) for the id-to-member lookup plus O(H) recursion stack, where H is the depth of the hierarchy (O(N) worst case).

Python
PYTHON
from typing import List

class Member:
    def __init__(self, uid: int, trust: int, mentees: List[int]):
        self.uid = uid
        self.trust = trust
        self.mentees = mentees

def total_trust(members: List['Member'], query_id: int) -> int:
    id_to_member = {m.uid: m for m in members}

    def dfs(uid: int) -> int:
        total = id_to_member[uid].trust
        for sub in id_to_member[uid].mentees:
            total += dfs(sub)
        return total

    return dfs(query_id)
JavaScript
JAVASCRIPT
class Member {

#### cons
tructor(uid, trust, mentees) {
        this.uid = uid;
        this.trust = trust;
        this.mentees = mentees;
    }
}

function totalTrust(members, queryId) {

#### cons
t idToMember = new Map();
    for (const m of members) idToMember.set(m.uid, m);

#### cons
t dfs = (uid) => {
        let total = idToMember.get(uid).trust;
        for (const sub of idToMember.get(uid).mentees) total += dfs(sub);
        return total;
    };

    return dfs(queryId);
}
Java
JAVA
import java.util.*;

class Member {
    int uid;
    int trust;
    List<Integer> mentees;
    Member(int uid, int trust, List<Integer> mentees) {
        this.uid = uid;
        this.trust = trust;
        this.mentees = mentees;
    }
}

class Solution {
    public int totalTrust(List<Member> members, int queryId) {
        Map<Integer, Member> idToMember = new HashMap<>();
        for (Member m : members)
            idToMember.put(m.uid, m);
        return dfs(queryId, idToMember);
    }

    private int dfs(int uid, Map<Integer, Member> idToMember) {
        int total = idToMember.get(uid).trust;
        for (int sub : idToMember.get(uid).mentees)
            total += dfs(sub, idToMember);
        return total;
    }
}
C++
CPP
#include <bits/stdc++.h>
using namespace std;

struct Member {
    int uid;
    int trust;
    vector<int> mentees;
};

class Solution {
 public:
  int totalTrust(vector<Member>& members, int queryId) {
    unordered_map<int, const Member*> idToMember;
    for (const Member& m : members)
      idToMember[m.uid] = &m;
    return dfs(queryId, idToMember);
  }

 private:
  int dfs(int uid, const unordered_map<int, const Member*>& idToMember) {
    int total = idToMember.at(uid)->trust;
    for (int sub : idToMember.at(uid)->mentees)
      total += dfs(sub, idToMember);
    return total;
  }
};
C
C
#include <stdlib.h>

typedef struct {
    int uid;
    int trust;
    int* mentees;    /* array of mentee ids */
    int menteeCount;
} Member;

/* idToIndex maps a member id to its position in the members array. */
static int dfs(int uid, Member* members, int* idToIndex) {
    Member* m = &members[idToIndex[uid]];
    int total = m->trust;
    for (int i = 0; i < m->menteeCount; i++)
        total += dfs(m->mentees[i], members, idToIndex);
    return total;
}

int totalTrust(Member* members, int membersSize, int queryId) {
    int maxId = 0;
    for (int i = 0; i < membersSize; i++)
        if (members[i].uid > maxId) maxId = members[i].uid;

    int* idToIndex = (int*)malloc(sizeof(int) * (maxId + 1));
    for (int i = 0; i < membersSize; i++)
        idToIndex[members[i].uid] = i;

    int ans = dfs(queryId, members, idToIndex);
    free(idToIndex);
    return ans;
}

#### Follow-up
questions
Convert the recursion to an explicit stack or BFS queue to avoid stack overflow on very deep hierarchies.
What if the relationships might accidentally contain a cycle? Add a visited set to guard against infinite recursion.
Support many repeated queries efficiently by precomputing every member's subtree total in one post-order pass and caching it.
Extend the answer to also return the number of subordinates or the average trust score per member in the subtree.

