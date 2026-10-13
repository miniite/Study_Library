---
type: "practice"
difficulty: "Intermediate"
---

# How would you find the top 5 IP addresses making the most requests in an access log using Python?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Log Filtering Scripts](<../../../../TOOLS/Python/Python_Language/Log_Filtering_Scripts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To find the top 5 IP addresses making the most requests in an access log using Python, we need to parse the log file, count the occurrences of each IP address, and then sort them by frequency to get the top 5. Let's break this down step-by-step.

First, we assume that our input is a standard HTTP access log with timestamps, request methods, paths, and IP addresses. A common format for such logs is:

CODE
<timestamp> <IP-address> "<request>" <status-code> <response-size>

The question is essentially asking us to process this log file and extract the most frequently occurring IP addresses. The core idea here involves reading the log line by line, parsing each line to get the IP address, storing it in a data structure (like a dictionary), incrementing counts for each IP, and finally sorting these counts to find the top 5.

