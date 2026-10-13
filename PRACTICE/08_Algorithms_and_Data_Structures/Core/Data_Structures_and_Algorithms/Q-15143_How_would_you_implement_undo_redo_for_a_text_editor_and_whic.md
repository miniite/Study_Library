---
type: "practice"
difficulty: "Intermediate"
---

# How would you implement undo/redo for a text editor and which data structures back it?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Exactly Once Dedup Structures](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Exactly_Once_Dedup_Structures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement undo/redo functionality in a text editor, we need to track state changes efficiently so that we can revert or replay them as needed. The core idea is to use two stacks: one for the history of actions and another for the redo operations. Here's how it works:

History Stack: This stack stores all the previous states of the document.
Redo Stack: This stack keeps track of undone actions that can be redone.

When a user performs an action (e.g., typing, deleting text), we push the current state onto the history stack and clear the redo stack. If the user wants to undo an action, we pop from the history stack and move the popped state into the redo stack. To redo an undone action, we simply pop from the redo stack and push it back onto the history stack.

Let's break down this solution step by step:

#### Triage
Question: How would you implement undo/redo for a text editor?
What is being probed:
Knowledge of data structures: Stacks.
Understanding of state management in software.

#### Trade-offs
between memory usage and performance.
Step-by-Step Solution

Data Structures Choice:

Stacks: Both history and redo stacks are used here because they allow us to maintain a linear sequence of operations, which is ideal for undo/redo functionality.

State Representation:

Each state can be represented as a snapshot of the document at that point in time.
For simplicity, we can store the entire text content or just changes (e.g., insertions and deletions).

Undo Operation:

When the user wants to undo an action, we pop from the history stack into the redo stack and update the current state of the document.

Redo Operation:

When the user wants to redo an action, we pop from the redo stack back into the history stack and update the current state of the document.

Handling Large Text Changes:

To optimize memory usage, consider storing only necessary information about changes rather than entire text snapshots.
For instance, record character insertions/deletions with their positions instead of reinserting large chunks of text.

#### Example
Code
JAVASCRIPT
class TextEditor {

#### cons
tructor() {
    this.history = []; // Stack to store document states
    this.redoStack = []; // Stack to store actions that can be redone
    this.currentText = ''; // Current state of the document
  }

  insert(text, pos) {
    if (this.history.length > 0 && !this.isRedoing()) {
      this.history.push(this.currentText);
      this.redoStack = [];
    }

#### cons
t currentText = this.currentText;
    this.currentText = currentText.slice(0, pos) + text + currentText.slice(pos);
    this.onAction();
  }

  delete(pos, length) {
    if (this.history.length > 0 && !this.isRedoing()) {
      this.history.push(this.currentText);
      this.redoStack = [];
    }

#### cons
t currentText = this.currentText;
    this.currentText = currentText.slice(0, pos) + currentText.slice(pos + length);
    this.onAction();
  }

  undo() {
    if (this.history.length === 0) return;

#### cons
t lastState = this.history.pop();
    this.redoStack.push(this.currentText);
    this.currentText = lastState;
  }

  redo() {
    if (this.redoStack.length === 0) return;

#### cons
t lastRedoAction = this.redoStack.pop();
    this.history.push(this.currentText);
    this.currentText = lastRedoAction;
  }

  onAction() {
    // Log action or update GUI
  }
}

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: Using two stacks—history for undo and redo for redo—provides a robust way to implement undo/redo functionality.

#### Common Mistakes
Overusing memory by storing entire text snapshots instead of just changes.
Not handling edge cases like initial state or concurrent editing operations.

#### Follow-Ups
Concurrency: How would you handle multiple users editing the same document simultaneously?
Performance Optimization: What are some ways to optimize memory usage when dealing with large documents?
Undo Limits: How do you manage the size of history and redo stacks?

**Code:**

```javascript
class TextEditor {
  // ... (same as above)
}
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD;
3. A[Current Text] --> B1(Insert "abc", 3)
4. B1 --> C1(Current Text with "abc" inserted at pos 3)
5. C1 --> D[History Stack = [""]]
6. D --> E[Redo Stack = []]
7. F1[Redo Stack = [""]]
8. F1 --> G1(Insert "def", 6)
9. G1 --> H1(Current Text with "def" inserted at pos 6)
10. H1 --> I[History Stack = [C1]]
11. I --> J[Redo Stack = []]
12. K1[Redo Stack = [""]]
13. K1 --> L1(Undo: Pop from History)
14. L1 --> M1(Current Text reverts to state C1)
15. M1 --> N[Redo Stack = [H1]]

