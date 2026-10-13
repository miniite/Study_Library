---
type: "practice"
difficulty: "Expert"
---

# Design a fan-out-on-write vs fan-out-on-read decision for a follower feed at the class level; how do you make the write path pluggable so celebrity accounts can use a different strategy than normal users?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Pub Sub Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Pub_Sub_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an efficient fan-out-on-write vs fan-out-on-read decision for a follower feed, we need to balance between real-time updates and storage optimization while ensuring that celebrity accounts can use a different strategy than normal users. This involves making the write path pluggable so that different strategies can be applied based on account type. Here’s how we approach this from first principles:

#### Triaging the Question

The key aspects of the question are:

Fan-Out-on-Write vs Fan-Out-on-Read: Choosing between writing to multiple followers simultaneously or reading follower data separately.
Pluggable Write Path: Ensuring that different account types (e.g., celebrity accounts) can use a specific write strategy.

#### First-Principles Solution

Understand the Use Cases:

Fan-Out-on-Write: Ideal for scenarios where real-time updates are critical, such as large-scale user interactions.
Fan-Out-on-Read: Useful for reducing write operations and improving performance by reading follower data on-demand.

Class-Level Design:

Define a FeedService class that handles the feed logic.
Use an abstract strategy pattern to make the write path pluggable.

Strategy Pattern Implementation:

Create different strategies for writing updates, e.g., FanOutOnWriteStrategy and FanOutOnReadStrategy.
Implement a FeedWriterFactory that returns the appropriate strategy based on the account type.

Account Type Handling:

Define an AccountType enum with values like NORMAL_USER, CELEBRITY, etc.
Use this enum to decide which write strategy to apply during feed updates.
Detailed Design
FeedService Class
PYTHON
class FeedService:
    def __init__(self, writer_factory: FeedWriterFactory):
        self.writer_factory = writer_factory

    def update_feed(self, user_id: str, new_post: Post) -> None:
        writer_strategy = self.writer_factory.get_writer_strategy(user_id)
        writer_strategy.write(new_post)
Writer Strategy Pattern
PYTHON
from abc import ABC, abstractmethod
from typing import List

class FeedWriterStrategy(ABC):
    @abstractmethod
    def write(self, post: Post) -> None:
        pass

class FanOutOnWriteStrategy(FeedWriterStrategy):
    def write(self, post: Post) -> None:
        # Implementation for fan-out-on-write
        print(f"Fan out on write strategy used. Writing to {len(followers)} followers.")

class FanOutOnReadStrategy(FeedWriterStrategy):
    def write(self, post: Post) -> None:
        # Implementation for fan-out-on-read
        print("Fan out on read strategy used. Reading follower data separately and updating.")

class FeedWriterFactory:
    @staticmethod
    def get_writer_strategy(user_id: str) -> FeedWriterStrategy:
        account_type = get_account_type(user_id)
        if account_type == "CELEBRITY":
            return FanOutOnWriteStrategy()
        else:
            return FanOutOnReadStrategy()

def get_account_type(user_id: str) -> str:
    # Assume this function fetches the user type from a database or cache
    return "NORMAL_USER"

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Common Mistakes

Failing to abstract the write path, which can lead to tightly coupled code.
Not considering edge cases like concurrent writes or performance implications of different strategies.

#### Follow-Ups

Discuss caching mechanisms for follower data.
Explore distributed systems and consistency models (e.g., eventual consistency vs. strong consistency).

#### Cons
ider latency requirements and how they impact the choice between fan-out-on-write and fan-out-on-read.

**Code:**

```python
class FeedService:
    def __init__(self, writer_factory: FeedWriterFactory):
        self.writer_factory = writer_factory

    def update_feed(self, user_id: str, new_post: Post) -> None:
        writer_strategy = self.writer_factory.get_writer_strategy(user_id)
        writer_strategy.write(new_post)

class FeedWriterStrategy(ABC):
    @abstractmethod
    def write(self, post: Post) -> None:
        pass

class FanOutOnWriteStrategy(FeedWriterStrategy):
    def write(self, post: Post) -> None:
        # Implementation for fan-out-on-write
        print(f"Fan out on write strategy used. Writing to {len(followers)} followers.")

class FanOutOnReadStrategy(FeedWriterStrategy):
    def write(self, post: Post) -> None:
        # Implementation for fan-out-on-read
        print("Fan out on read strategy used. Reading follower data separately and updating.")

class FeedWriterFactory:
    @staticmethod
    def get_writer_strategy(user_id: str) -> FeedWriterStrategy:
        account_type = get_account_type(user_id)
        if account_type == "CELEBRITY":
            return FanOutOnWriteStrategy()
        else:
            return FanOutOnReadStrategy()

def get_account_type(user_id: str) -> str:
    # Assume this function fetches the user type from a database or cache
    return "NORMAL_USER"
```

**Flow:**

1. No
2. Yes
3. User submits new post
4. Is user a celebrity?
5. Apply FanOutOnReadStrategy
6. Apply FanOutOnWriteStrategy

