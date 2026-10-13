---
type: "practice"
difficulty: "Basic"
---

# Design a menu system for a CLI/GUI where menus contain items or sub-menus. Which pattern models this tree?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ State Pattern](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/State_Pattern.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a menu system in both CLI and GUI environments, we need to model a hierarchical structure of menus and their items. The key is to represent this hierarchy efficiently and ensure that the design can easily support different user interfaces while maintaining consistency across them. A common pattern that models this tree-like structure effectively is the Command Pattern combined with Composite Pattern.

#### Triaging the Question

The question asks which design pattern(s) model a hierarchical menu system in both CLI and GUI environments. The core of the problem involves representing menus and items as nodes in a tree, where each node can either be an item or another menu (sub-menu). This naturally suggests using composite patterns to handle this hierarchy.

Step-by-Step Solution

Define the Menu Items:

Each menu item has basic properties like id, name, and potentially description.

Composite Pattern Implementation:

The Composite Pattern allows us to treat individual objects and compositions of objects uniformly.
We define a common interface for both leaf (menu items) and composite (sub-menus) nodes.

Command Pattern Integration:

Each menu item can have an associated action, which could be executed when the user selects it. The Command Pattern encapsulates these actions.

#### Implementation
Outline:

Define a base class or interface for all menu nodes.
Create subclasses for leaf and composite nodes.
Implement methods to handle the execution of commands.
Full Solution

Here's how we can implement this in code:

PYTHON
class MenuNode:
    def __init__(self, id, name):
        self.id = id
        self.name = name

    def execute(self):
        pass  # Base class does nothing, subclasses will override

class MenuItem(MenuNode):
    def __init__(self, id, name, command):
        super().__init__(id, name)
        self.command = command

    def execute(self):
        return self.command.execute()

class Menu(MenuNode):
    def __init__(self, id, name, items=None):
        super().__init__(id, name)
        if items is None:
            items = []
        self.items = items

    def add_item(self, item):
        self.items.append(item)

    def execute(self):
        for item in self.items:
            item.execute()

# Example Commands
class PrintCommand:
    @staticmethod
    def execute():
        print("Executing command")

# Create a menu system
root_menu = Menu(1, "Main Menu")
sub_menu = Menu(2, "Sub-menu")

item1 = MenuItem(3, "Item 1", PrintCommand())
item2 = MenuItem(4, "Item 2", PrintCommand())

sub_menu.add_item(item1)
sub_menu.add_item(item2)

root_menu.add_item(sub_menu)

# Execute the menu
root_menu.execute()

#### Closing Summary
and Common Mistakes/Follow-Ups

In this solution, we use a Composite Pattern to represent the hierarchical structure of menus and items. We also utilize the Command Pattern to encapsulate actions associated with each item. This design ensures that our implementation remains flexible and easy to extend for different user interfaces.

#### Common mistakes
include not properly separating concerns between the composite and leaf nodes, leading to tangled code. Also, failing to implement commands correctly can result in incomplete or incorrect functionality.

For follow-ups, one might explore how this system scales with many menus and items, performance optimizations, and integration with other parts of the application.

