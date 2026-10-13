---
type: "practice"
difficulty: "Deep"
---

# Design a matchmaking service that pairs players by skill latency and queue time at scale — how do you balance match quality against wait time shard the pools by region and avoid starving edge-skill players?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Search Backend Design](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Search_Backend_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a high-quality matchmaking service, we need to ensure that players are paired based on their skill levels while minimizing wait times. This involves balancing match quality (ensuring close skill matchups) with wait time (minimizing latency and queue time). We'll leverage sharding by region to optimize performance and avoid starving edge-skill players.

#### Triaging the Question

The core of this question is about designing a system that can handle millions or even billions of players at scale. The key aspects are:

Match Quality: Ensuring skill balance in matches.
Wait Time: Minimizing queue time for players.
Sharding by Region: Partitioning player data to optimize local processing and reduce latency.

#### Assumptions
include:

Players can be matched across regions but prioritized within the same region.
There's a wide distribution of skill levels among players.
The system should handle dynamic updates in player skill levels frequently.

#### First Principles Solution
Player Skill Level Tracking: Each player has a unique ID, a skill level (e.g., 1 to 500), and a region ID.
Sharding by Region: Players are distributed across multiple shards based on their region IDs. This reduces the number of players in each shard and minimizes communication latency between regions.
Skill-Based Sharding: Within each region, we further subdivide players into skill-based shards to balance match quality against wait time.
Queue Management: A prioritized queue system ensures that lower-skill players have faster access to matches compared to higher-skill players.
Detailed Design

Shard Initialization:

Initialize a set of shards for each region, where the number of shards can be proportional to the population or skill variance in that region.

Skill-Based Partitioning:

For example, if we have 500 different skill levels (1-500), we might create 10 shards within a shard. Each shard will contain players with similar skill levels.

Queue Management:

Implement a FIFO queue for each skill-based shard. Lower-skill players get priority to reduce wait time.
Use a weighted random selection mechanism to ensure that higher-skill players still occasionally get matched, albeit less frequently than lower-skill players.

Matchmaking Algorithm:

When a player joins the system or requests a match, they are first assigned to their appropriate skill-based shard.
The algorithm then looks for nearby matches within this shard and assigns them based on the desired balance between skill level and wait time.
If no suitable local matches exist, it might expand its search to other shards in the same region.

Edge-Skill Player Handling:

Ensure that edge-skill players (both very high and very low) are not left out by prioritizing their requests more aggressively within their shard or slightly expanding their search radius.
Use a "cold-start" mechanism for new players who haven't played enough to get an accurate skill level, such as placing them in a special "observation" queue.

#### Implementation
Considerations
Scalability: Ensure the system can scale horizontally by adding more shards and regions. This might involve using cloud services like AWS or Azure.
Latency Optimization: Use local data centers or edge computing to minimize network latency between players.

#### Cons
istency: Implement consistent sharding strategies across all regions to ensure fair distribution of players.

#### Closing Summary

To summarize, the design involves a combination of regional and skill-based sharding to balance match quality with wait time. The key is to prioritize queues within shards and use weighted random selection to handle edge cases while ensuring scalability and low latency.

#### Common Mistakes/Follow-Ups
Avoiding Imbalanced Shards: Ensure that each shard has a balanced distribution of players to avoid hotspots.
Ignoring Edge Cases: Pay special attention to handling very high or very low skill players, as they can skew matchmaking in certain regions.
Overcomplicating the Algorithm: A simple and efficient algorithm is often better than a complex one with many edge cases.

**Comparison:**

```
Shard	Skill Range
0	95 - 104
1	96 - 105
2	97 - 106
...	...
```

**Code:**

```python
class Player:
    def __init__(self, player_id, skill_level, region):
        self.player_id = player_id
        self.skill_level = skill_level
        self.region = region

class MatchmakingService:
    def __init__(self, num_shards_per_region=10):
        self.shards = {}
        for i in range(num_shards_per_region):
            self.shards[i] = []

    def add_player(self, player):
        shard_index = (player.skill_level - 1) % len(self.shards)
        self.shards[shard_index].append(player)

    def find_match(self, player):
        shard_index = (player.skill_level - 1) % len(self.shards)
        for i in range(len(self.shards[shard_index])):
            if not should_match(self.shards[shard_index][i], player):  # Placeholder function
                return self.shards[shard_index][i]
        return None

def should_match(player1, player2):
    # Example matching logic: players are matched within a certain skill range.
    return abs(player1.skill_level - player2.skill_level) <= 50
```

**Example input / output:**

```text
Input:
- Player with skill level 100 in US-East

Output:
- Match found with player of skill level 110 (within the matching range)
```

**Data:**

```json
[
  {"player_id": 1, "skill_level": 100, "region": "US-East"},
  {"player_id": 2, "skill_level": 110, "region": "US-East"},
  {"player_id": 3, "skill_level": 95, "region": "US-West"},
  {"player_id": 4, "skill_level": 105, "region": "US-West"}
]
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Player Joins
6. Shard Assignment
7. Is Player in Local Shard?
8. Match Found
9. Find Match Across Regions
10. Match Found?
11. Match
12. Retry or Observation Queue

