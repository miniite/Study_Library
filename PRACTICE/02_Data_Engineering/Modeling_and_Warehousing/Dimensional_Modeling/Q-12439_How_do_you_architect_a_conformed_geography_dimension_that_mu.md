---
type: "practice"
difficulty: "Deep"
---

# How do you architect a conformed geography dimension that must serve legal-entity, tax, shipping, and sales-territory rollups that disagree on boundaries?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Special Dimension Types](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Special_Dimension_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect a conformed geography dimension serving multiple rollups with conflicting boundaries (legal-entity, tax, shipping, and sales-territory), we need to design a robust data model that integrates these disparate requirements while maintaining consistency and performance. The key challenge is to create a unified view of geography that can accommodate varying boundary definitions without introducing redundancy or unnecessary complexity.

#### Triaging the Question

The question requires us to define a conformed geography dimension in a star or snowflake schema, where different business processes have conflicting views on geographic boundaries. This involves understanding how these boundaries are used and ensuring that our model accurately captures their differences while providing a consistent view for analysis.

#### First Principles Solution
Define the Geography Hierarchy: Start by defining a hierarchical geography dimension that includes levels such as country, state/region, city, and postal code.
Create Separate Rollup Dimensions: For each rollup (legal-entity, tax, shipping, sales-territory), create separate dimensions with their own boundary definitions.
Bridge Tables: Use bridge tables to link the unified geography dimension to the specific rollups where boundaries differ.
Step-by-Step Solution

Geography Dimension

Define a hierarchy of geographic levels (country, state/region, city, postal code).
SQL
CREATE TABLE Geography (
    GeoID INT PRIMARY KEY,
    Country VARCHAR(50),
    StateRegion VARCHAR(100),
    City VARCHAR(100),
    PostalCode VARCHAR(20)
);

Legal Entity Rollup Dimension

Define a hierarchy specific to legal entities, capturing boundaries relevant for this rollup.
SQL
CREATE TABLE LegalEntityGeo (
    LEGeoID INT PRIMARY KEY,
    Country VARCHAR(50),
    StateRegion VARCHAR(100),
    City VARCHAR(100),
    PostalCode VARCHAR(20)
);

Tax Rollup Dimension

Define a hierarchy specific to tax boundaries.
SQL
CREATE TABLE TaxGeo (
    TaxGeoID INT PRIMARY KEY,
    Country VARCHAR(50),
    StateRegion VARCHAR(100),
    City VARCHAR(100),
    PostalCode VARCHAR(20)
);

Shipping Rollup Dimension

Define a hierarchy specific to shipping zones.
SQL
CREATE TABLE ShippingGeo (
    ShipGeoID INT PRIMARY KEY,
    Country VARCHAR(50),
    StateRegion VARCHAR(100),
    City VARCHAR(100),
    PostalCode VARCHAR(20)
);

Sales Territory Rollup Dimension

Define a hierarchy specific to sales territories.
SQL
CREATE TABLE SalesTerritoryGeo (
    STGeoID INT PRIMARY KEY,
    Country VARCHAR(50),
    StateRegion VARCHAR(100),
    City VARCHAR(100),
    PostalCode VARCHAR(20)
);

Bridge Tables

Link the unified geography dimension to each rollup.
SQL
CREATE TABLE GeoLE (
    GeoID INT,
    LEGeoID INT,
    FOREIGN KEY (GeoID) REFERENCES Geography(GeoID),
    FOREIGN KEY (LEGeoID) REFERENCES LegalEntityGeo(LEGeoID)
);

CREATE TABLE GeoTax (
    GeoID INT,
    TaxGeoID INT,
    FOREIGN KEY (GeoID) REFERENCES Geography(GeoID),
    FOREIGN KEY (TaxGeoID) REFERENCES TaxGeo(TaxGeoID)
);

CREATE TABLE GeoShip (
    GeoID INT,
    ShipGeoID INT,
    FOREIGN KEY (GeoID) REFERENCES Geography(GeoID),
    FOREIGN KEY (ShipGeoID) REFERENCES ShippingGeo(ShipGeoID)
);

CREATE TABLE GeoSales (
    GeoID INT,
    STGeoID INT,
    FOREIGN KEY (GeoID) REFERENCES Geography(GeoID),
    FOREIGN KEY (STGeoID) REFERENCES SalesTerritoryGeo(STGeoID)
);

#### Closing Summary

By using a combination of unified and specialized geography dimensions, along with bridge tables, we can create a flexible data model that accommodates different boundary definitions while providing a consistent view for analysis. This approach ensures that the model remains scalable, maintainable, and performs efficiently.

#### Common Mistakes/Follow-Ups
Overlapping Boundaries: Ensure that bridge table relationships do not create overlaps or duplicate entries.

#### Performance Considerations
Use indexed columns in bridge tables to optimize query performance when joining multiple rollups.
Validation Rules: Implement validation rules on the bridge tables to enforce consistency between unified and specialized dimensions.

**Comparison:**

```
FLOWCHART · MERMAID
erDiagram
    Geography ||--o{ GeoLE : is related to
    Geography ||--o{ GeoTax : is related to
    Geography ||--o{ GeoShip : is related to
    Geography ||--o{ GeoSales : is related to

    LegalEntityGeo | ||--| GeoLE
    TaxGeo | ||--| GeoTax
    ShippingGeo | ||--| GeoShip
    SalesTerritoryGeo | ||--| GeoSales
```

