---
type: "practice"
difficulty: "Intermediate"
---

# Which approach, Kimball or Inmon, do you associate with star schemas and why?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Kimball Approach](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Kimball_Approach.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

In this response, we'll explore the differences between Kimball's and Inmon's approaches to data modeling, focusing on their relation to star schemas. We'll start by understanding what each methodology entails and then delve into why Kimball’s approach is more commonly associated with star schemas.

#### Triaging the Question

The question asks us to identify which data modeling approach—Kimball or Inmon—is typically linked with star schemas, and provides an opportunity to discuss the foundational principles of both methodologies. The core concepts involve understanding how data is organized and structured for business intelligence (BI) and analytics purposes.

#### Assumptions
and Context
Kimball Approach: Emphasizes a bottom-up approach where the model starts by defining the star schema.
Inmon Approach: Advocates for a top-down approach that focuses on creating a single integrated data warehouse.
Star Schema: A type of database schema consisting of one fact table along with multiple dimension tables, commonly used in BI and analytics.
Detailed Explanation from First Principles
Kimball’s Approach

Kimball’s methodology is widely adopted in the business intelligence community because it focuses on building a data mart first and then expanding to create additional data marts as needed. This approach aligns well with star schemas for several reasons:

Fact Table Focus: In a Kimball model, a single fact table forms the core of the schema, surrounded by multiple dimension tables. This structure is ideal for representing transactional data in a way that supports fast querying and analysis.
Business Requirement Driven: The design process begins with understanding business requirements and what metrics need to be tracked. The fact table captures these metrics, while dimensions provide context (e.g., time, location).
Performance Optimization: Fact tables are designed for high-performance data retrieval, often using denormalization techniques that reduce the number of joins required during analysis.
Inmon’s Approach

In contrast, Inmon's approach is more focused on creating a single integrated database that contains all necessary information from various sources. While this can be useful for comprehensive data management and governance, it doesn’t naturally align as closely with star schemas:

Holistic Design: Inmon advocates starting with a single, large warehouse that includes all possible dimensions and facts. This approach is more about unifying disparate systems rather than building specialized models.
Data Quality First: The initial focus is on ensuring high data quality before proceeding to build specific analytical models.
Why Kimball’s Approach is More Commonly Linked with Star Schemas

Star schemas are inherently bottom-up, which means they start by defining the core fact table and then add dimensions as needed. This aligns perfectly with the Kimball methodology:

Modularity: Kimball’s approach allows for modular design where new star schemas can be created without disturbing existing ones.
Scalability: As business needs evolve, additional star schemas can be added to address specific reporting or analytics requirements.
Performance: The denormalization techniques used in fact tables enhance query performance, which is critical for real-time BI and analysis.

#### Closing Summary

Kimball’s approach is more closely aligned with star schemas because it focuses on building specialized models (data marts) that are optimized for specific analytical purposes. This makes it easier to manage complex data environments where multiple types of analyses are required without overcomplicating the overall architecture.

#### Common Mistakes
and Follow-Ups
Misunderstanding Key Concepts: Failing to grasp the fundamental differences between Kimball’s bottom-up approach and Inmon’s top-down approach can lead to confusion.
Over-Engineering Data Warehouses: While a unified warehouse is valuable, it might not always be necessary or efficient for every organization, especially those with varying analytical needs.
Triangulating the Answer

To provide a balanced view:

For organizations with specific BI requirements, Kimball’s approach may offer more flexibility and performance.
Inmon’s methodology might be better suited for large enterprises where data integration is a primary concern.

In conclusion, while both approaches have their merits, Kimball’s method is more commonly associated with star schemas due to its modular and performant nature.

**Comparison:**

```
Fact Table (SalesFact)	Dimension Tables
SalesID, ProductID, CustomerID, StoreID, DateID, QuantitySold, Amount	ProductName, CustomerName, StoreName, Location, Year, Month, Day
SalesID	ProductID	CustomerID	StoreID	DateID	QuantitySold	Amount
1	100	4000	700	2023-01-01	50	49.99
ProductID	ProductName
100	Laptop
CustomerID	FirstName	LastName
4000	John	Doe
StoreID	StoreName	Location
700	Central	NYC
DateID	Year	Month	Day
2023-01-01	2023	1	1
```

**Code:**

```sql
-- Example of a simple fact table in Kimball's approach
CREATE TABLE SalesFact (
    SalesID INT PRIMARY KEY,
    ProductID INT,
    CustomerID INT,
    StoreID INT,
    DateID DATE,
    QuantitySold INT,
    Amount DECIMAL(10, 2)
);
```

**Example input / output:**

```text
Input:
- SalesFact table with sales data

Output:
- Query to join SalesFact with ProductDim, CustomerDim, StoreDim, and DateDim tables for analysis.
```

**Data:**

```json
{
  "SalesFact": {
    "SalesID": [1, 2, 3],
    "ProductID": [100, 200, 300],
    "CustomerID": [4000, 5000, 6000],
    "StoreID": [700, 800, 900],
    "DateID": ["2023-01-01", "2023-01-02", "2023-01-03"],
    "QuantitySold": [50, 75, 60],
    "Amount": [49.99, 74.99, 59.99]
  },
  "ProductDim": {
    "ProductID": [100, 200, 300],
    "ProductName": ["Laptop", "Smartphone", "Tablet"]
  },
  "CustomerDim": {
    "CustomerID": [4000, 5000, 6000],
    "FirstName": ["John", "Jane", "James"],
    "LastName": ["Doe", "Smith", "Johnson"]
  },
  "StoreDim": {
    "StoreID": [700, 800, 900],
    "StoreName": ["Central Store", "East Store", "West Store"],
    "Location": ["NYC", "Chicago", "Los Angeles"]
  },
  "DateDim": {
    "DateID": ["2023-01-01", "2023-01-02", "2023-01-03"],
    "Year": [2023, 2023, 2023],
    "Month": [1, 1, 1],
    "Day": [1, 2, 3]
  }
}
```

**Flow:**

1. No
2. Yes
3. Fact Table (SalesFact)
4. Is the analysis specific?
5. Build a single integrated datawarehouse
6. Build a data mart with starschema
7. Unified data management andgovernance
8. Optimized for specific analytics
9. Complex, but holistic design
10. Modular, scalable solution

