---
type: "practice"
difficulty: "Deep"
---

# Your infinite-scroll feed re-renders the whole list on every append and janks on low-end phones; how do you redesign the component's data structure and keys to keep appends O(1) and avoid re-render storms?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Maintaining Denormalized Views](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Idempotent_Consumer_Design/Maintaining_Denormalized_Views.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of infinite-scroll feeds janking due to re-renders on every append, we need a redesign that ensures each new item is appended in constant time O(1) and minimizes unnecessary re-renders. The key idea here is to refactor our data structure and use unique keys for React components efficiently.

First, let's understand the current problem: every time an item is added, the entire list gets re-rendered, leading to performance issues on low-end devices. This is inefficient because only a single element needs updating or insertion.

#### Triaging the Question

The interviewer likely wants you to:

Understand how React handles re-renders.
Identify what changes can be made in terms of data structure and keying.
Discuss potential trade-offs between complexity, performance, and maintainability.
Explore edge cases and failure modes.

#### Assumptions
The feed is using a component-based approach with React or a similar library.
The current implementation uses an array to store items.
Each item in the list has unique data that can be used for keying.
Step-by-Step Solution from First Principles

Understand React’s Key Mechanism: In React, key is a special prop that you assign to uniquely identify each element in the DOM tree. React uses this key to figure out what has changed, been added, or been removed when it re-renders.

Refactor Data Structure: Instead of re-rendering the entire list on every append, we can use a more efficient data structure that allows us to insert and render new elements in constant time O(1). A doubly linked list or a balanced binary tree might be suitable candidates here.

Implement Unique Keys: For each item in the list, ensure it has a unique key prop. This can be generated based on a combination of the item's primary key and its position in the list.

Optimized Insertion Strategy: When appending an item:

Use the data structure to find where the new item should go.
Update only the necessary parts of the DOM without re-rendering everything.
Detailed Solution

Doubly Linked List Approach: A doubly linked list can be used to keep track of each node’s position, making insertions and deletions O(1). However, updating the UI might still involve traversing the list in some cases.

Balanced Binary Search Tree (BST) Approach: A BST allows for efficient insertion and search operations. Each node can store a unique key derived from its content, ensuring that React only re-renders nodes around the new item.

Hybrid Approach with Map: Use a map to track indices of items in the list and update it whenever an item is added or removed. This allows for O(1) insertion while minimizing re-renders.

Code Example
JAVASCRIPT
// Hypothetical Doubly Linked List Node Class
class Node {

#### cons
tructor(data, prev = null, next = null) {
        this.data = data;
        this.prev = prev;
        this.next = next;
    }
}

class LinkedList {

#### cons
tructor() {
        this.head = null;
        this.tail = null;
        this.size = 0;
    }

    insertAfter(prevNode, newNode) {
        // Logic to insert a new node after the given previous node
        if (prevNode === this.tail) {
            this.tail.next = newNode;
            newNode.prev = prevNode;
            newNode.next = null;
            this.tail = newNode;
        } else if (prevNode === null) {
            newNode.prev = null;
            newNode.next = this.head;
            this.head.prev = newNode;
            this.head = newNode;
        } else {
            newNode.next = prevNode.next;
            newNode.prev = prevNode;
            prevNode.next.prev = newNode;
            prevNode.next = newNode;
        }
        this.size++;
    }

    // Method to get the key for a node
    getKey(node) {
        return `${node.data.id}-${Date.now()}`;
    }

    renderItems() {
        let currentNode = this.head;
        while (currentNode !== null) {

#### cons
t key = this.getKey(currentNode);
            // Render each item with its unique key
            <div key={key}>{currentNode.data.content}</div>;
            currentNode = currentNode.next;
        }
    }
}

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: By using a data structure that supports O(1) insertions, such as a doubly linked list or a balanced BST, we can minimize re-renders. Ensuring each item has a unique key is crucial for React to handle partial updates efficiently.

#### Common Mistakes
Failing to account for the need for unique keys in React.
Not considering edge cases where items are added at specific positions.
Overcomplicating with unnecessary complex data structures without clear benefits.

Further Exploration: The interviewer might ask about trade-offs between these approaches, such as memory usage and complexity. You could discuss scenarios where a simpler approach like re-rendering the entire list is preferable for smaller lists or less performance-critical applications.

**Comparison:**

| Node | Prev Node | Next Node |
|------|-----------|-----------|
| N1   | null      | N2        |
| N2   | N1        | N3        |
| N3   | N2        | null      |

**Code:**

```javascript
class Node {
    constructor(data, prev = null, next = null) {
        this.data = data;
        this.prev = prev;
        this.next = next;
    }
}

class LinkedList {
    constructor() {
        this.head = null;
        this.tail = null;
        this.size = 0;
    }

    insertAfter(prevNode, newNode) {
        if (prevNode === this.tail) {
            this.tail.next = newNode;
            newNode.prev = prevNode;
            newNode.next = null;
            this.tail = newNode;
        } else if (prevNode === null) {
            newNode.prev = null;
            newNode.next = this.head;
            this.head.prev = newNode;
            this.head = newNode;
        } else {
            newNode.next = prevNode.next;
            newNode.prev = prevNode;
            prevNode.next.prev = newNode;
            prevNode.next = newNode;
        }
        this.size++;
    }

    getKey(node) {
        return `${node.data.id}-${Date.now()}`;
    }

    renderItems() {
        let currentNode = this.head;
        while (currentNode !== null) {
            const key = this.getKey(currentNode);
            <div key={key}>{currentNode.data.content}</div>;
            currentNode = currentNode.next;
        }
    }
}
```

**Example input / output:**

```text
// Example Render Output
<div key="1-1687904500000">Item 1</div>
<div key="2-1687904500000">Item 2</div>
<div key="3-1687904500000">Item 3</div>
```

**Data:**

```json
{
  "nodes": [
    { "id": 1, "content": "Item 1" },
    { "id": 2, "content": "Item 2" },
    { "id": 3, "content": "Item 3" }
  ]
}
```

**Flow:**

1. Node
2. Prev Node
3. Next Node

