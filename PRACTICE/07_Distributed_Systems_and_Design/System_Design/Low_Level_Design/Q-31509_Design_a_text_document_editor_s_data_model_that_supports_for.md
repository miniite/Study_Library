---
type: "practice"
difficulty: "Deep"
---

# Design a text/document editor's data model that supports formatting like bold and headings.

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Extensible Rule Engines](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Extensible_Rule_Engines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a low-level data model for a text/document editor, we need to ensure it can handle complex formatting while maintaining clarity and efficiency. The core of this design involves representing documents as structured data, supporting various types of content (text, images, etc.), and managing different levels of formatting such as bold, headings, and inline styles.

#### Triage

The question asks for a low-level design that supports text/document editing with basic formatting capabilities like bold and headings. The interviewer is likely probing your understanding of data structures, efficient memory management, and how to represent complex content hierarchically. We will assume the system needs to support operations such as insertions, deletions, and updates while maintaining performance.

#### First-Principles Solution
Document Structure: Represent a document as a tree structure where each node can be a text element, an image, or another nested document (e.g., sections). Each text node will have properties like text content, formatting attributes, and children nodes.
Formatting Attributes: Define a set of standard formatting attributes such as bold, italic, underline, headingLevel (for headings).
Efficient Updates: Use event-driven architecture to handle changes efficiently without re-rendering the entire document.
Memory Management: Optimize memory usage by using lazy loading and efficient data structures.
Detailed Solution

Document Node Class:

PYTHON
class DocumentNode:
    def __init__(self, value=None):
        self.value = value  # Text content or other node reference
        self.formatting_attributes = {}  # Stores formatting attributes like bold, headingLevel
        self.children = []  # List of child nodes

    def add_child(self, child_node):
        if not isinstance(child_node, DocumentNode):
            raise TypeError("Child must be a DocumentNode")
        self.children.append(child_node)

    def remove_child(self, index):
        if 0 <= index < len(self.children):
            del self.children[index]
        else:
            raise IndexError("Invalid index")

    def update_attribute(self, attribute_name, value):
        self.formatting_attributes[attribute_name] = value

Document Class:

PYTHON
class Document:
    def __init__(self):
        self.root_node = DocumentNode()

    def insert_text(self, start_index, text, attributes=None):
        # Implementation for inserting text at a specific index

    def delete_text(self, start_index, end_index):
        # Implementation for deleting text between indices

Event System:

PYTHON
class EventSystem:
    def __init__(self):
        self.listeners = {}

    def subscribe(self, event_type, callback):
        if event_type not in self.listeners:
            self.listeners[event_type] = []
        self.listeners[event_type].append(callback)

    def publish(self, event_type, payload=None):
        if event_type in self.listeners:
            for listener in self.listeners[event_type]:
                listener(payload)

#### Closing Summary
and Common Mistakes

The design effectively supports a flexible document structure with rich text formatting. By using classes to encapsulate nodes and documents, we maintain clarity and ease of modification. The use of an event system ensures that changes can be efficiently propagated without unnecessary re-renders.

#### Common mistakes
include:

Not handling edge cases properly (e.g., incorrect indexing).
Failing to manage memory efficiently, leading to performance issues.
Overcomplicating the structure, making it hard to maintain and extend.

