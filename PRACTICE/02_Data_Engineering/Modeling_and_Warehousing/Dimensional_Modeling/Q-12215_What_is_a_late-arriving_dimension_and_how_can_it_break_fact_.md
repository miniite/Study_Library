---
type: "practice"
difficulty: "Deep"
---

# What is a late-arriving dimension and how can it break fact loading?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Special Dimension Types](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Special_Dimension_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In data warehousing, late-arriving dimensions refer to new or updated dimension records that are inserted into the system after some facts have already been loaded. This can lead to inconsistencies in the star or snowflake schema, as previously ingested fact data might not be correctly linked with these new dimensions.

#### Triage
The question is about understanding how late-arriving dimensions impact the loading of facts in a database model and their potential consequences on the integrity of your data warehouse. We assume you have a basic understanding of star and snowflake schemas, as well as the process of fact loading. The key idea here is to explore why and how these late-arriving dimensions can create issues.

First-principles reasoning: Let's start by considering the structure of a star schema. It consists of one or more fact tables that are joined with dimension tables via foreign keys. Each row in a fact table corresponds to an event, transaction, or metric, while each dimension table contains descriptive attributes related to those facts. When new dimensions (late-arriving) are introduced after some facts have been loaded, it’s challenging to accurately backdate these facts without creating inconsistencies.

Understanding the Problem: Suppose you have a sales fact table with records of daily sales for various products across different stores over several months. After six months, you realize that there is an important new dimension called "Store Location" that wasn't considered when loading the initial data. Now, how do you properly link these existing sales facts with their corresponding locations?

Impact on Data Integrity: Late-arriving dimensions can lead to incorrect linking of fact records and thus distort analysis results. For instance, if you try to retroactively assign a location to past sales without affecting other existing dimensions, you might violate the integrity constraints or cause data anomalies.

Potential Solutions:

Late Loading Approach: You could design your ETL process to handle late-arriving dimensions by scheduling regular updates that include these new dimensions. This way, all facts are consistently linked with accurate and complete dimension information.
Historical Linking: Another approach is to maintain historical records of each fact’s associated dimensions, allowing for the possibility of changes or corrections over time.

#### Trade-offs
and Best Practices:

Performance: Frequent updates can impact performance, especially in large databases.
Data Consistency: Ensuring that all facts are correctly linked with their respective dimensions requires careful planning and robust ETL processes.
Scalability: The approach you choose should also consider the scalability of your system as the number of late-arriving dimensions increases.

#### Closing summary
Late-arriving dimensions pose a significant challenge in data warehousing by disrupting the integrity of fact tables. Addressing this issue requires thoughtful design and implementation strategies to ensure accurate and consistent data throughout the lifecycle of the warehouse. Common mistakes include ignoring these issues, leading to incorrect analysis or data anomalies. To mitigate risks, adopting a robust ETL strategy is crucial.

**Comparison:**

```
Fact Sales	Store Locations
sales_id	1,2,3
store_id	A,B,C
product_id	X,Y,Z
```

**Code:**

```sql
-- Example ETL Process for Late-Loading Dimensions

-- Step 1: Identify late-arriving dimensions
WITH new_dimensions AS (
    SELECT * FROM store_locations WHERE location_id NOT IN (SELECT DISTINCT location_id FROM fact_sales)
)

-- Step 2: Update the dimension table to ensure completeness
INSERT INTO store_locations (location_id, city, state)
SELECT * FROM new_dimensions;

-- Step 3: Update fact tables with late-arriving dimensions
UPDATE fact_sales fs
SET fs.location_id = sl.location_id
FROM store_locations sl
WHERE fs.store_id = sl.store_id AND NOT EXISTS (
    SELECT 1 FROM fact_sales fs2 WHERE fs2.store_id = sl.store_id AND fs2.location_id = sl.location_id
);
```

**Example input / output:**

```text
Input:
- Existing fact_sales table with incomplete location information.
- New store_locations data including the missing locations.

Output:
- fact_sales updated to include correct location_id based on existing and new store_locations.
```

**Data:**

```json
{
  "fact_sales": {
    "sales_id": [1, 2, 3],
    "store_id": [A, B, C],
    "product_id": [X, Y, Z],
    "sale_date": ["2023-10-01", "2023-10-02", "2023-10-03"],
    "location_id": [NULL, NULL, 1] // Initially missing location information
  },
  "store_locations": {
    "location_id": [1, 2, 3],
    "city": ["New York", "Los Angeles", "Chicago"],
    "state": ["NY", "CA", "IL"]
  }
}
```

**Flow:**

1. Identify late-arrivingdimensions
2. (Insert new dimensions intothe dimension table)
3. Update fact tables withaccurate dimensioninformation
4. Ensure consistent and accuratedata in the warehouse

