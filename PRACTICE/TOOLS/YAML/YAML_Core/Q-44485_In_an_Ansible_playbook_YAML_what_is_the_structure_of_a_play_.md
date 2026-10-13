---
type: "practice"
difficulty: "Intermediate"
---

# In an Ansible playbook YAML what is the structure of a play and what does the 'tasks' key contain?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ YAML and JSON](<../../../../TOOLS/YAML/YAML_Core/YAML_and_JSON.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Ansible playbooks, a "play" defines a set of tasks to be executed on one or more hosts. The tasks key contains a list of instructions that are run sequentially within a play.

TL;DR: A YAML-based Ansible playbook consists of plays, each containing multiple tasks that define what actions should be taken.

#### Triaging the Question

This question is asking you to explain the structure of an Ansible playbook and specifically delve into what the tasks key does. The interviewer might be probing your understanding of basic Ansible concepts like playbooks and how they are structured in YAML. You will need to assume that a basic familiarity with Ansible is given, but you should still provide foundational context.

#### First-Principles Solution

Play Structure:

A playbook consists of one or more plays.
Each play is defined as a dictionary with specific keys and values.

Key Components of a Play:

name: A human-readable name for the play (optional but recommended).
hosts: The hosts on which this play should be executed.
tasks: A list of tasks to be performed in sequence within the play.

Tasks Key Content:

The tasks key contains a list of task dictionaries, each representing an individual action or command that Ansible will execute.
Each task dictionary can have keys like name, name, hosts, and various modules (like apt, service, etc.).

#### Example
Play in YAML
YAML
---
- name: Install and start Apache2
  hosts: webservers
  tasks:
    - name: Ensure Apache2 package is installed
      apt:
        name: apache2
        state: present

    - name: Start Apache2 service
      service:
        name: apache2
        state: started
        enabled: yes

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, each play in an Ansible playbook is a set of tasks executed on specified hosts. The tasks key lists these actions, ensuring the playbook runs as intended.

#### Common Mistakes
to Avoid:

Not specifying the correct modules or parameters.
Misunderstanding the scope (hosts) for which a task should be run.
Overlooking required state management in tasks like starting services or managing packages.
Further Discussion

The interviewer might ask you about conditional statements, loops, and other control flow constructs that can be used within tasks. You could also discuss best practices such as using roles to organize complex playbooks into reusable components.

**Code:**

```yaml
---
- name: Install and start Apache2
  hosts: webservers
  tasks:
    - name: Ensure Apache2 package is installed
      apt:
        name: apache2
        state: present

    - name: Start Apache2 service
      service:
        name: apache2
        state: started
        enabled: yes
```

