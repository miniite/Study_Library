---
type: "practice"
difficulty: "Intermediate"
---

# Design a Command-pattern based remote control for smart-home devices. How do you support macros of commands?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ State Pattern](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/State_Pattern.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In designing a command-pattern-based remote control for smart-home devices, our primary goal is to abstract and encapsulate each device's actions into individual commands that can be composed into more complex operations or "macros." This approach not only enhances the modularity of the system but also simplifies debugging and future enhancements. For an intermediate experience level (backend SDE-2/3), we need to ensure our design supports dynamic macro creation, command chaining, and potential undo/redo functionalities.

#### Triaging the Question

The question is essentially asking how to implement a macro feature that can compose multiple commands into one logical unit. The interviewer might be probing your understanding of:

Command Pattern: How it works and its benefits.
Macro Design Considerations: Dynamic creation, chaining, undo/redo.
Performance and Scalability: Ensuring the system remains efficient as more devices and macros are added.

#### First-Principles Solution
Step 1: Define the Command Interface

We start by defining a Command interface that all commands must implement:

PYTHON
class Command:
    def execute(self):
        raise NotImplementedError

    def undo(self):
        raise NotImplementedError
Step 2: Implement Concrete Commands for Each Device

For each smart home device (e.g., lights, thermostats), we create concrete command implementations. For instance, a LightSwitchOn and ThermostatSetTemp:

PYTHON
class LightSwitchOn(Command):
    def __init__(self, light):
        self.light = light

    def execute(self):
        self.light.turn_on()

    def undo(self):
        self.light.turn_off()

class ThermostatSetTemp(Command):
    def __init__(self, thermostat, temperature):
        self.thermostat = thermostat
        self.temperature = temperature
        self.prev_temp = None

    def execute(self):
        self.prev_temp = self.thermostat.get_current_temperature()
        self.thermostat.set_temperature(self.temperature)

    def undo(self):
        if self.prev_temp is not None:
            self.thermostat.set_temperature(self.prev_temp)
Step 3: Create Macro Command

A macro command can be implemented by chaining multiple commands and executing them sequentially or grouping them into a single command that executes in one go.

PYTHON
class MacroCommand(Command):
    def __init__(self, commands=None):
        self.commands = []
        if commands:
            self.commands.extend(commands)

    def execute(self):
        for command in self.commands:
            command.execute()

    def undo(self):
        for command in reversed(self.commands):
            command.undo()
Step 4: Handle Undo/Redo

To support undo and redo, we maintain a stack of executed commands. Each time a macro is executed, its commands are pushed to the stack.

PYTHON
class CommandHistory:
    def __init__(self):
        self.history = []
        self.current_index = -1

    def execute(self, command):
        if self.current_index < len(self.history) - 1:
            self.history = self.history[:self.current_index + 1]
        self.history.append(command)
        self.current_index += 1
        return True

    def undo(self):
        if self.current_index > 0:
            self.history[self.current_index].undo()
            self.current_index -= 1
        return False

history = CommandHistory()

#### Closing Summary
and Common Mistakes/Follow-Ups

This design allows for dynamic macro creation, where multiple commands can be combined into a single logical unit. The use of Command and MacroCommand interfaces provides flexibility and encapsulation, making it easier to add new devices or commands without altering existing code.

#### Common Mistakes

Lack of Error Handling: Ensure proper error handling for each command.
Memory Leaks: Carefully manage the execution history to avoid memory leaks.
Performance Issues: Consider optimizing macro creation and execution, especially when dealing with a large number of commands or devices.

#### Follow-Ups

How would you handle edge cases like invalid commands?
What about performance optimization for frequent macro operations?

**Code:**

```python
class Command:
    def execute(self):
        raise NotImplementedError

    def undo(self):
        raise NotImplementedError

class LightSwitchOn(Command):
    def __init__(self, light):
        self.light = light

    def execute(self):
        self.light.turn_on()

    def undo(self):
        self.light.turn_off()

class ThermostatSetTemp(Command):
    def __init__(self, thermostat, temperature):
        self.thermostat = thermostat
        self.temperature = temperature
        self.prev_temp = None

    def execute(self):
        self.prev_temp = self.thermostat.get_current_temperature()
        self.thermostat.set_temperature(self.temperature)

    def undo(self):
        if self.prev_temp is not None:
            self.thermostat.set_temperature(self.prev_temp)

class MacroCommand(Command):
    def __init__(self, commands=None):
        self.commands = []
        if commands:
            self.commands.extend(commands)

    def execute(self):
        for command in self.commands:
            command.execute()

    def undo(self):
        for command in reversed(self.commands):
            command.undo()

class CommandHistory:
    def __init__(self):
        self.history = []
        self.current_index = -1

    def execute(self, command):
        if self.current_index < len(self.history) - 1:
            self.history = self.history[:self.current_index + 1]
        self.history.append(command)
        self.current_index += 1
        return True

    def undo(self):
        if self.current_index > 0:
            self.history[self.current_index].undo()
            self.current_index -= 1
        return False

history = CommandHistory()
```

