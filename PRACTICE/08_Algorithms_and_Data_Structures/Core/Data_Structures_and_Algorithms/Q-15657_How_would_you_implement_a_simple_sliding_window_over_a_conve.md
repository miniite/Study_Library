---
type: "practice"
difficulty: "Intermediate"
---

# How would you implement a simple sliding window over a conversation to keep the token count under a limit, dropping the oldest turns first?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Prefix Sums and Sliding Windows](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Prefix_Sums_and_Sliding_Windows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to design a data structure that maintains a sliding window of recent conversation turns while ensuring the total token count does not exceed a specified limit. The core idea is to keep track of both the number of tokens and the order of messages, prioritizing older messages for removal when the token count exceeds the limit.

#### Triage

The question asks us to implement a mechanism that manages a sliding window over conversation turns, ensuring that the total token count remains under a specified threshold. This involves maintaining an ordered list or queue where we can efficiently add new items and remove old ones based on their age. The interviewer is probing our understanding of data structures, specifically queues and linked lists, and our ability to manage dynamic constraints.

#### First-Principles Solution

Understand the Requirements:

We need a data structure that maintains a sliding window over recent conversation turns.
The total token count must be kept under a specified limit ( T ).
When the token count exceeds ( T ), we drop the oldest turn(s) first.

Choose Appropriate Data Structure:

A queue is suitable because it allows us to maintain order and efficiently manage additions and removals.
We can use a doubly linked list with each node representing a conversation turn, containing the token count of that message.

Implement the Logic:

Maintain two pointers: head pointing to the oldest message in the window and tail pointing to the newest message added.
Use a hash map to store references to nodes for quick access when removing the head node.
Implement an insertion operation that adds new messages at the tail, updating token counts and checking if the total exceeds ( T ).

Handle Token Count Exceedance:

If adding a new message causes the token count to exceed ( T ), remove the oldest message (pointed by head) from both the queue and the hash map.
Adjust the token count accordingly.

Maintain Order and Efficiency:

Ensure that operations like insertion, deletion, and access are efficient to meet real-time constraints of a chat application.
Full Solution

Here is an implementation in Python:

PYTHON
class ConversationTurn:
    def __init__(self, message, token_count):
        self.message = message
        self.token_count = token_count

class SlidingWindow:
    def __init__(self, max_token_count):
        self.max_token_count = max_token_count
        self.queue = []
        self.head = None
        self.tail = None
        self.tokens_map = {}
        self.total_tokens = 0

    def add_turn(self, message, token_count):
        new_turn = ConversationTurn(message, token_count)

        if self.tail is not None:
            # Add the new turn at the tail of the queue
            self.tail.next = new_turn
            new_turn.prev = self.tail
            self.tail = new_turn
            self.queue.append(new_turn)
        else:
            self.head = self.tail = new_turn
            self.queue.append(new_turn)

        self.total_tokens += token_count

        # If total tokens exceed max_token_count, remove the oldest turn
        while self.total_tokens > self.max_token_count and self.head is not None:
            old_turn = self.head
            self.head = self.head.next
            if self.head is not None:
                self.head.prev = None
            else:
                self.tail = None
            del self.tokens_map[old_turn.message]
            self.total_tokens -= old_turn.token_count

        # Update the head's reference in the tokens map
        if self.head is not None:
            self.tokens_map[self.head.message] = self.head

    def get_oldest_token_count(self):
        return 0 if self.head is None else self.head.token_count

# Example usage
sliding_window = SlidingWindow(max_token_count=100)
messages = ["Hello!", "How are you?", "I'm good, thanks.", "This is a longer message that counts towards the token limit."]
tokens = [5, 5, 5, 20]

for i in range(len(messages)):
    sliding_window.add_turn(messages[i], tokens[i])
    print(f"Added: {messages[i]}, Current Token Count: {sliding_window.total_tokens}, Oldest Turn: {sliding_window.get_oldest_token_count()}")

#### Closing Summary
and Common Mistakes

This implementation ensures that the token count remains under a specified limit by managing a sliding window of recent messages. By using a doubly linked list, we can efficiently handle insertions and deletions in ( O(1) ) time on average.

#### Common Mistakes

Forgetting to update the total_tokens when adding or removing turns.
Not handling edge cases where there are no tokens to remove when exceeding the limit.
Mismanaging the pointers, especially during insertion and deletion operations.
Common Follow-Ups
Handling Edge Cases: How would your solution handle an empty queue?

#### Performance Considerations
What is the time complexity of adding a new turn? Removing an old turn?
Scalability: Can you scale this to handle thousands or millions of messages?

