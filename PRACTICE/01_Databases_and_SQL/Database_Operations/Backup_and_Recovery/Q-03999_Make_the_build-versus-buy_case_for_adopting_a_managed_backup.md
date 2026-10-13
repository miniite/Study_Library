---
type: "practice"
difficulty: "Expert"
---

# Make the build-versus-buy case for adopting a managed backup product like Rubrik or a cloud-native service versus building recovery orchestration in-house across your fleet.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Backup Retention and Restore at Scale](<../../../../01_Databases_and_SQL/Database_Operations/Backup_and_Recovery/Consistent_Backups_and_Restore_Safety/Backup_Retention_and_Restore_at_Scale.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to evaluate whether it makes more sense to build an in-house solution for backup and recovery orchestration or adopt a managed product like Rubrik. As a Database Developer Architect/Principal, my primary concerns are the cost, reliability, scalability, security, and ease of maintenance.

First, let's define what "build" means here. Building our own solution involves developing, testing, deploying, and maintaining an in-house backup system and recovery orchestration logic. On the other hand, adopting a managed product like Rubrik means leveraging a vendor-provided solution that typically includes ongoing support and updates. The key differences lie in the initial investment, operational overhead, and long-term benefits.

#### Triage

The core of this question is to determine if the cost-benefit ratio favors building or buying. We need to consider factors such as:

Initial development and deployment costs
Maintenance and operational costs
Reliability and data security
Scalability and performance
Time-to-market for new features

#### First-Principles Solution
1. Initial Development Costs

Building a solution in-house requires significant upfront investment in terms of developer time, infrastructure setup, and testing. For a database fleet of any size, the initial development could take several months to years depending on complexity.

For example, if we have a fleet of 100 databases with varying backup requirements, developing an orchestration layer that can handle all these might involve:

Writing scripts or custom logic for backups
Implementing logging and monitoring systems
Setting up version control and testing environments
2. Maintenance and Operational Costs

Once the system is built, it needs to be maintained and updated regularly. This includes continuous integration and deployment (CI/CD) practices, regular updates to ensure compliance with security standards, and ongoing support for any issues that arise.

For an in-house solution:

Continuous development efforts: 2+ FTEs dedicated to maintenance
Potential for high overhead if multiple teams are involved

Using a managed product like Rubrik significantly reduces this burden. For instance, Rubrik offers automated backup workflows and a user-friendly interface, which can drastically cut down on the operational time required.

3. Reliability and Data Security

Reliability is crucial for any production environment. A managed solution often provides built-in redundancy and disaster recovery features that are challenging to replicate in-house without significant investment.

Rubrik's features include:

Automatic failover capabilities
Point-in-time recovery
Granular access controls

In contrast, an in-house solution might rely on custom scripts or third-party tools, which could be less reliable and more prone to human error. For example, a single point of failure in the backup process can lead to data loss.

4. Scalability and Performance

Scalability is another critical factor. As your database fleet grows, managing backups and recovery becomes increasingly complex. A managed product like Rubrik scales easily with minimal effort on your part.

Rubrik's architecture supports:

Automated scaling of backup resources based on demand
Integration with cloud services for seamless migration

An in-house solution might require custom scripts or third-party tools that need to be constantly updated and optimized, which can be resource-intensive.

5. Time-to-Market for New Features

In a rapidly evolving environment, the ability to quickly adopt new features is crucial. Managed products like Rubrik often offer regular updates and new features through their service agreements.

For example, Rubrik regularly releases new versions with enhanced backup technologies such as AI-driven data protection. In contrast, an in-house solution might lag behind due to limited development resources.

#### Closing Summary

In summary, the build-versus-buy decision depends on a thorough cost-benefit analysis. While building a custom solution provides full control and flexibility, it comes at a significant initial and ongoing cost. On the other hand, adopting a managed product like Rubrik offers reliability, scalability, ease of maintenance, and time-to-market advantages.

#### Common Mistakes/Follow-Ups
Underestimating Maintenance Costs: Often, the long-term operational costs are overlooked when comparing initial development efforts.
Ignoring Vendor Lock-In Risks: While a managed product might seem expensive upfront, switching vendors can be complex and costly.
Not Considering Regulatory Compliance: Managed products often include built-in compliance features that can save time and resources.

Given these factors, the in-house solution is generally less cost-effective unless there are specific, compelling reasons to develop an internal solution (such as very tight security requirements or unique business processes).

**Comparison:**

```
Cost Component	In-House Solution	Managed Product (Rubrik)
Initial Dev. & Deployment	High ($Xk)	Lower to Moderate ($Yk)
Maintenance	Ongoing ($Zk/year)	Minimal ($Wk/year)
Reliability	Custom scripts, high risk	Built-in redundancy, low risk
Scalability	Manual scaling, complex	Automated, easy integration
```

**Code:**

```python
# Example of a simple backup script in Python for comparison
import os
import shutil

def create_backup(source_dir, target_path):
    if not os.path.exists(target_path):
        os.makedirs(target_path)

    for root, dirs, files in os.walk(source_dir):
        for file in files:
            src = os.path.join(root, file)
            dest = os.path.join(target_path, os.path.relpath(src, source_dir))
            shutil.copy2(src, dest)

create_backup("/path/to/databases", "/path/to/backups")
```

**Example input / output:**

```text
Input (database fleet): 30 databases, varying sizes and backup frequencies.
Output (backup plan):
- Automated backups for each database at specified intervals
- Scheduled point-in-time recovery for last 7 days
```

**Data:**

```yaml
# Example database configuration
databases:
  - name: db1
    path: /path/to/db1
    backup_frequency: daily
  - name: db2
    path: /path/to/db2
    backup_frequency: hourly
```

**Flow:**

1. Initial Dev & Deployment Cost
2. In-House Solution
3. Managed Product (Rubrik)
4. Ongoing Maintenance Costs
5. Minimal Ongoing Costs
6. High Risk Reliability
7. Low Risk, Built-in Features
8. Build vs Buy Decision
9. Managed Product (Rubrik)

