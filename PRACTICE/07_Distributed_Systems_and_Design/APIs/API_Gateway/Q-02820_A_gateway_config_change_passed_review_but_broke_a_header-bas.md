---
type: "practice"
difficulty: "Deep"
---

# A gateway config change passed review but broke a header-based route because staging strips a header a middlebox adds in prod; how do you build config testing with production-representative traffic replay to catch env-specific breakage?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Gateway Config Testing and Rollout](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Config_Testing_and_Rollout.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

To address the issue of header-based routing breakages between environments, we need to implement a robust configuration testing framework that can simulate production-like conditions in staging. This involves capturing and replaying real production traffic, which includes headers that are stripped or added at different stages. By doing so, we ensure that our gateway configurations remain compatible across all environments.

#### Triaging the Question

The core of this question revolves around ensuring that API Gateway configurations remain functional when moving from one environment (e.g., staging) to another (e.g., production). The specific challenge is that a header-based route in staging fails due to headers being stripped out by a middlebox, which adds them back in prod. This issue highlights the need for a comprehensive testing strategy that accounts for these environment-specific behaviors.

#### Assumptions
We have access to both staging and production environments.
Production traffic can be recorded and replayed in a controlled manner.
The gateway configuration changes are part of a CI/CD pipeline with automated tests.
Headers stripped or added by the middlebox are significant for routing decisions.

#### First-Principles Solution

Traffic Capture: Use tools like tcpdump to capture real production traffic that includes all necessary headers. This step ensures we have a representative sample of how requests behave in production.

Proxying and Replay: Set up a proxy server (e.g., NGINX) that can replay this captured traffic in staging, including the stripped and added headers. This setup will allow us to see if our gateway configurations behave as expected in an environment that closely mimics production.

Configuration Testing Framework: Develop a testing framework that automates the process of comparing captured production traffic with staged traffic. This involves:

Setting up test cases based on header-based routing rules.
Replaying each request from the capture and verifying its handling by the gateway in staging.
Using metrics like response time, error rates, and specific headers to validate behavior.

Automation and Integration: Integrate this testing framework into our CI/CD pipeline so that every configuration change is automatically tested against captured production traffic before being deployed to staging or production.

#### Implementation
Steps:

Setup Traffic Capture:

BASH
tcpdump -i eth0 -w /path/to/capture.pcap "port 80"

This command captures all HTTP traffic on the eth0 interface and saves it to a file named capture.pcap.

Install NGINX as a Proxy:

BASH
sudo apt-get update
sudo apt-get install nginx

Configure NGINX to Replay Traffic: Create an NGINX configuration file (e.g., /etc/nginx/conf.d/replay.conf) with the following content:

NGINX
http {
    include /etc/nginx/mime.types;
    server {
        listen 8080;

        location / {
            proxy_pass http://localhost:80;
            break_on_error on;
            error_log stderr debug;
        }
    }
}

Verify Configuration and Start NGINX:

BASH
sudo systemctl restart nginx

Ensure that the NGINX server is running correctly by accessing it via http://localhost:8080.

Develop Testing Framework: Write Python scripts using libraries like scapy to parse captured traffic and replay requests through our gateway setup.

Integrate with CI/CD Pipeline: Use tools like Jenkins, GitLab CI, or GitHub Actions to automate the testing process.

#### Closing Summary
+ Common Mistakes/Follow-Ups:
This approach ensures that we catch any environment-specific breakages early in the development cycle.

#### Common mistakes
include not properly configuring traffic capture and replay, failing to account for all headers involved, and not integrating this into a CI/CD pipeline effectively.

#### Follow-ups
might involve optimizing the testing framework for performance or expanding it to cover other types of traffic patterns.

**Code:**

```bash
# Example Bash command for tcpdump
tcpdump -i eth0 -w /path/to/capture.pcap "port 80"
```

**Flow:**

1. Capture Production Traffic
2. Check for Specific Headers
3. Replay Traffic in Staging
4. Test Gateway Configurations
5. Automate Testing Framework
6. Integrate with CI/CD Pipeline

