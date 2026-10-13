---
type: "practice"
difficulty: "Other"
---

# Earliest Session Date Per Member

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Hash Map](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Hash_Map.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a list of sessions. Each session record describes one time a member signed in, played some rounds, and signed out. A session has four fields:

memberId — an integer identifying the member,
kioskId — an integer identifying the machine the member used,
sessionDate — the calendar date of the session, given as an ISO date string in YYYY-MM-DD form,
roundsPlayed — a non-negative integer count of rounds played that day (may be 0).

A member may appear in many sessions, but no member has two sessions on the same date (the pair (memberId, sessionDate) is unique across the list).

For every distinct member that appears in the list, report their first session date — the earliest sessionDate among all of that member's sessions. Produce one output pair (memberId, firstDate) per distinct member. The output may be returned in any order.

Because the dates are stored as zero-padded ISO strings (YYYY-MM-DD), comparing them lexicographically is the same as comparing them chronologically, so no date parsing is required.

#### Cons
traints

0 <= sessions.length <= 100000.
memberId and kioskId are non-negative 32-bit signed integers.
0 <= roundsPlayed <= 10^6.
Every sessionDate is a valid date in YYYY-MM-DD format.
For a fixed member, all sessionDate values are distinct.

#### Example
s & test cases
TEXT
Input: sessions = [ (memberId:1, kioskId:2, sessionDate:"2016-03-01", roundsPlayed:5), (1, 2, "2016-05-02", 6), (2, 3, "2017-06-25", 1), (3, 1, "2016-03-02", 0), (3, 4, "2018-07-03", 5) ]
-> Output: [ (1, "2016-03-01"), (2, "2017-06-25"), (3, "2016-03-02") ]
Why: member 1's two dates -> min is 2016-03-01; member 2 has one date; member 3's two dates -> min is 2016-03-02.

Input: sessions = []
-> Output: []
Why: no sessions means no members and therefore no output rows.

Input: sessions = [ (7, 9, "2021-12-31", 3) ]
-> Output: [ (7, "2021-12-31") ]
Why: a single session — its date is trivially the member's earliest.

Input: sessions = [ (4, 1, "2020-02-09", 0), (4, 1, "2019-11-30", 2), (4, 8, "2020-01-01", 7) ]
-> Output: [ (4, "2019-11-30") ]
Why: all sessions belong to one member; the minimum date wins even though it is not the first record listed.

Input: sessions = [ (10, 5, "2022-06-05", 4), (11, 6, "2022-06-05", 4), (10, 5, "2022-06-04", 1) ]
-> Output: [ (10, "2022-06-04"), (11, "2022-06-05") ]
Why: two members can share the same date; each member's earliest is tracked independently.

Input: sessions = [ (2, 3, "2000-01-01", 0), (2, 3, "2000-01-02", 0), (2, 3, "1999-12-31", 0) ]
-> Output: [ (2, "1999-12-31") ]
Why: the earliest date crosses a year boundary; ISO strings still compare correctly (1999 < 2000).

#### Approach

This is a group-by-minimum aggregation. Maintain a hash map keyed by memberId whose value is the smallest sessionDate seen so far for that member. Iterate once over all sessions: on the first sighting of a member, store its date; on later sightings, replace the stored date only when the new date is smaller. Because the dates are zero-padded ISO strings, ordinary string comparison equals chronological order, so no date parsing is needed. In SQL this is exactly SELECT memberId, MIN(sessionDate) FROM sessions GROUP BY memberId; the hash map is how an engine executes that GROUP BY ... MIN in memory. The pattern generalizes to any keyed running aggregate (min/max/sum/count) computed in one streaming pass.

Complexity

Time: O(n) where n is the number of session records — a single pass, each record doing an O(1) average hash lookup and an O(1) date-string comparison (the date has fixed length 10). Space: O(k) where k is the number of distinct members, for the hash map holding one date per member.

Python
PYTHON
def earliest_session_date(sessions):
    """First (earliest) sessionDate per memberId.

    sessions: list of dicts with keys memberId, kioskId, sessionDate, roundsPlayed.
    ISO YYYY-MM-DD strings compare lexicographically == chronologically.
    """
    earliest = {}
    for s in sessions:
        member, date = s["memberId"], s["sessionDate"]
        if member not in earliest or date < earliest[member]:
            earliest[member] = date
    return list(earliest.items())
JavaScript
JAVASCRIPT
function earliestSessionDate(sessions) {
  // First (earliest) sessionDate per memberId; ISO date strings sort chronologically.

#### cons
t earliest = new Map();
  for (const s of sessions) {

#### cons
t cur = earliest.get(s.memberId);
    if (cur === undefined || s.sessionDate < cur) earliest.set(s.memberId, s.sessionDate);
  }
  return [...earliest.entries()].map(([memberId, firstDate]) => ({ memberId, firstDate }));
}
Java
JAVA
import java.util.*;

class Solution {
    static class Session {
        int memberId, kioskId, roundsPlayed;
        String sessionDate;
        Session(int m, int k, String d, int r) {
            memberId = m; kioskId = k; sessionDate = d; roundsPlayed = r;
        }
    }

    // Earliest sessionDate per member in O(n); ISO strings compare chronologically.
    public Map<Integer, String> earliestSessionDate(List<Session> sessions) {
        Map<Integer, String> earliest = new HashMap<>();
        for (Session s : sessions) {
            String cur = earliest.get(s.memberId);
            if (cur == null || s.sessionDate.compareTo(cur) < 0)
                earliest.put(s.memberId, s.sessionDate);
        }
        return earliest;
    }
}
C++
CPP
#include <string>
#include <vector>
#include <utility>
#include <unordered_map>
using namespace std;

struct Session { int memberId; int kioskId; string sessionDate; int roundsPlayed; };

// Earliest sessionDate per member in O(n); ISO dates compare lexicographically.
vector<pair<int, string>> earliestSessionDate(const vector<Session>& sessions) {
    unordered_map<int, string> earliest;
    earliest.reserve(sessions.size() * 2 + 1);
    for (const auto& s : sessions) {
        auto it = earliest.find(s.memberId);
        if (it == earliest.end() || s.sessionDate < it->second)
            earliest[s.memberId] = s.sessionDate;
    }
    return vector<pair<int, string>>(earliest.begin(), earliest.end());
}
C
C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdbool.h>

typedef struct { int memberId; int kioskId; char *sessionDate; int roundsPlayed; } Session;
typedef struct { int memberId; char *firstDate; } Result;

typedef struct { int key; char *date; bool used; } Slot;

/* For each distinct memberId, keep the lexicographically smallest sessionDate
   (== chronologically earliest for ISO YYYY-MM-DD). Open-addressing hash map. */
Result *earliestSessionDate(Session *sessions, int n, int *outN) {
    int cap = 4;
    while (cap < (n + 1) * 2) cap <<= 1;
    Slot *tab = (Slot *)calloc(cap, sizeof(Slot));
    int distinct = 0;
    for (int i = 0; i < n; i++) {
        unsigned int h = (unsigned int)sessions[i].memberId * 2654435761u & (cap - 1);
        while (tab[h].used && tab[h].key != sessions[i].memberId) h = (h + 1) & (cap - 1);
        if (!tab[h].used) {
            tab[h].used = true;
            tab[h].key  = sessions[i].memberId;
            tab[h].date = sessions[i].sessionDate;
            distinct++;
        } else if (strcmp(sessions[i].sessionDate, tab[h].date) < 0) {
            tab[h].date = sessions[i].sessionDate;
        }
    }
    Result *res = (Result *)malloc(sizeof(Result) * (distinct > 0 ? distinct : 1));
    int k = 0;
    for (int i = 0; i < cap; i++)
        if (tab[i].used) { res[k].memberId = tab[i].key; res[k].firstDate = tab[i].date; k++; }
    free(tab);
    *outN = distinct;
    return res;
}

#### Follow-up
questions
Also return the kioskId used on that first session (a group-by that carries a companion column, not just the min).
Report each member's earliest date together with their total roundsPlayed across all sessions, in the same pass.
What changes if a member could have two sessions on the same date (the uniqueness guarantee is dropped)?
Instead of the earliest date, find members whose first session is also their only session.

