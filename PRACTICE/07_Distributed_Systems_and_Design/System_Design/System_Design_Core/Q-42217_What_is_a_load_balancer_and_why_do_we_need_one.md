---
type: "practice"
difficulty: "Basic"
---

# What is a load balancer and why do we need one?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Load Balancers](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Load_Balancers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage
— what is really being asked

This is a basic-concept question, so the interviewer wants a clear, correct mental model — not a deep dive into consistent hashing. They're checking that you know what a load balancer is, where it sits, and why it's one of the first building blocks of any scalable web system. A great fresher/early answer defines it plainly, gives the reasons we need it (scale, availability, health), and names one or two concrete examples. Let me build it up from first principles.

The problem a load balancer solves

Imagine your app runs on one server. Two problems appear immediately:

It can't handle the traffic. One machine has a finite amount of CPU, memory, and network bandwidth. When a million users show up, that single server slows to a crawl or crashes.
It's a single point of failure. If that one server goes down (crash, deploy, hardware fault), your entire site is down. There is no backup.

The natural fix is to run many identical copies of your app on many servers (this is called horizontal scaling). But now a new question appears: when a user's request comes in, which server should handle it? The user only knows one address (www.example.com); they shouldn't have to know or care about individual servers.

That "traffic cop" that sits in front of your pool of servers and decides where each request goes is the load balancer.

Definition

A load balancer is a component (hardware or software) that sits between clients and a group of backend servers, and distributes incoming requests across those servers so no single server is overwhelmed.

The client sends its request to the load balancer's address. The load balancer picks a healthy backend server, forwards the request, gets the response, and returns it to the client. To the user it looks like one server; behind the scenes there might be dozens or hundreds.

Why we need one — the concrete reasons

1. Distribute load (scalability). It spreads requests evenly across servers, so each does a fair share. Add more servers as traffic grows, and the load balancer starts using them — you scale horizontally without changing the client.

2. High availability / fault tolerance. The load balancer continuously runs health checks (e.g., pinging /health every few seconds). If a server stops responding, the load balancer stops sending it traffic and reroutes to the healthy ones. Users never notice one server died — this removes the single-point-of-failure problem.

3. Zero-downtime deploys & maintenance. You can take one server out of rotation, update it, put it back, and repeat — the site stays up the whole time (rolling deploys) because the load balancer routes around the server being worked on.

4. Better performance. By keeping any one server from being overloaded, response times stay low. Some load balancers also do useful extra work: SSL/TLS termination (handling HTTPS encryption so backends don't have to), caching, and session stickiness (keeping a user tied to the same server when needed).

5. Security & a clean front door. It hides backend servers behind a single public entry point, and can help absorb or filter bad traffic (e.g., during a DDoS).

How it works, simply
CODE
                 ┌── Server A
Users → [ Load Balancer ] ── Server B
                 └── Server C

The load balancer needs a rule to pick a server — a load-balancing algorithm. The simplest is round robin: server A, then B, then C, then A again, cycling through. Another is least connections: send the next request to whichever server currently has the fewest active connections. (Choosing among these is a natural follow-up question.)

Types you should be able to name
Layer 4 (transport) load balancer: routes based on IP address and port only — fast, doesn't look at the actual content. Example: AWS Network Load Balancer.
Layer 7 (application) load balancer: understands HTTP — can route by URL path, hostname, or headers (e.g., send /api/* to one pool, /images/* to another). Example: AWS Application Load Balancer, Nginx, HAProxy.

Real-world examples: Nginx and HAProxy (software), AWS ELB/ALB, Google Cloud Load Balancer, and hardware appliances like F5. Companies like Netflix, Amazon, and Flipkart put load balancers in front of essentially every service.

Summary — what a strong answer emphasizes
A load balancer is the traffic distributor sitting between clients and a pool of backend servers.
We need it for three big reasons: spread load (scale horizontally), stay available (health checks route around failed servers, no single point of failure), and deploy/maintain without downtime.
Bonus value: SSL termination, caching, sticky sessions, security, a single clean entry point.
Know that it uses an algorithm (round robin, least connections) to pick a server, and that L4 vs L7 is the type distinction.

#### Common mistakes
confusing a load balancer with an API gateway or reverse proxy (they overlap but a load balancer's core job is distribution + health checking); forgetting to mention health checks, which are what actually deliver the availability benefit; or claiming it stores data (it doesn't — it routes).

Likely follow-ups: "Which algorithms?" (round robin, least connections, IP hash), "What happens to user sessions if requests bounce between servers?" (sticky sessions or externalize session state to Redis), and "Isn't the load balancer itself a single point of failure?" (yes — so you run it in a redundant/HA pair or use a managed, auto-replicated one like AWS ALB).

**Comparison:**

```
Aspect	Without a load balancer (1 server)	With a load balancer (many servers)
Handle high traffic	No — one machine caps out	Yes — spread across the pool, add servers
Server failure	Whole site is down	Reroute to healthy servers, users unaffected
Deploys	Downtime while updating	Rolling, zero-downtime
Entry point	Exposes the one server	Single clean public address, backends hidden
```

**Code:**

```
NGINX
Copy
# Minimal Nginx load balancer: distribute traffic across 3 app servers
http {
  upstream app_servers {
    # default algorithm is round robin
    server 10.0.0.11:8080;
    server 10.0.0.12:8080;
    server 10.0.0.13:8080;
    # least_conn;   # uncomment to use least-connections instead
  }

  server {
    listen 80;
    location / {
      proxy_pass http://app_servers;   # forward to the pool
      proxy_set_header Host $host;
    }
  }
}
```

**Example input / output:**

```text
Round-robin over 3 servers (A, B, C):

  Request 1  -> Server A
  Request 2  -> Server B
  Request 3  -> Server C
  Request 4  -> Server A   (cycles back)
  Request 5  -> Server B

Health check makes B unhealthy (stops responding):

  Request 6  -> Server A
  Request 7  -> Server C   (B skipped automatically — users unaffected)
```

**Flow:**

1. skipped
2. Clients
3. Load Balancerhealth checks + algorithm
4. Server A - healthy
5. Server B - healthy
6. Server C - unhealthy

