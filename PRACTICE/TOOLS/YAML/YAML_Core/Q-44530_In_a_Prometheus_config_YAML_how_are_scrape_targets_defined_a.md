---
type: "practice"
difficulty: "Intermediate"
---

# In a Prometheus config YAML how are scrape targets defined as a list under a job?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Refactoring Helm Values](<../../../../TOOLS/YAML/YAML_Core/Refactoring_Helm_Values.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To define scrape targets in a Prometheus configuration using YAML, you need to understand that Prometheus scrapes metrics from various sources or "jobs". A job defines both the target to be scraped and the labels associated with it. In an intermediate context, let's break down how this is done.

First, we'll start by summarizing the key points:

Scrape targets are defined under a job in the Prometheus configuration.
Each job entry lists its scrape targets as a list of URL-like strings or service discovery configurations.
Additional metadata like labels can be attached to these targets for better organization and filtering.

Now, let's dive into the details.

#### Triage

The question is asking about defining scrape targets in Prometheus YAML configuration. Prompts are seeking an understanding of how to structure the YAML file to include job definitions with their respective target URLs or service discovery configurations. This involves understanding the syntax and best practices for defining jobs and targets.

#### First-Principles Solution

Prometheus scrapes metrics from multiple sources, each defined as a job in its configuration. Each job can have one or more scrape targets listed under it. The basic structure of a job definition might look like this:

YAML
scrape_configs:
  - job_name: 'example_job'
    static_configs:
      - targets: ['localhost:9100', 'server2.example.com:4242']

Here, job_name is an identifier for the job, and static_configs lists the scrape targets as a list of strings representing host:port pairs or service discovery configurations.

#### Common Mistakes/Follow-Ups

#### Common mistakes
include:

Not defining a unique job_name.
Misplacing target definitions under another section of the configuration.
Using incorrect syntax or unsupported target formats.

#### Follow-ups
might include discussing advanced topics like using external labels, configuring relabeling rules, and integrating with service discovery plugins.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, defining scrape targets in Prometheus involves specifying them within a job under the scrape_configs section. Each job can have multiple targets listed as strings or through more dynamic means like service discovery. Ensuring proper syntax and understanding the broader context of configuration management are crucial.

**Code:**

```yaml
# Example Prometheus YAML configuration snippet
scrape_configs:
  - job_name: 'example_job'
    static_configs:
      - targets: ['localhost:9100', 'server2.example.com:4242']
```

