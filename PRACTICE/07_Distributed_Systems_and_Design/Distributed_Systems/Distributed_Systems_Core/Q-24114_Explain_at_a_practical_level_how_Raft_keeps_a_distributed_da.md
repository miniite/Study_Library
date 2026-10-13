---
type: "practice"
difficulty: "Deep"
---

# Explain at a practical level how Raft keeps a distributed database's replicas consistent: leader election, log replication, and the commit rule. What subtlety around committing entries from a previous term can bite a database's durability guarantee?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Quorums and Consensus](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Quorums_and_Consensus.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Raft is a consensus algorithm used to maintain consistency across replicas in distributed systems. At its core, Raft ensures that all replicas eventually agree on the same log entries by electing a leader and replicating logs through a carefully orchestrated process. The key components are leader election, log replication, and the commit rule.

Firstly, let's break down each of these components:

Leader Election: This is how Raft selects one replica as the leader to coordinate the consensus process.
Log Replication: The leader replicates its logs to all other replicas, ensuring that they have the same log entries.
The Commit Rule: This rule determines when a log entry can be considered committed and applied to the database.

Now, let's dive into each component in detail:

1. Leader Election

In Raft, leaders are elected through a process called election. Each replica starts an election by sending a RequestVote message to other replicas with its current term number. A replica can vote for another if it has not voted in the current term or if it has already voted for that candidate. The candidate that receives more than half of the votes becomes the leader.

Subtleties around Leader Election:

Multiple Simultaneous Elections: If multiple replicas start elections at the same time, a raft might enter an indefinite election phase where no replica can win majority.
Timeouts and Heartbeats: Each replica periodically sends Heartbeat messages to maintain its leadership. If a replica does not receive any votes for a certain number of timeout intervals (a combination of heartbeat timeouts), it will assume the leader has failed and start an election.
2. Log Replication

Once elected, the leader is responsible for replicating log entries to all other replicas:

Append Entries RPC: The leader sends AppendEntries requests to each replica, containing a list of new log entries.
Replica Response: Each replica checks if these entries are consistent with its existing logs. If they are, it stores the new entries and acknowledges the request.

Subtleties around Log Replication:

Out-of-Order Entries: Logs can be out of order due to network partitions or failure recovery.
Conflict Resolution: When a replica receives conflicting entries, it chooses the later one with the higher term number. This ensures that the highest-term log entry wins.
3. The Commit Rule

The commit rule in Raft determines when an entry is considered committed and can be applied to the database:

Majority Acknowledgment: An entry is committed if it is stored in at least half of the replicas' logs.
Commit Index: When a replica successfully receives AppendEntries requests for an entry, it increments its commit index.

Subtleties around Committing Entries from a Previous Term: One subtle issue that can affect durability is when a leader changes during log replication. If the new leader has not yet received all the logs from previous terms, committing entries from those terms might lead to data loss or inconsistency.

Leader Change During Log Replication: If a leader fails and another replica becomes the leader before the old leader has caught up with its log, the new leader may discard some of the logs and make them unavailable for committing. This can break the durability guarantee if not handled properly.

#### Closing Summary

In summary, Raft ensures consistency by carefully managing leader election, log replication, and commit rules. However, subtleties such as out-of-order entries, leader change during log replication, and handling of previous term logs can challenge the database's durability guarantees. It is crucial to design robust systems that handle these edge cases effectively.

#### Common Mistakes
and Follow-Ups
Ignoring Leader Change: Not properly handling scenarios where a leader fails before catching up with its log entries.
Improper Log Replication: Failing to ensure that all replicas have the latest logs before committing new entries.
Concurrency Issues: Handling concurrent updates from multiple leaders or out-of-order entries.

**Comparison:**

```
Term	Command	Index
1	create_user	1
2	update_user	2
3	delete_user	3
```

**Code:**

```python
# Example of a simple Raft implementation pseudocode for leader election
class Raft:
    def start_election(self):
        self.voted_for = None  # Track if this server has voted in current term
        self.term = 0          # Incremented to new value by every election timeout

    def send_request_vote(self, peer_id):
        request = RequestVote(term=self.term, candidate_id=self.id)
        response = peer.request_vote(request)
        if response.granted:
            return True
        else:
            return False

# Example of a simple Raft implementation pseudocode for log replication
class Raft:
    def send_append_entries(self, peers):
        entries = self.log.entries  # Get the new log entries to replicate
        append_entries_request = AppendEntriesRequest(term=self.term, leader_id=self.id, entries=entries)
        responses = [peer.append_entries(append_entries_request) for peer in peers]
        if all([resp.success for resp in responses]):
            return True
        else:
            return False

# Example of a simple Raft implementation pseudocode for committing logs
class Raft:
    def commit_logs(self):
        committed_index = min(max(self.commit_index, self.match_index), len(self.log))
        for i in range(committed_index + 1):
            entry = self.log[i]
            # Apply the entry to the database state
```

**Example input / output:**

```text
Input:
- New log entries to replicate: [{"term": 3, "command": "delete_user", "index": 3}]

Output:
- After replication:
  - Leader's log: [{"term": 1, "command": "create_user", "index": 1}, {"term": 3, "command": "delete_user", "index": 3}]
  - Replica logs: [{"term": 1, "command": "create_user", "index": 1}, {"term": 2, "command": "update_user", "index": 2}]

- After commit:
  - Leader's log: [{"term": 1, "command": "create_user", "index": 1}, {"term": 3, "command": "delete_user", "index": 3}]
  - Replica logs: [{"term": 1, "command": "create_user", "index": 1}, {"term": 2, "command": "update_user", "index": 2}]
  - Commit index incremented to 3
```

**Data:**

```json
{
    "log": [
        {"term": 1, "command": "create_user", "index": 1},
        {"term": 2, "command": "update_user", "index": 2}
    ],
    "commit_index": 1,
    "match_index": [2]
}
```

**Flow:**

1. Leader Election
2. Timeout
3. Send RequestVote
4. Voted
5. Grant Vote
6. Return True
7. Did not vote in this term
8. Grant Vote
9. Return True
10. Heartbeat
11. Send Heartbeat to Replicas
12. Replica receives AppendEntries
13. Verify Entries
14. Store New Log
15. Increment Commit Index

