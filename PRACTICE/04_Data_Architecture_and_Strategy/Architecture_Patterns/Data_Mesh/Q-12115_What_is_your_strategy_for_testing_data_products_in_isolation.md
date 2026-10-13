---
type: "practice"
difficulty: "Expert"
---

# What is your strategy for testing data products in isolation and in composition so a mesh does not become impossible to reason about?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Data Product Discoverability](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Mesh/Data_Product_Discoverability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Data Mesh architecture, ensuring that individual data products can be tested in isolation and in composition is crucial. This prevents a complex system from becoming unmanageable and ensures that each component operates independently while still contributing to the overall mesh. Here's my strategy for testing in both scenarios:

Isolation Testing: Each data product should be treated as an independent unit, much like a microservice in a microservices architecture. We can use techniques such as unit tests and integration tests to verify that each product functions correctly on its own.
Composition Testing: Since Data Mesh involves the composition of multiple data products, we need to ensure that these components work seamlessly together. This requires end-to-end testing where the output from one data product is used as input for another.

The core idea here is to maintain modularity and flexibility within the mesh while ensuring robustness through comprehensive testing.

**Comparison:**

```
Data Product	Version	Test Coverage
Customer Data	v1.0	Unit Tests
Sales Data	v2.0	Integration Tests
```

**Code:**

```python
# Example of a unit test for a simple transformation function in Python
import unittest

def transform_data(input_data):
    return [x * 2 for x in input_data]

class TestTransformData(unittest.TestCase):
    def test_transform_data(self):
        self.assertEqual(transform_data([1, 2, 3]), [2, 4, 6])
```

**Example input / output:**

```

Example input:

JSON
Copy
{
  "customer_data": [1, 2, 3],
  "sales_data": [4, 5, 6]
}

Output from transform_data function in isolation:

JSON
Copy
{
  "transformed_customer_data": [2, 4, 6]
}

Example end-to-end composition test scenario:

JSON
Copy
{
  "customer_sales_data": {
    "customer_data": [1, 2, 3],
    "sales_data": [4, 5, 6]
  }
}

Expected output after processing through the data mesh (assuming transform_data is one of several steps):

JSON
Copy
{
  "final_output": [
    {"id": 1, "value": 8},
    {"id": 2, "value": 10},
    {"id": 3, "value": 12}
  ]
}
```

**Data:**

```json
{
  "data_product_1": {
    "id": 1,
    "name": "Customer Data",
    "version": "v1.0"
  },
  "data_product_2": {
    "id": 2,
    "name": "Sales Data",
    "version": "v2.0"
  }
}
```

**Flow:**

1. Data Product 1
2. Unit Tests
3. Pass
4. Deploy
5. Data Product 2
6. Integration Tests
7. Pass
8. Compose with Data Products
9. Compose with Other Products
10. End-to-End Tests
11. Final Output

