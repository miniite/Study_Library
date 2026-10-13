---
type: "practice"
difficulty: "Other"
---

# Merge People With Their Location Records

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Hash Map](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Hash_Map.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given two lists of records that describe people and where they live.

The first list, people, contains one record per person. Each record has three fields:

id — a unique integer identifier for the person,
firstName — the person's first name,
lastName — the person's last name.

The second list, locations, contains address records. Each record has:

recordId — a unique integer identifier for the address record (it does not appear in the output),
personId — the id of the person this address belongs to,
city — the name of the city,
region — the name of the region/state.

Not every person has a matching address, and every person has at most one matching address record. Some address records may refer to a personId that does not appear in people (those addresses are simply ignored).

Produce one output row for every person in people — whether or not a matching address exists — containing firstName, lastName, city, and region. If the person has no matching address record, both city and region must be reported as null (None in Python; a "no value" marker in C/C++/Java as shown in the reference code). The output rows may be returned in any order.

This is exactly a left outer join of people onto locations keyed by the person id.

#### Cons
traints

0 <= people.length <= 100000
0 <= locations.length <= 100000
Every id and personId fits in a 32-bit signed integer.
Ids in people are unique; each personId appears at most once in locations.
firstName, lastName, city, region are non-empty strings of length up to 100.

#### Example
s & test cases
TEXT
Input: people = [ {id:1, firstName:"Ravi", lastName:"Sharma"}, {id:2, firstName:"Meera", lastName:"Nair"} ],
       locations = [ {recordId:1, personId:2, city:"Pune", region:"Maharashtra"}, {recordId:2, personId:9, city:"Austin", region:"Texas"} ]
-> Output: [ ("Ravi","Sharma",null,null), ("Meera","Nair","Pune","Maharashtra") ]
Why: person 1 has no address so city/region are null; the address for personId 9 matches nobody and is dropped.

Input: people = [ {id:5, firstName:"Lena", lastName:"Ito"} ], locations = []
-> Output: [ ("Lena","Ito",null,null) ]
Why: no address records at all, so every person gets nulls (a left join still emits the left row).

Input: people = [], locations = [ {recordId:1, personId:3, city:"Oslo", region:"Oslo"} ]
-> Output: []
Why: no people means no output rows regardless of how many addresses exist.

Input: people = [ {id:7, firstName:"Omar", lastName:"Ali"}, {id:8, firstName:"Sara", lastName:"Kim"} ],
       locations = [ {recordId:1, personId:8, city:"Seoul", region:"Seoul"}, {recordId:2, personId:7, city:"Cairo", region:"Cairo"} ]
-> Output: [ ("Omar","Ali","Cairo","Cairo"), ("Sara","Kim","Seoul","Seoul") ]
Why: both people have a match; address order does not matter because the join is keyed by id.

Input: people = [ {id:10, firstName:"Ana", lastName:"Cruz"}, {id:11, firstName:"Bo", lastName:"Ng"} ],
       locations = [ {recordId:1, personId:10, city:"Lima", region:"Lima"} ]
-> Output: [ ("Ana","Cruz","Lima","Lima"), ("Bo","Ng",null,null) ]
Why: mixed case — one person matched, the other unmatched with nulls, both still appear.

#### Approach

This is a left outer join. Build a hash map from personId to its (at most one) address record in a single pass over locations, then iterate over people in order and look each id up in O(1) average. If the lookup hits, copy city/region; if it misses, emit null/null. Because we iterate people (the left side) unconditionally, every person appears in the output exactly once, which is what distinguishes a left join from an inner join. In SQL this is literally Person LEFT JOIN Address ON Person.id = Address.personId; the hash-map lookup is how a relational engine executes that join in memory.

Complexity

Time: O(P + L), where P and L are the number of people and location records — one pass to index the locations by id and one pass over the people. Space: O(L) for the id-to-location hash map (plus O(P) for the output).

Python
PYTHON
def merge_people(people, locations):
    """Left-join people onto locations by id. Missing address -> (None, None)."""
    by_id = {loc["personId"]: loc for loc in locations}
    result = []
    for p in people:
        loc = by_id.get(p["id"])
        city = loc["city"] if loc else None
        region = loc["region"] if loc else None
        result.append((p["firstName"], p["lastName"], city, region))
    return result
JavaScript
JAVASCRIPT
function mergePeople(people, locations) {
  // Left-join people onto locations by id in O(P + L).

#### cons
t byId = new Map();
  for (const loc of locations) byId.set(loc.personId, loc);

  return people.map((p) => {

#### cons
t loc = byId.get(p.id);
    return {
      firstName: p.firstName,
      lastName: p.lastName,
      city: loc ? loc.city : null,
      region: loc ? loc.region : null,
    };
  });
}
Java
JAVA
import java.util.*;

class Solution {
    static class Person {
        int id; String firstName, lastName;
        Person(int id, String f, String l) { this.id = id; firstName = f; lastName = l; }
    }
    static class Location {
        int personId; String city, region;
        Location(int p, String c, String r) { personId = p; city = c; region = r; }
    }
    static class Row {
        String firstName, lastName, city, region; // city/region null when unmatched
        Row(String f, String l, String c, String r) { firstName = f; lastName = l; city = c; region = r; }
    }

    // Left-join people onto locations by id in O(P + L).
    public List<Row> mergePeople(List<Person> people, List<Location> locs) {
        Map<Integer, Location> byId = new HashMap<>();
        for (Location l : locs) byId.put(l.personId, l);

        List<Row> res = new ArrayList<>(people.size());
        for (Person p : people) {
            Location l = byId.get(p.id);
            res.add(new Row(p.firstName, p.lastName,
                            l == null ? null : l.city,
                            l == null ? null : l.region));
        }
        return res;
    }
}
C++
CPP
#include <string>
#include <vector>
#include <optional>
#include <unordered_map>
using namespace std;

struct Person   { int id; string firstName, lastName; };
struct Location  { int personId; string city, region; };
struct Row { string firstName, lastName; optional<string> city, region; };

// Left-join people onto locations by id in O(P + L).
vector<Row> mergePeople(const vector<Person>& people, const vector<Location>& locs) {
    unordered_map<int, const Location*> byId;
    byId.reserve(locs.size() * 2 + 1);
    for (const auto& l : locs) byId[l.personId] = &l;

    vector<Row> res;
    res.reserve(people.size());
    for (const auto& p : people) {
        Row r; r.firstName = p.firstName; r.lastName = p.lastName;
        auto it = byId.find(p.id);
        if (it != byId.end()) { r.city = it->second->city; r.region = it->second->region; }
        res.push_back(std::move(r));
    }
    return res;
}
C
C
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

typedef struct { int id; char *firstName; char *lastName; } Person;
typedef struct { int personId; char *city; char *region; } Location;
/* city/region are NULL when the person has no matching address record. */
typedef struct { char *firstName; char *lastName; char *city; char *region; } Row;

typedef struct { int key; int idx; bool used; } Slot;

/* Left-join `people` onto `locations` by id, using an open-addressing hash map. */
Row *mergePeople(Person *people, int pn, Location *locs, int ln, int *outN) {
    int cap = 4;
    while (cap < (ln + 1) * 2) cap <<= 1;
    Slot *tab = (Slot *)calloc(cap, sizeof(Slot));
    for (int i = 0; i < ln; i++) {
        unsigned int h = (unsigned int)locs[i].personId * 2654435761u & (cap - 1);
        while (tab[h].used && tab[h].key != locs[i].personId) h = (h + 1) & (cap - 1);
        tab[h].used = true; tab[h].key = locs[i].personId; tab[h].idx = i;
    }
    Row *res = (Row *)malloc(sizeof(Row) * (pn > 0 ? pn : 1));
    for (int i = 0; i < pn; i++) {
        res[i].firstName = people[i].firstName;
        res[i].lastName  = people[i].lastName;
        res[i].city = NULL; res[i].region = NULL;
        unsigned int h = (unsigned int)people[i].id * 2654435761u & (cap - 1);
        while (tab[h].used) {
            if (tab[h].key == people[i].id) {
                res[i].city   = locs[tab[h].idx].city;
                res[i].region = locs[tab[h].idx].region;
                break;
            }
            h = (h + 1) & (cap - 1);
        }
    }
    free(tab);
    *outN = pn;
    return res;
}

#### Follow-up
questions
What if a person can have more than one address? (Emit one output row per matching address, and one null-filled row if there are none.)
Return the rows sorted by lastName then firstName instead of any order — how does that change the complexity?
Also report how many people have no address on file, in the same pass.
The lists no longer fit in memory — how would you perform this join (external sort-merge join or partitioned hash join)?

