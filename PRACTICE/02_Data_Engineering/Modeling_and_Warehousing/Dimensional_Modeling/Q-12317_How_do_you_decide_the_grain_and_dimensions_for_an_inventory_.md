---
type: "practice"
difficulty: "Deep"
---

# How do you decide the grain and dimensions for an inventory snapshot when stock is tracked per store, per SKU, per day, and per bin location?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Fact Table Grain](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Fact_Table_Grain.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a database to track inventory snapshots with a granularity of tracking stocks per store, per SKU, per day, and per bin location, we need to carefully consider the grain and dimensions of our data model. The key is to strike a balance between accuracy, query performance, and storage efficiency.

The core problem here involves deciding on the appropriate level of detail (grain) for each dimension in the star or snowflake schema. We'll need to understand what “dimensions” mean in this context: store, SKU, day, and bin location are our primary dimensions. For instance, "store" might represent a specific retail outlet, "SKU" is the stock-keeping unit (e.g., a particular product), "day" is the date of the inventory count, and "bin location" could be the storage area in a warehouse.

#### Triage

The question is essentially asking about how to structure our data model for an accurate representation of daily inventory while ensuring efficient querying. The interviewer might be probing into:

Understanding of star vs. snowflake schema.

#### Trade-offs
between data granularity and query performance.

#### Cons
ideration of storage costs.
Potential future requirements that could influence the design.

#### First Principles Solution
1. Defining Dimensions and Facts
Dimensions: These include store_id, sku_id, day, bin_location.
Facts: The fact table will hold the actual inventory counts, such as quantity_on_hand.
2. Choosing the Grain

Given that we need to track stock per store, per SKU, per day, and per bin location, a snowflake schema would be suitable because it allows for better normalization of dimensions.

Store: This can be considered at a very high level since the store is likely not changing frequently.
SKU: Each product has its own unique identifier.
Day: The snapshot date is critical as we are capturing daily changes.
Bin Location: Fine-grained tracking within each SKU and day.
3. Dimension Table Design

For each dimension, create a separate table with surrogate keys:

SQL
CREATE TABLE stores (
    store_id INT PRIMARY KEY,
    store_name VARCHAR(255)
);

CREATE TABLE skus (
    sku_id INT PRIMARY KEY,
    product_name VARCHAR(255),
    category VARCHAR(100)
);

CREATE TABLE days (
    day_date DATE PRIMARY KEY,
    day_of_year INT
);

CREATE TABLE bin_locations (
    bin_location_id INT PRIMARY KEY,
    store_id INT,
    section VARCHAR(50),
    row INT,
    column INT,
    FOREIGN KEY (store_id) REFERENCES stores(store_id)
);
4. Fact Table Design

The fact table will be a wide and flat structure, capturing the inventory counts:

SQL
CREATE TABLE inventory_snapshots (
    snapshot_id INT AUTO_INCREMENT PRIMARY KEY,
    store_id INT,
    sku_id INT,
    day_date DATE,
    bin_location_id INT,
    quantity_on_hand INT,
    FOREIGN KEY (store_id) REFERENCES stores(store_id),
    FOREIGN KEY (sku_id) REFERENCES skus(sku_id),
    FOREIGN KEY (day_date) REFERENCES days(day_date),
    FOREIGN KEY (bin_location_id) REFERENCES bin_locations(bin_location_id)
);

#### Closing Summary
and Common Mistakes

In this schema, each grain of data is finely detailed but normalized. The dimensions are separated into their own tables to ensure referential integrity and ease of querying.

#### Common mistakes
include:

Over-normalizing the fact table, which can lead to performance issues.
Under-normalizing the dimension tables, leading to redundancy.
Failing to consider storage costs and query performance trade-offs.
Additional Considerations

Future requirements might include additional dimensions such as "employee" who last touched an inventory count or "warehouse conditions." These could be added in a similar manner, ensuring that the data model remains flexible yet efficient.

**Code:**

```

-- Example code for dimension and fact table creation

CREATE TABLE stores ( store_id INT PRIMARY KEY, store_name VARCHAR(255) );

CREATE TABLE skus ( sku_id INT PRIMARY KEY, product_name VARCHAR(255), category VARCHAR(100) );

CREATE TABLE days ( day_date DATE PRIMARY KEY, day_of_year INT );

CREATE TABLE bin_locations ( bin_location_id INT PRIMARY KEY, store_id INT, section VARCHAR(50), row INT, column INT, FOREIGN KEY (store_id) REFERENCES stores(store_id) );

CODE
Copy

```sql
-- Fact table creation

CREATE TABLE inventory_snapshots (
    snapshot_id INT AUTO_INCREMENT PRIMARY KEY,
    store_id INT,
    sku_id INT,
    day_date DATE,
    bin_location_id INT,
    quantity_on_hand INT,
    FOREIGN KEY (store_id) REFERENCES stores(store_id),
    FOREIGN KEY (sku_id) REFERENCES skus(sku_id),
    FOREIGN KEY (day_date) REFERENCES days(day_date),
    FOREIGN KEY (bin_location_id) REFERENCES bin_locations(bin_location_id)
);
```

