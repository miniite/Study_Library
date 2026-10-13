---
type: "practice"
difficulty: "Expert"
---

# How do you handle product master data (PIM) where the same SKU has different attributes per channel/region — is that one master with variants, many masters, or a different pattern entirely?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Multi-domain MDM Programs](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Multi-domain_MDM_Programs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To manage product master data (PIM) in scenarios where an SKU has different attributes across channels and regions, we need to consider both the business requirements and technical feasibility. The approach can vary widely depending on whether you are dealing with a single global master record, multiple local masters for each region, or some hybrid model. Here's how we can reason through this problem from first principles.

#### Triaging the Question

The core of the question is about handling attribute variability in product data across different channels and regions. The interviewer is probing your understanding of Master Data Management (MDM) concepts, specifically around data normalization, decentralization, and consistency. We'll need to make some key assumptions:

Different attributes can mean varying fields like pricing, availability, or localized descriptions.
Channels might refer to online platforms, stores, apps, etc., which could have unique requirements.

#### First Principles Solution

Identify Business Requirements:

Single Master with Variants: This approach consolidates all product data into a single master record but allows for variant records that capture channel-specific attributes.
Multiple Masters (Decentralized): Each region or channel maintains its own master copy of the data, which can lead to better local control and performance.
Hybrid Model: A combination where you have one global master with localized variants. This strikes a balance between centralization and decentralization.

Data Model Design:

Single Master with Variants:
SQL
CREATE TABLE Products (
    ProductID INT PRIMARY KEY,
    SKU VARCHAR(10),
    Brand VARCHAR(50),
    Category VARCHAR(50),
    Description TEXT
);

CREATE TABLE ChannelAttributes (
    ChannelAttributeID INT PRIMARY KEY,
    ProductID INT,
    Channel VARCHAR(50),
    Price DECIMAL(8, 2),
    Availability BOOLEAN,
    FOREIGN KEY (ProductID) REFERENCES Products(ProductID)
);
Multiple Masters:
SQL
CREATE TABLE RegionalProducts (
    ProductID INT PRIMARY KEY,
    SKU VARCHAR(10),
    Brand VARCHAR(50),
    Category VARCHAR(50),
    Description TEXT
);

CREATE TABLE ChannelAttributes (
    AttributeID INT PRIMARY KEY,
    ProductID INT,
    Channel VARCHAR(50),
    Price DECIMAL(8, 2),
    Availability BOOLEAN,
    FOREIGN KEY (ProductID) REFERENCES RegionalProducts(ProductID)
);
Hybrid Model:
SQL
CREATE TABLE GlobalProducts (
    ProductID INT PRIMARY KEY,
    SKU VARCHAR(10),
    Brand VARCHAR(50),
    Category VARCHAR(50),
    Description TEXT
);

CREATE TABLE ChannelAttributes (
    AttributeID INT PRIMARY KEY,
    ProductID INT,
    Channel VARCHAR(50),
    Price DECIMAL(8, 2),
    Availability BOOLEAN,
    FOREIGN KEY (ProductID) REFERENCES GlobalProducts(ProductID)
);

CREATE TABLE RegionalVariants (
    VariantID INT PRIMARY KEY,
    ProductID INT,
    Region VARCHAR(50),
    AttributeID INT,
    FOREIGN KEY (ProductID) REFERENCES GlobalProducts(ProductID),
    FOREIGN KEY (AttributeID) REFERENCES ChannelAttributes(AttributeID)
);

#### Cons
istency and Governance:

Establish a governance framework to ensure data consistency, especially in the hybrid model where changes need to be propagated.
Use data lineage tools to track how updates are handled across different masters.

#### Closing Summary

In summary, handling product master data with varying attributes per channel/region requires careful consideration of business needs and technical constraints. A single master with variants provides centralized control but can introduce complexity in variant management. Multiple regional masters offer better local control but complicate data consistency. A hybrid model balances these concerns by maintaining a global record while allowing localized adjustments.

#### Common Mistakes/Follow-ups
Failing to Consider Performance: Ensuring that the chosen approach doesn’t degrade performance, especially in read-heavy scenarios.
Ignoring Data Integrity: Focusing solely on functionality without proper checks and balances.
Lack of Scalability: Not considering how the solution will scale as the number of channels or regions grows.

**Code:**

```

-- Single Master with Variants Schema CREATE TABLE Products ( ProductID INT PRIMARY KEY, SKU VARCHAR(10), Brand VARCHAR(50), Category VARCHAR(50), Description TEXT );

CREATE TABLE ChannelAttributes ( ChannelAttributeID INT PRIMARY KEY, ProductID INT, Channel VARCHAR(50), Price DECIMAL(8, 2), Availability BOOLEAN, FOREIGN KEY (ProductID) REFERENCES Products(ProductID) );

CODE
Copy

```sql
-- Multiple Masters Schema
CREATE TABLE RegionalProducts (
    ProductID INT PRIMARY KEY,
    SKU VARCHAR(10),
    Brand VARCHAR(50),
    Category VARCHAR(50),
    Description TEXT
);

CREATE TABLE ChannelAttributes (
    AttributeID INT PRIMARY KEY,
    ProductID INT,
    Channel VARCHAR(50),
    Price DECIMAL(8, 2),
    Availability BOOLEAN,
    FOREIGN KEY (ProductID) REFERENCES RegionalProducts(ProductID)
);
SQL
Copy
-- Hybrid Model Schema
CREATE TABLE GlobalProducts (
    ProductID INT PRIMARY KEY,
    SKU VARCHAR(10),
    Brand VARCHAR(50),
    Category VARCHAR(50),
    Description TEXT
);

CREATE TABLE ChannelAttributes (
    AttributeID INT PRIMARY KEY,
    ProductID INT,
    Channel VARCHAR(50),
    Price DECIMAL(8, 2),
    Availability BOOLEAN,
    FOREIGN KEY (ProductID) REFERENCES GlobalProducts(ProductID)
);

CREATE TABLE RegionalVariants (
    VariantID INT PRIMARY KEY,
    ProductID INT,
    Region VARCHAR(50),
    AttributeID INT,
    FOREIGN KEY (ProductID) REFERENCES GlobalProducts(ProductID),
    FOREIGN KEY (AttributeID) REFERENCES ChannelAttributes(AttributeID)
);
```

