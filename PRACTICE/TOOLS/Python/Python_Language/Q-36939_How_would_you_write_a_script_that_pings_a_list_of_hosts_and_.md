---
type: "practice"
difficulty: "Intermediate"
---

# How would you write a script that pings a list of hosts and reports which ones are unreachable?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Comparing Time Windows](<../../../../TOOLS/Python/Python_Language/Comparing_Time_Windows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle this problem, we need to create a Python script that can ping multiple hostnames or IP addresses from a list and report which ones are unreachable (i.e., not responding). The script will use the subprocess module to execute the ping command for each hostname/IP address. We'll store the results in a dictionary and then print out the unreachable hosts at the end.

#### Triage

The core of this question is asking us to write a Python script that performs pinging operations on multiple targets, captures the results, and categorizes them as reachable or unreachable. The interviewer might be probing your understanding of:

Basic shell commands in Python.
Handling lists and dictionaries.
Exception handling.
Output formatting.
Step-by-Step Solution
Define a list of hosts to ping.
Use the subprocess module to execute the ping command for each host.
Capture the output from the ping command and determine if the host is reachable or not.
Store the results in a dictionary with keys as the hosts and values indicating their reachability.
Print out the unreachable hosts.

**Code:**

```python
import subprocess

# Define a list of hosts to ping
hosts = ["192.0.2.1", "8.8.8.8", "10.0.0.1"]  # Example IPs

# Function to ping a host and determine its reachability
def ping_host(host):
    try:
        result = subprocess.run(["ping", "-c", "4", host], stdout=subprocess.PIPE, stderr=subprocess.PIPE)
        if result.returncode == 0:
            return True
        else:
            return False
    except Exception as e:
        print(f"Error pinging {host}: {e}")
        return False

# Dictionary to store the results
ping_results = {}

# Ping each host and record its reachability
for host in hosts:
    is_reachable = ping_host(host)
    ping_results[host] = "Reachable" if is_reachable else "Unreachable"

# Print out unreachable hosts
unreachable_hosts = [host for host, status in ping_results.items() if status == "Unreachable"]
print("Unreachable Hosts:")
for host in unreachable_hosts:
    print(host)
Closing Summary and Common Mistakes/Follow-Ups

The script successfully pings a list of specified hosts and reports which ones are unreachable. Common mistakes to watch out for include not handling exceptions properly, misinterpreting the ping command output, or forgetting to store and retrieve the results.

If the interviewer asks follow-up questions, you might be asked about:

Optimizing the script for better performance.
Handling different types of network errors.
Logging the output to a file instead of printing it to the console.
Scaling the solution if dealing with many hosts.
```

