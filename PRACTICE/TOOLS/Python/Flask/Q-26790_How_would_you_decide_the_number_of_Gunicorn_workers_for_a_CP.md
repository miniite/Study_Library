---
type: "practice"
difficulty: "Deep"
---

# How would you decide the number of Gunicorn workers for a CPU-bound ML inference Flask service?

**Difficulty:** Deep · **Levels:** Fresher · [↑ Long-running Work in Flask](<../../../../TOOLS/Python/Flask/Long-running_Work_in_Flask.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To determine the optimal number of Gunicorn workers for a CPU-bound ML inference Flask service, we need to balance the trade-offs between CPU utilization and the overhead introduced by context switching and inter-process communication (IPC). This decision involves understanding the specifics of your workload and the hardware resources available. Let's break it down step-by-step.

First, we need to understand that Gunicorn is a Python WSGI HTTP Server for UNIX, often used as a WSGI application server. Each worker in Gunicorn handles incoming requests by executing the Flask application code, which in this case processes CPU-bound ML inference tasks.

#### Triage
and Assumptions

#### Assumptions

The service is CPU-bound: This means the primary bottleneck will be the processing power required to run your ML models.
We are using Gunicorn as a worker process manager with Flask serving the application.
The machine has multiple CPUs or cores available.

What We Are Probing:

Understanding how Gunicorn workers operate and interact with the CPU.
Evaluating the trade-offs between more vs fewer workers in terms of performance and resource utilization.

#### First-Principles Solution

Identify Your Application’s Load: Start by profiling your application to understand its baseline behavior under varying loads. Use tools like top, htop, or monitoring frameworks like Prometheus to observe the CPU usage and response times with different worker counts.

Understand Gunicorn’s Worker Model:

Multiple Workers: When you start multiple workers, each worker processes incoming requests in parallel. This can lead to better load distribution but increases overhead due to context switching and IPC between workers.
Worker Types:
sync (default): Each request is processed by a separate thread on the same process.
gevent: Uses Greenlets to handle asynchronous I/O, which might be useful if you have IO-bound tasks but not for CPU-bound tasks.

Determine the Ideal Worker Count:

A general rule of thumb is to set the number of workers equal to the number of available CPUs or cores.
However, this needs to be validated with profiling data. For instance, if running 4 workers on a 4-core CPU results in higher context switching overhead and does not improve performance, you might opt for fewer workers.

#### Cons
ider Hardware and Network Latency:

If the network latency is high or there are significant I/O operations, fewer workers may be better.
For pure CPU-bound tasks, more workers can utilize all available cores effectively.

Test with Different Configurations:

Run your service with different numbers of workers (e.g., 2, 4, 8) and observe the performance metrics.
Choose the configuration that provides the best balance between throughput and response time without increasing unnecessary overhead.

#### Closing Summary
+ Common Mistakes
Common Mistake: Always setting the number of workers to a high value like n, assuming more is always better. In reality, this can lead to higher context switching overhead.
Best Practice: Start with a conservative approach (e.g., equal to or slightly less than the number of cores) and fine-tune based on profiling data.

#### Example
Code Snippet
PYTHON
# Gunicorn configuration snippet
bind = "0.0.0.0:8000"
workers = 4  # Number of workers, set to match available CPU cores

# Additional configurations
threads = 2  # Number of threads per worker (if using sync workers)
worker_class = 'sync'  # Use gevent for async tasks if applicable
Self-Check

This solution covers all key aspects and is comprehensive.

