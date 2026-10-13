---
type: "practice"
difficulty: "Deep"
---

# What's your framework for build-vs-buy on schema-migration tooling — when do you standardize on Flyway/Liquibase/Atlas/Bytebase vs build in-house, and how do you avoid lock-in at fleet scale?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Coordinated Multi-store Schema Changes](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution/Coordinated_Multi-store_Schema_Changes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding between building an in-house schema-migration tool or using off-the-shelf solutions like Flyway, Liquibase, Atlas, or Bytebase, a robust framework must be established to consider multiple factors such as existing tech stack, team expertise, project scale, and long-term maintainability. Let's dive into the decision-making process from first principles.

#### Triaging the Question

The question is essentially asking for a strategic approach on when to standardize on pre-built solutions versus building custom tools. The key aspects being probed are:

Risk Management: Evaluating potential technical debt and vendor lock-in.
Team Expertise and Infrastructure: Assessing the current team's capabilities and infrastructure readiness.
Project Scale and Complexity: Considering whether a small or large project would benefit more from a pre-built solution versus an in-house build.
Long-term Maintainability: Ensuring that any chosen tool will meet future needs without significant refactoring.

#### Assumptions
The database is relational, as Flyway, Liquibase, and Atlas are primarily designed for SQL schema management.
The project has a need for robust version control of the database schema.
The team is willing to invest time in setting up and maintaining any solution chosen.
Step-by-Step Solution
1. Evaluate Existing Tech Stack

First, assess the current tech stack. If your organization already heavily uses Flyway or Liquibase within other projects, it might be more efficient to standardize on one of these tools due to ease of integration and shared expertise. Conversely, if there are no existing users, building in-house may provide unique insights into how the tool can be tailored specifically for your needs.

2. Consider Team Expertise

Evaluate the team's skill set:

Flyway/Liquibase: These tools have extensive documentation and community support, making them accessible even to those with limited experience.
Atlas/Bytebase: While these might require more setup time, they offer advanced features like UI-based migrations and CI/CD integrations.
3. Assess Project Scale

The scale of the project plays a critical role:

Small Projects: For smaller projects where customization isn't needed, using a pre-built tool with a proven track record can save time.
Large Projects or Multi-Team Environments: For large-scale migrations across multiple teams and databases, robust tools like Atlas or Bytebase might be necessary to ensure consistency and visibility.
4. Long-term Maintainability

#### Cons
ider the long-term maintainability:

Custom Build: Building in-house allows for future customization but requires ongoing maintenance.
Pre-built Solutions: While there's a risk of vendor lock-in, pre-built tools often come with community support and updates that reduce long-term overhead.
5. Cost Considerations

Evaluate the total cost of ownership (TCO):

Development Time vs. Maintenance Costs: Custom builds might require significant upfront investment but lower maintenance costs.
Vendor TCO: Pre-built solutions often have associated licensing fees, but these can be offset by reduced development time and ongoing support.
6. Compliance and Security

Check compliance requirements:

Ensure that any tool chosen complies with internal policies and external regulations.

#### Closing Summary

In summary, the decision between building in-house or using a pre-built solution like Flyway, Liquibase, Atlas, or Bytebase hinges on a balanced evaluation of team expertise, project scale, long-term maintainability, cost considerations, and compliance requirements. A pragmatic approach is to standardize where possible but remain flexible enough to tailor solutions as needed.

#### Common Mistakes/Follow-Ups
Overlooking Long-Term Maintenance: Teams often underestimate the effort required to maintain custom-built tools.
Ignoring Vendor Support: Relying solely on in-house expertise without considering the benefits of community support and frequent updates.
Neglecting Security Compliance: Failing to consider how a chosen tool aligns with security policies.
Triaging Common Follow-Ups

What about databases other than SQL?

For NoSQL databases, tools like Atlas or Bytebase might not be as suitable, leading to the need for custom solutions.

How do we handle legacy systems?

Legacy systems often require more bespoke migration strategies that may not align with off-the-shelf tools.

**Comparison:**

| Tool       | Features              | Pros                         | Cons                    |
|------------|-----------------------|------------------------------|-------------------------|
| Flyway     | Simple, open-source   | Easy to use                  | Limited UI support      |
| Liquibase  | Advanced features     | Robust with extensive docs   | More complex setup      |
| Atlas      | Full-featured         | Comprehensive UI             | Cost of subscription    |
| Bytebase   | CI/CD ready           | Modern and well-supported    | Requires initial config|

**Code:**

```python
# Example of a basic Flyway setup in Python
def apply_migration():
    # Initialize Flyway instance
    flyway = Flyway.configure().locations('db/migration').load()

    # Migrate the database
    flyway.migrate()

# Example usage
apply_migration()
```

**Example input / output:**

```text
Input:
flyway migrate

Output:
+------------+-------------------+
| Version    | Description       |
+------------+-------------------+
| 1.0        | Initial schema    |
| 2.0        | Added user table  |
| 3.0        | Fixed bugs in UI  |
+------------+-------------------+
```

**Data:**

```json
{
  "flywayConfig": {
    "locations": "db/migration",
    "encoding": "UTF-8",
    "url": "jdbc:mysql://localhost:3306/mydb",
    "user": "root",
    "password": "pass"
  }
}
```

**Flow:**

1. Define Project Needs
2. Existing Tech Stack
3. Team Expertise
4. Project Scale
5. Long-term Maintainability
6. Cost Considerations
7. Compliance and Security
8. Decision Point: Build vs. Buy

