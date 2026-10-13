---
type: "practice"
difficulty: "Deep"
---

# An Ansible variable defined in group_vars YAML is being overridden unexpectedly. Explain the YAML file precedence in Ansible's variable hierarchy and how you would prove which file won.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Refactoring Helm Values](<../../../../TOOLS/YAML/YAML_Core/Refactoring_Helm_Values.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with Ansible, understanding the variable hierarchy can be crucial to avoid unexpected behavior, such as a variable being overridden by another one. In Ansible, variables are stored in multiple locations and have different precedence levels. Let's break down how YAML files fit into this hierarchy and discuss how you can diagnose which file is causing an override.

TL;DR

Ansible uses a multi-layered approach for variable resolution, with the following order of precedence:

Environment Variables: ANSIBLE_*
Playbook vars/set_fact: Defined within playbooks or roles.
Inventory Group and Host Variables: group_vars and host_vars.
Role Default Variables: defaults/main.yml.
Role Variable Files: Custom files in the role directory like main.yml, extra.yml.

To determine which file is overriding a variable, you can use ansible-inventory --list or debug modules to inspect variables at different stages of execution.

#### Triaging the Question

The question revolves around understanding Ansible's variable precedence and how to identify which YAML file might be causing an unexpected override. The key points are:

Understanding the role of various YAML files in the variable hierarchy.
Identifying where the issue might be occurring.
Using tools or techniques to pinpoint the conflicting definition.

#### First-Principles Solution
Step 1: Understand Variable Precedence

Ansible follows a specific order for resolving variables:

Environment Variables: Ansible allows you to override variables via environment variables, which are checked first.
Playbook vars/set_fact: Variables defined within playbooks or roles take precedence over external definitions.
Inventory Group and Host Variables:
group_vars/<inventory_group>: Variables for a specific group of hosts.
host_vars/<hostname>: Variables for individual hosts.
Role Default Variables (defaults/main.yml): These are the default values for variables within roles.
Role Variable Files: Custom files like main.yml, extra.yml, etc., in role directories.
Step 2: Debugging Override Issues

To identify which file is causing an override, you can:

Use ansible-inventory --list: This command provides a JSON representation of the inventory and variables defined by various sources.
Run Playbooks with Debug Statements: Use the debug module to print variable values at different stages of execution.
Check Role Dependencies: Ensure that role dependencies are correctly managed, as this can affect how variables are resolved.
Step 3: Verifying with Example Code

Here's an example playbook and related files:

YAML
# main.yml in the role directory
---
my_variable: default_value

# group_vars/all
---
my_variable: group_default

# host_vars/example_host
---
my_variable: host_specific

# Playbook
---
- name: Debug variable values
  hosts: all
  gather_facts: no
  tasks:
    - debug:
        msg: "Value of my_variable is {{ lookup('env', 'ANSIBLE_VAR_MY_VARIABLE') }}"
      when: ansible_env.ANSIBLE_VAR_MY_VARIABLE is defined

    - name: Show playbook variables
      debug:
        var: ansible_play_hosts_all | json_query('[*].my_variable')

    - name: Show inventory variables
      debug:
        msg: "Value of my_variable from group_vars: {{ lookup('file', 'group_vars/all') }}"
Step 4: Analyzing the Output

Run the playbook and observe the output. The debug module will show you which value is being used at each stage:

YAML
# Sample Debug Output
"msg": "Value of my_variable from group_vars: {'my_variable': 'group_default'}"
"ansible_play_hosts_all": [
  {
    "my_variable": "host_specific"
  },
  ...
]

From this, you can see that the my_variable in the host-specific file (host_vars/example_host) takes precedence over the group variable.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Understanding Ansible's variable hierarchy is crucial to avoid unexpected behavior. Always check the order of precedence and use tools like ansible-inventory --list or debug statements to pinpoint where variables are being defined or overridden.

#### Common mistakes
include:

Overlooking environment variables.
Not checking host-specific files before group variables.
Ignoring role dependencies that might shadow default values.

#### Follow-up
questions could involve scaling this solution to multiple environments or roles, managing large inventories, and integrating variable management with version control systems like Git.

**Comparison:**

```
Variable Source	Order of Precedence
Environment Variables	1
Playbook vars/set_fact	2
Inventory Group and Host Variables	3
Role Default Variables	4
Role Variable Files	5
```

**Code:**

```yaml
# main.yml in the role directory
---
my_variable: default_value

# group_vars/all
---
my_variable: group_default

# host_vars/example_host
---
my_variable: host_specific

# Playbook
---
- name: Debug variable values
  hosts: all
  gather_facts: no
  tasks:
    - debug:
        msg: "Value of my_variable is {{ lookup('env', 'ANSIBLE_VAR_MY_VARIABLE') }}"
      when: ansible_env.ANSIBLE_VAR_MY_VARIABLE is defined

    - name: Show playbook variables
      debug:
        var: ansible_play_hosts_all | json_query('[*].my_variable')

    - name: Show inventory variables
      debug:
        msg: "Value of my_variable from group_vars: {{ lookup('file', 'group_vars/all') }}"
```

**Example input / output:**

```text
- Command Run:
  ansible-inventory --list

- Output:
  {
    "_meta": {
      "hostvars": {
        "example_host": {
          "my_variable": "group_default"
        }
      }
    },
    "all": {},
    ...
  }

- Playbook Debug Statements:
  - "Value of my_variable is group_default"
  - "Value of my_variable from group_vars: {'my_variable': 'group_default'}"
  - "Value of my_variable in playbook variables: host_specific"
```

**Data:**

```json
{
  "ansible_play_hosts_all": [
    {
      "my_variable": "host_specific"
    }
  ]
}
```

**Flow:**

1. EnvVar
2. Override
3. PlaybookVars
4. InventoryGroup
5. InventoryHost
6. RoleDefault
7. RoleVariables

