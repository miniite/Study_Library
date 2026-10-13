---
type: "practice"
difficulty: "Basic"
---

# What does 'slowly changing dimension' mean in plain language?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ SCD Type 2 vs Mini-dimensions](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/SCD_Type_2_vs_Mini-dimensions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To explain "slowly changing dimension" in plain language, let's break it down first. A slowly changing dimension (SCD) is a concept used in data modeling to handle changes in attributes of entities over time. Unlike static dimensions, which remain constant, SCDs evolve as new information becomes available.

In the context of a data warehouse or analytics system, SCDs are crucial because real-world business processes often involve historical changes that need to be tracked and reflected accurately. For example, a customer's address might change, or an employee’s department might shift over time. Handling these changes correctly is essential for maintaining the integrity and usefulness of the data.

Now, let's dive into the different types of SCDs from first principles:

Type 0: This is the simplest type where no new records are created to reflect historical changes. Instead, attributes change within the same record over time. However, this can lead to inaccurate reporting and analysis if not properly managed.

Type 1: In Type 1 SCDs, existing records are updated with the latest information. This approach is straightforward but can obscure historical data, making it challenging to trace changes over time.

Type 2: Here, a new record is created for each change in an attribute, while the original record remains intact. Each version of the dimension captures a snapshot at different points in time. Type 2 SCDs are widely used because they preserve historical data and allow for accurate analysis across periods.

Type 3: This type involves creating separate dimensions for current and historical records. It’s more complex but provides detailed tracking capabilities, making it suitable for advanced analytics needs where precise historical context is critical.

To illustrate, consider a company's employee table:

Type 1: If an employee changes departments, the department field in their existing record would be updated.
Type 2: A new record with the current department and an effective date (e.g., start of the month) would be added, keeping the old records unchanged.
Type 3: Separate tables for "Employees_Current" and "Employees_Historical" could be maintained, where historical changes are tracked separately.

The choice between these types depends on specific business requirements, such as whether historical context is needed, the complexity of data queries, and storage constraints.

In summary, slowly changing dimensions help manage evolving attributes in a structured way, ensuring that historical data remains available for accurate reporting and analysis.

