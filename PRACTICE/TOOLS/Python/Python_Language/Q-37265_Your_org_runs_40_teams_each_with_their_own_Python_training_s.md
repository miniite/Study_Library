---
type: "practice"
difficulty: "Expert"
---

# Your org runs 40 teams each with their own Python training stack and framework versions. How would you drive convergence to a golden Python runtime image without a big-bang mandate that stalls every team's roadmap?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Admission Control and Load Shedding](<../../../../TOOLS/Python/Python_Language/Python_Tail_Latency/Admission_Control_and_Load_Shedding.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of driving convergence to a golden Python runtime image across 40 teams, we need to balance technical and organizational constraints while ensuring minimal disruption to ongoing projects. We will adopt an iterative approach, leveraging version control systems and virtualization to incrementally unify the Python stack.

Firstly, the core idea is to use a combination of containerization (Docker) and a configuration management tool (like Ansible or Kubernetes) to standardize the environment without imposing a top-down mandate. This way, we can gradually phase in the new runtime image while maintaining operational flexibility for each team.

**Comparison:**

```
Team Request	Python Version	Docker Image	Virtual Environment
Team A	3.7	python:3.7	customenv
Team B	3.7	python:3.7	myenv
```

**Code:**

```
Example Dockerfile snippet for Python 3.8

FROM python:3.8-slim

RUN pip install --no-cache-dir numpy pandas requests

CMD ["python", "app.py"]

CODE
Copy

```bash
# Ansible playbook to manage deployment
- name: Install Python dependencies
  apt:
    name: "{{ packages }}"
    update_cache: yes
  vars:
    packages:
      - python3.8
      - python3-pip
```

**Example input / output:**

```text
Input (Team Request):
- Team A asks for a Python 3.7 Docker image with specific dependencies.
- Team B requests the same but wants to use a different virtual environment.

Output (Ansible Playbook Response):
- Ansible deploys both images using separate containers, ensuring both teams can start working with their requested environments.
```

**Flow:**

1. Receive Team Requests
2. Analyze & Categorize
3. Generate Docker Configs
4. Deploy Images Using Ansible
5. Monitor for Issues
6. Clean Up & Optimize

