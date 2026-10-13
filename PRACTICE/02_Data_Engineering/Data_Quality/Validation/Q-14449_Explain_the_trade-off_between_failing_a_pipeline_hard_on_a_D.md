---
type: "practice"
difficulty: "Deep"
---

# Explain the trade-off between failing a pipeline hard on a DQ breach versus loading with a quarantine table and alerting. When have you chosen each in production?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Validation Coverage and Quarantine](<../../../../02_Data_Engineering/Data_Quality/Validation/Validation_Coverage_and_Quarantine.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To explain the trade-offs between failing a data pipeline hard on a Data Quality (DQ) breach versus loading with a quarantine table and alerting, we need to consider several factors including system resilience, operational impact, cost, and business criticality. In a production environment, these decisions are often driven by the nature of the DQ issues and the risk tolerance of the organization.

#### Triage
Understanding the Question

The core question here is about balancing immediate data integrity against long-term stability and operational efficiency. The interviewer wants to understand how you would handle DQ breaches in different scenarios and the rationale behind your choices. They are probing for an understanding of trade-offs, such as the impact on downstream processes, potential risks if incorrect data is propagated, and the importance of notifications for timely intervention.

Triaging Assumptions
Context: We're dealing with a Data Pipeline that includes ETL (Extract, Transform, Load) processes.
DQ Breaches: These could range from minor inconsistencies to critical errors affecting the integrity of the data.
System Impact: The system should be designed to handle various levels of DQ breaches without completely failing.
Alerting Mechanism: This includes notifications via emails, SMS, or other alert mechanisms.
Step-by-Step First Principles Solution
Failing Hard on a DQ Breach
Immediate Integrity Assurance: Ensures that only high-quality data is processed and stored, maintaining the integrity of downstream processes.
Risk Mitigation: Prevents incorrect data from being used in critical business operations or analytics, reducing risk.
Operational Stability: Simplifies operational complexity by avoiding quarantine tables, ensuring a cleaner state.

#### Pros

Ensures data quality is maintained at all costs.
Simplifies monitoring and validation processes.

#### Cons

May lead to pipeline failures, causing potential downtime.
Can disrupt business operations if the breach affects critical data streams.
Loading with Quarantine Table and Alerting
Data Segregation: Separates erroneous data into quarantine tables, allowing it to be reviewed and corrected without disrupting other processes.
Operational Continuity: Ensures that the main pipeline continues running, minimizing downtime and maintaining service availability.
Risk Management: Provides a mechanism for addressing DQ issues in a controlled manner, reducing the risk of incorrect data being used.

#### Pros

Maintains operational continuity by avoiding hard failures.
Provides time to investigate and resolve issues without immediate disruption.

#### Cons

Can lead to propagation of erroneous data if not managed properly.
Adds complexity in managing quarantine tables and alerting mechanisms.

#### Closing Summary

Choosing between failing a pipeline on DQ breaches versus loading with quarantine tables depends heavily on the specific business context. Failing hard is appropriate for high-risk, mission-critical pipelines where data integrity cannot be compromised. On the other hand, using quarantine tables offers better operational continuity at the cost of potential data propagation risks.

#### Common Mistakes
and Follow-Ups
Mistake 1: Ignoring DQ breaches entirely can lead to catastrophic data issues.
Mistake 2: Failing hard too frequently can cause unnecessary downtime and impact user trust.

#### Follow-Up
Questions:
How do you prioritize different types of DQ checks?
What are the implications of using a quarantine table for real-time vs. batch processing pipelines?

#### Example Scenario

In one project, we faced an issue where incorrect data was being loaded due to a validation failure in a high-traffic transactional system. The decision was made to fail the pipeline immediately because the risk of incorrect data affecting downstream critical reports was too high. In contrast, for slower-running batch processes that were less critical, we implemented quarantine tables and robust alerting mechanisms.

**Comparison:**

```
Scenario	Action	Pros	Cons
Failing Hard	Fail pipeline	Ensures high-quality data	Can cause downtime
	on DQ breach	Risk mitigation	May disrupt business ops
Quarantine Table	Load with alert	Maintains operational	Risk of erroneous data
	on DQ breach	Time to correct issues	Adds complexity
```

**Code:**

```
Example validation function in Python

def validate_data(data): if not is_valid_data(data): # Placeholder for actual validation logic raise DataValidationException("Invalid data detected")

CODE
Copy

```sql
-- SQL example of failing the pipeline on DQ breach
INSERT INTO main_table (data)
VALUES (:data)
ON DUPLICATE KEY UPDATE status = 'ERROR';
```

**Example input / output:**

```text
Input:
- Data containing invalid values

Output:
- Pipeline fails immediately with a DataValidationException, stopping further processing.
- No data is loaded into the main table.
```

**Data:**

```json
{
  "data_source": "transactional_system",
  "validation_status": "failed",
  "error_message": "Invalid data detected"
}
```

**Flow:**

1. Yes
2. No
3. Data Pipeline
4. Is Data Valid?
5. Load Data into Main Table
6. Fail Pipeline with Error

