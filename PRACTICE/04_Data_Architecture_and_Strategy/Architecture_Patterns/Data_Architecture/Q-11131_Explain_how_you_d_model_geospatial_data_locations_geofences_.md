---
type: "practice"
difficulty: "Intermediate"
---

# Explain how you'd model geospatial data (locations, geofences) across your lake and warehouse for a delivery product, and where forcing it into a generic columnar store falls apart.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Vector Data in the Lakehouse](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Vector_Data_in_the_Lakehouse.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To effectively manage geospatial data in a data architecture for a delivery product, we need to balance performance, flexibility, and adherence to best practices. Geospatial data involves locations (points), polygons (geofences), and potentially complex spatial operations, which can be challenging when using generic columnar storage systems like traditional SQL warehouses. Let's break down the solution step by step.

First, we should understand that geospatial data requires specific handling due to its multidimensional nature and the need for efficient querying of spatial relationships. In a data architecture, this involves separating concerns between the lake (raw data) and the warehouse (processed data).

#### Triaging the Question

The question is essentially asking how to handle geospatial data in a way that maximizes performance and flexibility while minimizing complexity. The key is understanding where generic columnar stores fall short for spatial operations.

#### Assumptions
Delivery Product: We have an application focused on deliveries, which requires real-time location tracking and complex geofencing.
Data Layers:
Lake (Raw Data): Store raw data in its original format to preserve the integrity of the source data.
Warehouse (Processed Data): Process and optimize data for querying performance.

#### First-Principles Solution
Lake Layer

In the lake, we should store geospatial data as close to its original form as possible. This can be in a variety of formats such as JSON or a binary format that can capture complex geometries like polygons (geofences). For example:

JSON
{
  "id": "12345",
  "location": {"type": "Point", "coordinates": [-73.935242, 40.730610]},
  "geofences": [
    {
      "type": "Polygon",
      "coordinates": [[-73.938, 40.732], [-73.937, 40.732], [-73.937, 40.731], [-73.938, 40.731]]
    }
  ]
}
Warehouse Layer

For the warehouse layer, we need to transform and optimize this data for querying and analytics. We can use a relational database (e.g., PostgreSQL with PostGIS) or a spatially optimized columnar store like GeoMesa.

Relational Database: If using PostgreSQL with PostGIS:

SQL
CREATE TABLE delivery_locations (
    id SERIAL PRIMARY KEY,
    location geometry(Point, 4326),
    geofences geography(GeometryCollection, 4326)
);

Spatially Optimized Columnar Store: If using GeoMesa with Hadoop or Spark:

SQL
CREATE TABLE delivery_locations (
    id INT,
    location ST_Point,
    geofences ARRAY<ST_Polygon>
);
Query Optimization

When querying for locations within a geofence, we can use spatial indexes and specialized functions provided by the database. For example, in PostgreSQL:

SQL
SELECT * FROM delivery_locations WHERE ST_Intersects(location, 'POLYGON((-73.938 40.732, -73.937 40.732, -73.937 40.731, -73.938 40.731))');

#### Closing Summary

By separating the raw geospatial data in the lake and processing it in a spatially optimized warehouse, we can ensure that both storage and querying are efficient and flexible. The key is to leverage specialized databases or tools for handling spatial data.

#### Common Mistakes/Follow-Ups
Generic Columnar Store Pitfall: Using generic column stores like Parquet without spatial optimizations will result in suboptimal performance when performing complex spatial queries.
Indexing Strategy: Ensure that proper spatial indexes are created to speed up query times.
Hybrid Approach: Consider a hybrid approach where the lake contains raw data and metadata, while the warehouse processes this for specific use cases.

**Comparison:**

```
id	location	geofences
1	POINT(-73.935242 40.7306)	POLYGON((-73.938 40.732, -73.937 40.732, -73.937 40.731, -73.938 40.731))
```

**Code:**

```sql
-- PostgreSQL with PostGIS Example
CREATE TABLE delivery_locations (
    id SERIAL PRIMARY KEY,
    location geometry(Point, 4326),
    geofences geography(GeometryCollection, 4326)
);

-- Insert Data
INSERT INTO delivery_locations (location, geofences) VALUES
(ST_GeomFromText('POINT(-73.935242 40.730610)', 4326),
 ST_GeogFromText('POLYGON((-73.938 40.732, -73.937 40.732, -73.937 40.731, -73.938 40.731))'));
```

**Example input / output:**

```text
-- Query to check if a location is within a geofence
SELECT * FROM delivery_locations WHERE ST_Intersects(location, 'POLYGON((-73.938 40.732, -73.937 40.732, -73.937 40.731, -73.938 40.731))');
```

**Data:**

```json
{
  "id": "12345",
  "location": {"type": "Point", "coordinates": [-73.935242, 40.730610]},
  "geofences": [
    {
      "type": "Polygon",
      "coordinates": [[-73.938, 40.732], [-73.937, 40.732], [-73.937, 40.731], [-73.938, 40.731]]
    }
  ]
}
```

**Flow:**

1. No
2. Yes
3. Raw Data Lake
4. Is the data structured?
5. Lake Storage (JSON/Parquet)
6. Warehouse Storage (PostGIS,GeoMesa)
7. Transform & Query OptimizedWarehouse

