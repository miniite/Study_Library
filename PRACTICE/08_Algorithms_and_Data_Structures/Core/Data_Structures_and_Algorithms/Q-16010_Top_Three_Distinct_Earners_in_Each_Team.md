---
type: "practice"
difficulty: "Other"
---

# Top Three Distinct Earners in Each Team

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Hash Map](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Hash_Map.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A company keeps two collections of records and wants to know, for every team, who its best-paid members are.

You are given:

A list of members. Each member has a name (text), a pay (integer), and a teamId (the id of the team the member belongs to).
A list of teams. Each team has a teamId (unique) and a teamName (text).

For each team, a member is a top earner if that member's pay is among the top three distinct pay values within that team. Note that "distinct" is measured over the pay values, not over the members: if several members share the same pay, they all rank together, and that shared value counts as a single entry when we pick the top three.

Return every member who is a top earner in their team, as a list of triples (teamName, name, pay). The order of the returned rows does not matter.

Clarifications:

If a team has fewer than three distinct pay values, then every member of that team is a top earner (there simply are not three tiers to fall below).
Ties are inclusive: if two members tie for the third-highest distinct pay, both are returned.
A member always references a team that exists in the teams list.

Input

members: a collection of records, each { name, pay, teamId }. May be empty.
teams: a collection of records, each { teamId, teamName }.

Output

A collection of records { teamName, name, pay }, one per top-earning member, in any order.

#### Cons
traints

0 <= number of members <= 10^4
1 <= number of teams <= 10^4
0 <= pay <= 10^9
Every teamId on a member matches some team's teamId.
No two members share the exact same combination of name, pay, and teamId.

#### Example
s & test cases
TEXT
Input: members = [(Alice,85,10),(Bob,80,20),(Carol,60,20),(Dan,90,10),(Eve,69,10),(Frank,85,10),(Gina,70,10)], teams = [(10,Blue),(20,Gold)] -> Output: [(Blue,Dan,90),(Blue,Alice,85),(Blue,Frank,85),(Blue,Gina,70),(Gold,Bob,80),(Gold,Carol,60)]
Why: Normal case. Team Blue distinct pays are 90,85,70,69; the top three distinct are 90,85,70, so Eve (69) is dropped while Alice and Frank both qualify at the tied value 85. Team Gold has only two distinct pays, so both members qualify.

Input: members = [(Ann,50,1),(Ben,50,1),(Cid,50,1)], teams = [(1,Solo)] -> Output: [(Solo,Ann,50),(Solo,Ben,50),(Solo,Cid,50)]
Why: All pays are equal, so there is a single distinct value. Fewer than three distinct pays means everyone is a top earner.

Input: members = [(Ron,30,7),(Sue,20,7),(Tim,20,7),(Uma,10,7)], teams = [(7,Ops)] -> Output: [(Ops,Ron,30),(Ops,Sue,20),(Ops,Tim,20),(Ops,Uma,10)]
Why: Distinct pays are 30,20,10 — exactly three — so every member qualifies, and the two members tied at 20 (the second-highest distinct value) are both kept.

Input: members = [(Xi,100,3),(Yo,90,3),(Zed,80,3),(Wo,70,3)], teams = [(3,Core)] -> Output: [(Core,Xi,100),(Core,Yo,90),(Core,Zed,80)]
Why: Four distinct pays; only the top three distinct (100,90,80) survive, so Wo (70) is excluded — the core threshold test.

Input: members = [], teams = [(5,Empty)] -> Output: []
Why: Edge case with no members; a team that exists but has nobody yields no rows.

#### Approach

Group by team, then keep a bounded "top-K distinct" tracker per group with K = 3. Pass one: for every member, feed its pay into its team's tracker, which maintains at most the three largest DISTINCT pay values in descending order (duplicates are ignored, an out-of-range smaller value is dropped). Pass two: a member is a top earner exactly when its pay is at least the smallest value currently held by its team's tracker — that value is the k-th highest distinct pay where k = min(3, distinct count), so any pay meeting or exceeding it lies in the top three tiers, and ties are naturally included. The bounded tracker turns what looks like a per-group sort into O(1) work per element, giving overall linear time. This is the group-by-then-select-top-K pattern (a windowed ranking / dense-rank filter expressed with hashing instead of a full ordering).

Complexity

Time: O(N + T), where N is the number of members and T the number of teams. Each member is touched a constant number of times (two passes), and every tracker update is O(1) because each tracker holds at most three values. Space: O(T) for the per-team name map and the top-3 trackers; the output is at most O(N).

Python
PYTHON
from typing import List, Tuple

def top_distinct_earners(members: List[Tuple[str, int, int]],
                         teams: List[Tuple[int, str]]) -> List[Tuple[str, str, int]]:
    # members: list of (name, pay, team_id); teams: list of (team_id, team_name)
    name_of = {team_id: team_name for team_id, team_name in teams}
    top = {team_id: [] for team_id, _ in teams}   # team_id -> up to 3 distinct pays, desc

    def add(vals: List[int], p: int) -> None:
        if p in vals:                             # value already tracked
            return
        pos = len(vals)
        while pos > 0 and vals[pos - 1] < p:
            pos -= 1
        if pos >= 3:
            return
        vals.insert(pos, p)
        del vals[3:]                              # keep at most three

    for _, pay, team_id in members:
        add(top[team_id], pay)

    result = []
    for name, pay, team_id in members:
        vals = top.get(team_id)
        if not vals:
            continue
        if pay >= vals[-1]:                       # >= smallest tracked top value
            result.append((name_of[team_id], name, pay))
    return result
JavaScript
JAVASCRIPT
function topDistinctEarners(members, teams) {
  // members: [{ name, pay, teamId }], teams: [{ teamId, teamName }]

#### cons
t nameOf = new Map();

#### cons
t top = new Map(); // teamId -> array of up to 3 distinct pays, descending
  for (const t of teams) {
    nameOf.set(t.teamId, t.teamName);
    top.set(t.teamId, []);
  }

#### cons
t add = (vals, p) => {
    if (vals.includes(p)) return; // value already tracked
    let pos = vals.length;
    while (pos > 0 && vals[pos - 1] < p) pos--;
    if (pos >= 3) return;
    vals.splice(pos, 0, p);
    if (vals.length > 3) vals.pop();
  };

  for (const m of members) add(top.get(m.teamId), m.pay);

#### cons
t res = [];
  for (const m of members) {

#### cons
t vals = top.get(m.teamId);
    if (!vals || vals.length === 0) continue;
    if (m.pay >= vals[vals.length - 1]) {
      res.push({ teamName: nameOf.get(m.teamId), memberName: m.name, pay: m.pay });
    }
  }
  return res;
}
Java
JAVA
import java.util.*;

class Solution {
    public static class Member {
        String name; int pay; int teamId;
        public Member(String name, int pay, int teamId) {
            this.name = name; this.pay = pay; this.teamId = teamId;
        }
    }
    public static class Team {
        int teamId; String name;
        public Team(int teamId, String name) { this.teamId = teamId; this.name = name; }
    }
    public static class Earner {
        String teamName, memberName; int pay;
        public Earner(String teamName, String memberName, int pay) {
            this.teamName = teamName; this.memberName = memberName; this.pay = pay;
        }
    }

    public List<Earner> topDistinctEarners(List<Member> members, List<Team> teams) {
        Map<Integer, String> nameOf = new HashMap<>();
        Map<Integer, int[]>  top    = new HashMap<>();      // teamId -> {v0,v1,v2,count}
        for (Team t : teams) {
            nameOf.put(t.teamId, t.name);
            top.put(t.teamId, new int[]{0, 0, 0, 0});
        }
        for (Member m : members) add(top.get(m.teamId), m.pay);

        List<Earner> res = new ArrayList<>();
        for (Member m : members) {
            int[] tr = top.get(m.teamId);
            if (tr == null || tr[3] == 0) continue;
            int threshold = tr[tr[3] - 1];                  // smallest tracked top value
            if (m.pay >= threshold)
                res.add(new Earner(nameOf.get(m.teamId), m.name, m.pay));
        }
        return res;
    }

    private void add(int[] tr, int p) {
        int count = tr[3];
        for (int i = 0; i < count; i++) if (tr[i] == p) return;   // already tracked
        int pos = count;
        while (pos > 0 && tr[pos - 1] < p) pos--;
        if (pos >= 3) return;
        int last = Math.min(count, 2);
        for (int i = last; i > pos; i--) tr[i] = tr[i - 1];
        tr[pos] = p;
        if (count < 3) tr[3] = count + 1;
    }
}
C++
CPP
#include <vector>
#include <string>
#include <unordered_map>
using namespace std;

struct Member { string name; int pay; int teamId; };
struct Team   { int teamId; string name; };
struct Earner { string teamName; string memberName; int pay; };

class Solution {
public:
    vector<Earner> topDistinctEarners(const vector<Member>& members,

#### cons
t vector<Team>& teams) {
        unordered_map<int, string> nameOf;
        for (const auto& t : teams) nameOf[t.teamId] = t.name;

        // teamId -> up to 3 distinct pays kept in descending order
        unordered_map<int, vector<int>> top;
        for (const auto& t : teams) top[t.teamId];          // ensure entry exists
        for (const auto& m : members) add(top[m.teamId], m.pay);

        vector<Earner> res;
        for (const auto& m : members) {
            auto it = top.find(m.teamId);
            if (it == top.end() || it->second.empty()) continue;
            int threshold = it->second.back();              // smallest tracked top value
            if (m.pay >= threshold)
                res.push_back({nameOf[m.teamId], m.name, m.pay});
        }
        return res;
    }

private:
    void add(vector<int>& vals, int p) {
        for (int v : vals) if (v == p) return;              // already tracked
        int pos = (int)vals.size();
        while (pos > 0 && vals[pos - 1] < p) pos--;
        if (pos >= 3) return;
        vals.insert(vals.begin() + pos, p);
        if ((int)vals.size() > 3) vals.pop_back();
    }
};
C
C
#include <stdlib.h>
#include <string.h>

/* Output container: parallel arrays of size `size`. */
typedef struct {

#### cons
t char** teamName;

#### cons
t char** memberName;
    int*         pay;
    int          size;
} EarnerResult;

/* Open-addressing slot: teamId -> team name + a tracker of the top-3 distinct pays. */
typedef struct {
    int         teamId;
    char        used;

#### cons
t char* name;
    int         vals[3];   /* top-3 distinct pays, kept in descending order */
    int         count;     /* how many of vals are filled (0..3)          */
} TeamSlot;

static unsigned int hashInt(int k, unsigned int mask) {
    unsigned int x = (unsigned int)k * 2654435761u;
    x ^= x >> 15;
    return x & mask;
}

static TeamSlot* findSlot(TeamSlot* table, unsigned int mask, int teamId) {
    unsigned int i = hashInt(teamId, mask);
    while (table[i].used && table[i].teamId != teamId) i = (i + 1) & mask;
    return &table[i];
}

/* Insert pay p into a team's descending top-3-distinct tracker. */
static void trackerAdd(TeamSlot* s, int p) {
    for (int i = 0; i < s->count; i++)
        if (s->vals[i] == p) return;          /* value already tracked */
    int pos = s->count;
    while (pos > 0 && s->vals[pos - 1] < p) pos--;
    if (pos >= 3) return;                     /* smaller than all three and full */
    int last = (s->count < 3) ? s->count : 2;
    for (int i = last; i > pos; i--) s->vals[i] = s->vals[i - 1];
    s->vals[pos] = p;
    if (s->count < 3) s->count++;
}

EarnerResult topDistinctEarners(const char** teamNames, const int* teamIds, int teamCount,

#### cons
t char** memberNames, const int* memberPays,

#### cons
t int* memberTeam, int memberCount) {
    unsigned int cap = 1;
    while (cap < (unsigned int)(teamCount * 2 + 1)) cap <<= 1;
    unsigned int mask = cap - 1;
    TeamSlot* table = (TeamSlot*)calloc(cap, sizeof(TeamSlot));

    for (int i = 0; i < teamCount; i++) {
        TeamSlot* s = findSlot(table, mask, teamIds[i]);
        s->used = 1; s->teamId = teamIds[i]; s->name = teamNames[i]; s->count = 0;
    }
    for (int i = 0; i < memberCount; i++) {
        TeamSlot* s = findSlot(table, mask, memberTeam[i]);
        if (s->used) trackerAdd(s, memberPays[i]);
    }

    int room = memberCount > 0 ? memberCount : 1;
    EarnerResult r;
    r.teamName   = (const char**)malloc(sizeof(char*) * room);
    r.memberName = (const char**)malloc(sizeof(char*) * room);
    r.pay        = (int*)malloc(sizeof(int) * room);
    r.size = 0;

    for (int i = 0; i < memberCount; i++) {
        TeamSlot* s = findSlot(table, mask, memberTeam[i]);
        if (!s->used || s->count == 0) continue;
        int threshold = s->vals[s->count - 1];   /* min of the tracked top values */
        if (memberPays[i] >= threshold) {
            r.teamName[r.size]   = s->name;
            r.memberName[r.size] = memberNames[i];
            r.pay[r.size]        = memberPays[i];
            r.size++;
        }
    }
    free(table);
    return r;
}

#### Follow-up
questions
Generalize to the top K distinct earners per team instead of exactly three — where does the O(1) tracker stop being cheaper than sorting?
Return the teams sorted by name and, within a team, members sorted by descending pay then name; how does that change the complexity?
The data now streams in and does not fit in memory; maintain the answer online as members arrive and can also be removed.
Instead of the top three distinct pays, return members whose pay is strictly greater than the team's median pay.