**Code:**

```sql
CREATE TABLE Geography (
    GeoID INT PRIMARY KEY,
    Country VARCHAR(50),
    StateRegion VARCHAR(100),
    City VARCHAR(100),
    PostalCode VARCHAR(20)
);

CREATE TABLE LegalEntityGeo (
    LEGeoID INT PRIMARY KEY,
    Country VARCHAR(50),
    StateRegion VARCHAR(100),
    City VARCHAR(100),
    PostalCode VARCHAR(20)
);

CREATE TABLE TaxGeo (
    TaxGeoID INT PRIMARY KEY,
    Country VARCHAR(50),
    StateRegion VARCHAR(100),
    City VARCHAR(100),
    PostalCode VARCHAR(20)
);

CREATE TABLE ShippingGeo (
    ShipGeoID INT PRIMARY KEY,
    Country VARCHAR(50),
    StateRegion VARCHAR(100),
    City VARCHAR(100),
    PostalCode VARCHAR(20)
);

CREATE TABLE SalesTerritoryGeo (
    STGeoID INT PRIMARY KEY,
    Country VARCHAR(50),
    StateRegion VARCHAR(100),
    City VARCHAR(100),
    PostalCode VARCHAR(20)
);

CREATE TABLE GeoLE (
    GeoID INT,
    LEGeoID INT,
    FOREIGN KEY (GeoID) REFERENCES Geography(GeoID),
    FOREIGN KEY (LEGeoID) REFERENCES LegalEntityGeo(LEGeoID)
);

CREATE TABLE GeoTax (
    GeoID INT,
    TaxGeoID INT,
    FOREIGN KEY (GeoID) REFERENCES Geography(GeoID),
    FOREIGN KEY (TaxGeoID) REFERENCES TaxGeo(TaxGeoID)
);

CREATE TABLE GeoShip (
    GeoID INT,
    ShipGeoID INT,
    FOREIGN KEY (GeoID) REFERENCES Geography(GeoID),
    FOREIGN KEY (ShipGeoID) REFERENCES ShippingGeo(ShipGeoID)
);

CREATE TABLE GeoSales (
    GeoID INT,
    STGeoID INT,
    FOREIGN KEY (GeoID) REFERENCES Geography(GeoID),
    FOREIGN KEY (STGeoID) REFERENCES SalesTerritoryGeo(STGeoID)
);
```

**Example input / output:**

```text
-- Example Query to Join Unified Geography with Legal Entity Rollup
SELECT G.Country, G.StateRegion, G.City, G.PostalCode,
       LE.Country AS LECountry, LE.StateRegion AS LERegion, LE.City AS LECity, LE.PostalCode AS LEPostalCode
FROM Geography G
JOIN GeoLE GL ON G.GeoID = GL.GeoID
JOIN LegalEntityGeo LE ON GL.LEGeoID = LE.LEGeoID;
```

**Data:**

```json
{
  "Geography": [
    { "GeoID": 1, "Country": "USA", "StateRegion": "CA", "City": "San Francisco", "PostalCode": "94105" },
    { "GeoID": 2, "Country": "USA", "StateRegion": "NY", "City": "New York", "PostalCode": "10001" }
  ],
  "LegalEntityGeo": [
    { "LEGeoID": 3, "Country": "USA", "StateRegion": "CA", "City": "San Francisco", "PostalCode": "94105" },
    { "LEGeoID": 4, "Country": "USA", "StateRegion": "NY", "City": "New York", "PostalCode": "10001" }
  ],
  "TaxGeo": [
    { "TaxGeoID": 5, "Country": "USA", "StateRegion": "CA", "City": "San Francisco", "PostalCode": "94105" },
    { "TaxGeoID": 6, "Country": "USA", "StateRegion": "NY", "City": "New York", "PostalCode": "10001" }
  ],
  "ShippingGeo": [
    { "ShipGeoID": 7, "Country": "USA", "StateRegion": "CA", "City": "San Francisco", "PostalCode": "94105" },
    { "ShipGeoID": 8, "Country": "USA", "StateRegion": "NY", "City": "New York", "PostalCode": "10001" }
  ],
  "SalesTerritoryGeo": [
    { "STGeoID": 9, "Country": "USA", "StateRegion": "CA", "City": "San Francisco", "PostalCode": "94105" },
    { "STGeoID": 10, "Country": "USA", "StateRegion": "NY", "City": "New York", "PostalCode": "10001" }
  ],
  "GeoLE": [
    { "GeoID": 1, "LEGeoID": 3 },
    { "GeoID": 2, "LEGeoID": 4 }
  ],
  "GeoTax": [
    { "GeoID": 1, "TaxGeoID": 5 },
    { "GeoID": 2, "TaxGeoID": 6 }
  ],
  "GeoShip": [
    { "GeoID": 1, "ShipGeoID": 7 },
    { "GeoID": 2, "ShipGeoID": 8 }
  ],
  "GeoSales": [
    { "GeoID": 1, "STGeoID": 9 },
    { "GeoID": 2, "STGeoID": 10 }
  ]
}
```

**Flow:**

1. Geography
2. LegalEntityGeo
3. TaxGeo
4. ShippingGeo
5. SalesTerritoryGeo

