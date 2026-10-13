---
type: "practice"
difficulty: "Deep"
---

# An interviewer probe: you said you'd standardize on one orchestration tool — what happens to the 200 legacy cron jobs and the team that refuses to migrate? How does the strategy survive contact with reality?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Late and Out-of-order Data Strategy](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Data_Platform_Strategy/Late_and_Out-of-order_Data_Strategy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a data platform strategy context, standardizing on one orchestration tool is a critical decision aimed at increasing efficiency, reducing maintenance overhead, and ensuring consistency across our operations. However, the reality of transitioning from 200 legacy cron jobs to this new orchestration tool, especially when faced with resistance from a team that prefers not to migrate, can pose significant challenges. This situation requires a thoughtful approach to ensure the strategy's long-term success.

Firstly, let’s define the key aspects we are dealing with:

Legacy Cron Jobs: These are existing workflows and processes that currently rely on cron jobs for scheduling.
Resistant Team: Members who may be comfortable with their current tools and resist change due to various reasons such as comfort, fear of disruption, or perceived inefficiencies in the new tool.

To address these challenges, we need to adopt a strategy that respects both the legacy systems and the team’s concerns while paving a path towards eventual standardization. Here are the steps I would take:

Assess the Current State: Before making any changes, it's crucial to understand the current state of the 200 cron jobs. This includes identifying which jobs are critical for business operations, their frequency, and dependencies.

Identify Migration Priorities: Not all cron jobs will need immediate migration. We should prioritize those that have a direct impact on core business functions or those that can be easily converted without significant disruptions.

Pilot Program: Launch a pilot program to transition a subset of the cron jobs to the new orchestration tool. This helps in validating the benefits and addressing any unforeseen issues early on.

Training and Support: Provide comprehensive training and support for the team that will be using the new tool. This can include workshops, documentation, and ongoing assistance to ease their transition.

Incentivize Migration: Offer incentives such as improved job scheduling capabilities, better monitoring, or reduced maintenance overhead as benefits of migrating. Highlight how standardization can improve overall system reliability and efficiency.

Communication and Change Management: Engage in transparent communication with the team about why this change is necessary and how it will benefit everyone. Address any concerns they may have and involve them in the decision-making process to build buy-in.

Failing Fast Gracefully: If a job cannot be migrated due to technical constraints, we should establish a clear plan for handling these cases. This could include creating custom scripts or plugins within the new tool to support non-migratable jobs where necessary.

Continuous Improvement: After initial migration phases, continuously review and refine the process based on feedback from the team and actual usage patterns of the new tool.

By taking this multi-faceted approach, we can ensure that our strategy for standardizing on one orchestration tool not only survives contact with reality but also thrives in a dynamic and resistant environment. The key is to balance change management with operational continuity and foster a culture of collaboration and improvement.

**Comparison:**

| Cron Job ID | Name                | Frequency   | Status Before Migration | Status After Migration |
|-------------|---------------------|-------------|------------------------|-----------------------|
| 1           | BackupDatabase      | daily       | cron                   | migrated              |
| 2           | GenerateMonthlyReport | monthly    | cron                   | plugin-created        |

**Code:**

```python
# Example of a simple migration script (pseudo-code)
def migrate_cron_job(cron_job):
    if job_can_be_migrated(cron_job):  # Check if the job can be converted to new orchestration tool
        convert_to_new_tool(cron_job)
        log_migration_success(cron_job)
    else:
        create_custom_plugin(cron_job)  # Create a custom plugin for non-migratable jobs

def job_can_be_migrated(cron_job):
    # Check conditions and return True or False
    pass

def convert_to_new_tool(cron_job):
    # Convert the cron job to new tool format
    pass

def log_migration_success(cron_job):
    # Log successful migration
    pass

def create_custom_plugin(cron_job):
    # Create a custom plugin for non-migratable jobs
    pass
```

**Example input / output:**

| Cron Job ID | Status Before Migration | Status After Migration |
|-------------|------------------------|-----------------------|
| 1           | cron                  | migrated              |
| 2           | cron                  | plugin-created        |

**Data:**

```json
{
  "cron_jobs": [
    {
      "id": 1,
      "name": "BackupDatabase",
      "frequency": "daily",
      "status": "migrated"
    },
    {
      "id": 2,
      "name": "GenerateMonthlyReport",
      "frequency": "monthly",
      "status": "plugin-created"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Can Job Be Migrated?
5. Convert to New Tool
6. Migrate Success Log
7. Create Custom Plugin
8. Migrate Job Using Plugin
9. End Migration Process

