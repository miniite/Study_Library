---
type: "practice"
difficulty: "Other"
---

# Agents With No Dealings for a Named Client

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Hash Set](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Hash_Set.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given three collections that describe a small sales system.

Agents — each agent record has:

agent_id (unique integer)
name (string)
base_salary (integer)
commission_rate (integer)
join_date (date)

Clients — each client record has:

client_id (unique integer)
name (string)
city (string)

Deals — each deal record has:

deal_id (unique integer)
deal_date (date)
client_id (integer, references a client)
agent_id (integer, references an agent)
amount (integer)

Given a target client name T, return the name of every agent who never closed a single deal with any client whose name equals T. An agent qualifies when none of their deals point at a client named T. This deliberately includes agents who have no deals at all, since they trivially never dealt with T.

Note that two or more distinct clients may happen to share the same name. If several clients are all named T, then a deal with any of them counts as a dealing with T.

Only agent_id/name, client_id/name, and each deal's (agent_id, client_id) are needed to compute the answer; the remaining columns are part of the schema but do not affect the result.

Return the list of agent names in any order.

#### Cons
traints

1 <= number of agents <= 10^3
0 <= number of clients <= 10^3
0 <= number of deals <= 10^4
All agent_id and client_id values are positive integers <= 10^6.
Every name is a non-empty string; names are compared case-sensitively and exactly.
Every deal's agent_id matches some agent and every deal's client_id matches some client.
The target name T is a non-empty string; it may match zero, one, or several clients.

Output: a list of agent names (in any order).

#### Example
s & test cases
TEXT
Notation: agents as `(id, name)`, clients as `(id, name)`, deals as `(agent_id, client_id)`.

1. Input: agents=`[(1,Nora),(2,Amit),(3,Marco),(4,Priya),(5,Alex)]`, clients=`[(1,AURORA),(2,CITRUS),(3,IVORY),(4,GROVE)]`, deals=`[(4,3),(5,4),(1,1),(4,1)]`, T=`AURORA` -> Output: `[Amit, Marco, Alex]`
   Why: only Nora(1) and Priya(4) have a deal with client 1 (AURORA); the other three qualify.

2. Input: agents=`[(1,Nora),(2,Amit)]`, clients=`[(1,CITRUS)]`, deals=`[(1,1),(2,1)]`, T=`AURORA` -> Output: `[Nora, Amit]`
   Why: no client is named AURORA, so no agent is ever excluded (target does not exist).

3. Input: agents=`[(1,Nora),(2,Amit),(3,Marco)]`, clients=`[(1,AURORA)]`, deals=`[(1,1)]`, T=`AURORA` -> Output: `[Amit, Marco]`
   Why: Nora dealt with AURORA; Amit and Marco have no deals at all and therefore qualify.

4. Input: agents=`[(1,Nora),(2,Amit)]`, clients=`[(1,AURORA)]`, deals=`[(1,1),(2,1)]`, T=`AURORA` -> Output: `[]`
   Why: every agent dealt with AURORA, so the result is empty (min-size output edge case).

5. Input: agents=`[(1,Nora),(2,Amit),(3,Marco)]`, clients=`[(1,AURORA),(2,AURORA),(3,GROVE)]`, deals=`[(1,1),(2,2),(3,3)]`, T=`AURORA` -> Output: `[Marco]`
   Why: clients 1 and 2 are both named AURORA; Nora and Amit each dealt with one of them, so only Marco (who dealt with GROVE) qualifies. Tests the duplicate-name union.

#### Approach

This is an anti-join (set-difference) expressed over three tables. First resolve the target name to the set of client ids that carry it (a name may map to several ids). Then scan the deals once and collect the id of every agent that touched one of those target clients into a "blocked" hash set. Finally return the agents whose id is not in the blocked set. Hash-set membership is O(1), so the whole computation is linear. Agents with no deals are handled for free: they never appear in the deal scan, so they are never blocked and are correctly included.

Complexity

Time: O(A + C + D), where A, C, D are the numbers of agents, clients, and deals. Space: O(A + C) for the two hash sets. Justification: one pass over clients marks the target-named ids, one pass over deals marks blocked agents, and one pass over agents produces the answer, each with O(1) hash-set operations.

Python
PYTHON
def agents_without_client(agents, clients, deals, target):
    """agents: list[(id, name)]  clients: list[(id, name)]  deals: list[(agent_id, client_id)]"""
    target_clients = {cid for cid, name in clients if name == target}
    blocked = {agent_id for agent_id, client_id in deals if client_id in target_clients}
    return [name for aid, name in agents if aid not in blocked]

if __name__ == "__main__":
    agents = [(1, "Nora"), (2, "Amit"), (3, "Marco"), (4, "Priya"), (5, "Alex")]
    clients = [(1, "AURORA"), (2, "CITRUS"), (3, "IVORY"), (4, "GROVE")]
    deals = [(4, 3), (5, 4), (1, 1), (4, 1)]
    print(agents_without_client(agents, clients, deals, "AURORA"))  # ['Amit', 'Marco', 'Alex']
JavaScript
JS
function agentsWithoutClient(agents, clients, deals, target) {
  // agents: [{id, name}]  clients: [{id, name}]  deals: [{agentId, clientId}]

#### cons
t targetClients = new Set(
    clients.filter((c) => c.name === target).map((c) => c.id)
  );

#### cons
t blocked = new Set(
    deals.filter((d) => targetClients.has(d.clientId)).map((d) => d.agentId)
  );
  return agents.filter((a) => !blocked.has(a.id)).map((a) => a.name);
}

// demo

#### cons
t agents = [
  { id: 1, name: "Nora" }, { id: 2, name: "Amit" }, { id: 3, name: "Marco" },
  { id: 4, name: "Priya" }, { id: 5, name: "Alex" },
];

#### cons
t clients = [
  { id: 1, name: "AURORA" }, { id: 2, name: "CITRUS" },
  { id: 3, name: "IVORY" }, { id: 4, name: "GROVE" },
];

#### cons
t deals = [
  { agentId: 4, clientId: 3 }, { agentId: 5, clientId: 4 },
  { agentId: 1, clientId: 1 }, { agentId: 4, clientId: 1 },
];

#### cons
ole.log(agentsWithoutClient(agents, clients, deals, "AURORA")); // ["Amit","Marco","Alex"]
Java
JAVA
import java.util.*;

public class Solution {
    record Agent(int id, String name) {}
    record Client(int id, String name) {}
    record Deal(int agentId, int clientId) {}

    static List<String> agentsWithoutClient(List<Agent> agents, List<Client> clients,
                                            List<Deal> deals, String target) {
        Set<Integer> targetClients = new HashSet<>();     // clients named target
        for (Client c : clients)
            if (c.name().equals(target)) targetClients.add(c.id());

        Set<Integer> blocked = new HashSet<>();           // agents who dealt with target
        for (Deal d : deals)
            if (targetClients.contains(d.clientId())) blocked.add(d.agentId());

        List<String> res = new ArrayList<>();
        for (Agent a : agents)
            if (!blocked.contains(a.id())) res.add(a.name());
        return res;
    }

    public static void main(String[] args) {
        List<Agent> agents = List.of(new Agent(1,"Nora"), new Agent(2,"Amit"),
                new Agent(3,"Marco"), new Agent(4,"Priya"), new Agent(5,"Alex"));
        List<Client> clients = List.of(new Client(1,"AURORA"), new Client(2,"CITRUS"),
                new Client(3,"IVORY"), new Client(4,"GROVE"));
        List<Deal> deals = List.of(new Deal(4,3), new Deal(5,4), new Deal(1,1), new Deal(4,1));
        System.out.println(agentsWithoutClient(agents, clients, deals, "AURORA")); // [Amit, Marco, Alex]
    }
}
C++
CPP
#include <bits/stdc++.h>
using namespace std;

struct Agent  { int id; string name; };
struct Client { int id; string name; };
struct Deal   { int agentId; int clientId; };

vector<string> agentsWithoutClient(const vector<Agent>& agents,

#### cons
t vector<Client>& clients,

#### cons
t vector<Deal>& deals,

#### cons
t string& target) {
    unordered_set<int> targetClients;          // ids of clients named target
    for (const auto& c : clients)
        if (c.name == target) targetClients.insert(c.id);

    unordered_set<int> blocked;                // agents who dealt with target
    for (const auto& d : deals)
        if (targetClients.count(d.clientId)) blocked.insert(d.agentId);

    vector<string> res;
    for (const auto& a : agents)
        if (!blocked.count(a.id)) res.push_back(a.name);
    return res;
}

int main() {
    vector<Agent>  agents  = {{1,"Nora"},{2,"Amit"},{3,"Marco"},{4,"Priya"},{5,"Alex"}};
    vector<Client> clients = {{1,"AURORA"},{2,"CITRUS"},{3,"IVORY"},{4,"GROVE"}};
    vector<Deal>   deals   = {{4,3},{5,4},{1,1},{4,1}};
    for (const string& n : agentsWithoutClient(agents, clients, deals, "AURORA"))
        cout << n << "\n";                     // Amit Marco Alex
    return 0;
}
C
C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct { int id; const char *name; } Agent;
typedef struct { int id; const char *name; } Client;
typedef struct { int agentId; int clientId; } Deal;

/* Writes the names of qualifying agents into out[] and returns their count.
   An agent qualifies if none of their deals involve a client named target. */
int agentsWithoutClient(const Agent *agents, int na,

#### cons
t Client *clients, int nc,

#### cons
t Deal *deals, int nd,

#### cons
t char *target, const char **out) {
    int maxId = 0;
    for (int i = 0; i < na; i++) if (agents[i].id  > maxId) maxId = agents[i].id;
    for (int i = 0; i < nc; i++) if (clients[i].id > maxId) maxId = clients[i].id;

    /* client id -> is it named target? */
    char *isTarget = (char *)calloc((size_t)maxId + 1, 1);
    for (int i = 0; i < nc; i++)
        if (strcmp(clients[i].name, target) == 0) isTarget[clients[i].id] = 1;

    /* agent id -> has a deal with a target-named client */
    char *blocked = (char *)calloc((size_t)maxId + 1, 1);
    for (int i = 0; i < nd; i++)
        if (deals[i].clientId <= maxId && isTarget[deals[i].clientId])
            blocked[deals[i].agentId] = 1;

    int count = 0;
    for (int i = 0; i < na; i++)
        if (!blocked[agents[i].id]) out[count++] = agents[i].name;

    free(isTarget);
    free(blocked);
    return count;
}

int main(void) {
    Agent  agents[]  = {{1,"Nora"},{2,"Amit"},{3,"Marco"},{4,"Priya"},{5,"Alex"}};
    Client clients[] = {{1,"AURORA"},{2,"CITRUS"},{3,"IVORY"},{4,"GROVE"}};
    Deal   deals[]   = {{4,3},{5,4},{1,1},{4,1}};

#### cons
t char *out[16];
    int k = agentsWithoutClient(agents, 5, clients, 4, deals, 4, "AURORA", out);
    for (int i = 0; i < k; i++) printf("%s\n", out[i]);   /* Amit Marco Alex */
    return 0;
}

#### Follow-up
questions
What if the target is given as a client id instead of a name, or if you must exclude agents who dealt with any of several target names at once?
Instead of "never dealt with T", return agents who dealt only with T and no other client.
What if agent names can repeat across different agent_ids — should the output deduplicate names or keep one per agent?
Extend to also report, for each qualifying agent, the total amount they billed to all other clients.

