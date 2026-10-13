---
type: "practice"
difficulty: "Expert"
---

# You're standardizing isolation policy across 400 microservices on a shared Aurora fleet. How do you set a defensible default and let teams opt up or down without silent guarantee drift?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Isolation for Shared Subsystems](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Setting_Isolation_Levels/Isolation_for_Shared_Subsystems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we need to first understand the isolation levels (Read Uncommitted - RU, Read Committed - RC, Repeatable Read - RR, Serializable) and their implications on transactional integrity. The goal is to set a default policy that minimizes the risk of silent guarantee drift while allowing teams flexibility. We'll start by defining our understanding of these isolation levels, then design a solution that balances security and operational efficiency.

First-principles reasoning reveals that RU allows dirty reads, RC prevents dirty reads but not non-repeatable reads or phantom reads, RR ensures no non-repeatable reads or phantom reads but may lock more rows than necessary, and Serializable provides the highest level of consistency at the cost of performance due to excessive locking. Given this context, we can set a default policy of Read Committed (RC) while providing mechanisms for teams to opt up or down as needed.

To achieve this:

Define Default Isolation Level: Set RC as the default isolation level. This balances between minimizing read anomalies and ensuring reasonable performance.
Opt-In/Opt-Out Mechanism: Provide a mechanism for services to explicitly choose different levels if necessary, with clear documentation on the trade-offs.
Audit Trails and Monitoring: Implement robust auditing and monitoring to detect any drift from the default setting without explicit permission.

Let’s break down each step in detail:

Step 1: Define Default Isolation Level

Setting RC as the default ensures that services generally read a consistent snapshot of data, preventing dirty reads but allowing for some level of non-repeatable reads or phantom reads. This choice strikes a balance between performance and consistency.

Step 2: Opt-In/Opt-Out Mechanism

For teams requiring higher isolation levels (RR/Serializable) or lower ones (RU), we need a clear process to manage changes:

Change Request Process: Teams must submit a change request detailing the reasons for opting in or out of the default.
Review and Approval: Changes are reviewed by an internal standards team to ensure they do not introduce unacceptable risks.
Step 3: Audit Trails and Monitoring

Implementing audit trails ensures that any deviation from the default isolation level is logged, making it easy to track and revert if necessary. Continuous monitoring helps in identifying and rectifying silent guarantee drift early:

#### Example
Schema for Monitoring:
SQL
CREATE TABLE IF NOT EXISTS `isolation_level_changes` (
  `id` INT AUTO_INCREMENT PRIMARY KEY,
  `service_name` VARCHAR(255),
  `old_isolation_level` ENUM('RU', 'RC', 'RR', 'SERIALIZABLE'),
  `new_isolation_level` ENUM('RU', 'RC', 'RR', 'SERIALIZABLE'),
  `change_date` TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  `reason` TEXT
);

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, setting a defensible default isolation level of Read Committed (RC) with opt-in mechanisms ensures that we maintain a balance between performance and data integrity. The key is to have clear processes for changing the default settings and robust monitoring to prevent silent guarantee drift.

#### Common mistakes
include:

Failing to document the reasoning behind changes.
Lack of proper change control, leading to unauthorized modifications.
Insufficient monitoring, resulting in undetected consistency issues.

#### Follow-up
questions might explore additional scenarios such as handling edge cases during concurrent transactions or ensuring compliance with regulatory requirements.

