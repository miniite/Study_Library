---
type: "practice"
difficulty: "Deep"
---

# What is an accumulating snapshot fact table and give a real example such as order fulfillment.

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Data Warehouse Fundamentals](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Data_Warehouse_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

An accumulating snapshot fact table in data warehousing is used to track changes over time while maintaining historical context, essentially capturing the cumulative state of an entity at each point in time. This approach contrasts with a traditional transactional fact table which might only record individual transactions without retaining historical states.

#### Triaging the Question

The interviewer is likely probing your understanding of how data warehousing systems handle incremental updates and maintain historical information. Key assumptions include familiarity with the differences between accumulating snapshots, transactional facts, and slowly-changing dimensions (SCDs).

To solve this problem from first principles:

Understand the Entity: Identify what an "order fulfillment" process entails.
Determine Key Metrics: What quantities or states of the order are relevant over time?
Design Accumulating Snapshot Columns: These columns will capture the state at each snapshot point.

#### First-Principles Solution
Step 1: Define the Entity and Key Metrics

In an "order fulfillment" scenario, key metrics include:

Order ID (unique identifier for each order)
Customer ID
Product ID
Quantity of product ordered
Fulfillment Date
Step 2: Identify Snapshot Points

Snapshot points in this context could be at the end of each day or week. Each snapshot captures the state of all fulfilled orders up to that point.

Step 3: Design Accumulating Snapshot Columns

For an accumulating snapshot fact table:

Order ID: Unique identifier for each order.
Customer ID: Identifier for the customer placing the order.
Product ID: Identifier for the product being ordered.
Quantity Ordered: Total quantity of the product ordered over all snapshots up to the current one.
Fulfillment Date: The date at which this snapshot was recorded.

The schema might look like:

SQL
CREATE TABLE Fulfillment_Snapshots (
    OrderID INT,
    CustomerID INT,
    ProductID INT,
    Quantity_Ordered INT,
    Fulfillment_Date DATE,
    SnapshotDate DATE  -- Time when the state is captured
);
Step 4: Inserting Data

When a new fulfillment occurs, you insert or update the snapshot table:

SQL
INSERT INTO Fulfillment_Snapshots (OrderID, CustomerID, ProductID, Quantity_Ordered, Fulfillment_Date, SnapshotDate)
VALUES (101, 2023, 456, 3, '2023-10-01', '2023-10-01');

-- For subsequent updates
UPDATE Fulfillment_Snapshots
SET Quantity_Ordered = Quantity_Ordered + 2
WHERE OrderID = 101 AND SnapshotDate <= '2023-10-05';

#### Closing Summary
and Common Mistakes/Follow-Ups

To summarize, an accumulating snapshot fact table is crucial for maintaining a historical record of changes over time while ensuring each snapshot captures the cumulative state. In our example with order fulfillment, this approach helps track the total quantity fulfilled per product up to each snapshot point.

#### Common Mistakes

Forgetting to include SnapshotDate which tracks when the state was captured.
Not updating existing rows correctly (inserting a new row instead of updating).

#### Follow-ups
might explore how to handle more complex scenarios, such as partial fulfillments or overlapping snapshots.

**Comparison:**

| OrderID | CustomerID | ProductID | Quantity_Ordered | Fulfillment_Date | SnapshotDate |
|---------|------------|-----------|------------------|------------------|--------------|
| 101     | 2023       | 456       | 5                | 2023-10-01       | 2023-10-05   |

**Code:**

```sql
CREATE TABLE Fulfillment_Snapshots (
    OrderID INT,
    CustomerID INT,
    ProductID INT,
    Quantity_Ordered INT,
    Fulfillment_Date DATE,
    SnapshotDate DATE
);

INSERT INTO Fulfillment_Snapshots (OrderID, CustomerID, ProductID, Quantity_Ordered, Fulfillment_Date, SnapshotDate)
VALUES (101, 2023, 456, 3, '2023-10-01', '2023-10-01');

UPDATE Fulfillment_Snapshots
SET Quantity_Ordered = Quantity_Ordered + 2
WHERE OrderID = 101 AND SnapshotDate <= '2023-10-05';
```

**Example input / output:**

```text
INSERT INTO Fulfillment_Snapshots (OrderID, CustomerID, ProductID, Quantity_Ordered, Fulfillment_Date, SnapshotDate)
VALUES (101, 2023, 456, 3, '2023-10-01', '2023-10-01');

-- Subsequent update
UPDATE Fulfillment_Snapshots
SET Quantity_Ordered = Quantity_Ordered + 2
WHERE OrderID = 101 AND SnapshotDate <= '2023-10-05';
```

**Data:**

```json
{
  "OrderID": 101,
  "CustomerID": 2023,
  "ProductID": 456,
  "Quantity_Ordered": 3,
  "Fulfillment_Date": "2023-10-01",
  "SnapshotDate": "2023-10-01"
}
```

**Flow:**

1. Yes
2. No
3. Order Fulfillment
4. Is the order fulfilled?
5. Fulfill Order
6. Update Snapshot Table
7. Log New State in Snapshot
8. Continue Monitoring

