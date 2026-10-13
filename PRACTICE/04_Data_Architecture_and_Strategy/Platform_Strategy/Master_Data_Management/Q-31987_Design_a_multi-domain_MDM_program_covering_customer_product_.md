---
type: "practice"
difficulty: "Expert"
---

# Design a multi-domain MDM program covering customer, product, supplier, and location — how do you sequence domains, and why is customer usually not the one you should start with?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Multi-domain MDM Programs](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Multi-domain_MDM_Programs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Master Data Management (MDM) involves creating a central repository of master data that is consistent, accurate, and reliable across an organization's various systems and processes. When designing a multi-domain MDM program covering customer, product, supplier, and location domains, the sequencing of these domains is crucial to ensure efficient and effective data management.

The key idea here is that while it might be tempting to start with the customer domain due to its centrality in many business operations, practical considerations often make this approach suboptimal. In this answer, we'll explore why a different sequence might be more advantageous and provide a detailed solution for sequencing these domains from first principles.

#### Triage

The question is asking about the optimal sequence for integrating multiple master data domains into an MDM program, with a specific focus on why customer data is typically not the starting point. The interviewer is probing your understanding of domain interdependencies, strategic planning in MDM projects, and the practical implications of different sequences.

#### First-Principles Solution
Understand Domain Interdependencies: Each master data domain interacts with others in complex ways. For example, customer data often depends on product or supplier data for accurate representations.
Strategic Planning Considerations: The sequence should align with business objectives and the organization's operational maturity.
Data Availability and Quality: Assess which domains have more reliable data sources first.
Sequencing Domains

Start with Product Domain:

Reasoning: Products are foundational in many industries, often driving downstream processes like manufacturing or inventory management.
Steps:
Define Core Attributes: Establish the essential attributes of products (e.g., name, category, manufacturer).
Integrate Supplier Data: Link product data to supplier information for better supply chain visibility and procurement decisions.
Create Product Hierarchy: Organize products into categories and subcategories.

Next: Supplier Domain:

Reasoning: Reliable supplier information is critical for quality control, logistics, and cost management.
Steps:
Define Core Attributes: Establish attributes like company name, contact details, and financial health metrics.
Link to Product Data: Ensure that product data references accurate suppliers.
Integrate Supplier Performance Metrics: Track supplier performance over time.

Follow with Customer Domain:

Reasoning: While customer data is central, it often relies on the quality of upstream data (products and suppliers).
Steps:
Define Core Attributes: Establish attributes like name, contact details, transaction history.
Link to Product Data: Ensure that sales and marketing efforts are based on accurate product information.
Integrate Customer Segmentation: Use data from the product and supplier domains for better segmentation.

Finalize with Location Domain:

Reasoning: Locations provide context for operations, logistics, and customer experience.
Steps:
Define Core Attributes: Establish attributes like address, geographic coordinates, and operational roles.
Integrate with Other Domains: Ensure that product and supplier locations are correctly identified.

#### Closing Summary

The sequence of domains in an MDM program should prioritize foundational data domains (e.g., products and suppliers) before moving to more dependent ones like customers. This approach ensures a solid base for downstream processes, reduces integration complexity, and minimizes the risk of circular dependencies.

#### Common Mistakes/Follow-Ups
Starting with Customer Data Too Early: Customers depend on accurate product and supplier information.
Overlooking Interdependencies: Poorly integrated upstream data can lead to cascading errors in MDM.
Neglecting Quality Control: Rushing into customer data without ensuring the quality of other domains can result in suboptimal solutions.
Why Customer Data is Not Usually Started First

Starting with customer data might seem intuitive, but it often leads to issues such as:

Circular Dependencies: Ensuring that product and supplier data are correctly linked before fully addressing customer data.
Data Inconsistencies: Without robust product and supplier data, customer records may be inaccurate or incomplete.

**Comparison:**

```
Product Name	Category	Manufacturer
Laptop	Electronics	SupplierA
Company Name	Contact Details	Financial Health
SupplierA	email: contact@suppliera.com	85
```

**Code:**

```python
# Example of linking product to supplier in a simplified MDM schema

class Product:
    def __init__(self, name, category, manufacturer):
        self.name = name
        self.category = category
        self.manufacturer = manufacturer

class Supplier:
    def __init__(self, company_name, contact_details, financial_health):
        self.company_name = company_name
        self.contact_details = contact_details
        self.financial_health = financial_health

# Example data
product1 = Product("Laptop", "Electronics", "SupplierA")
supplier1 = Supplier("SupplierA", {"email": "contact@suppliera.com"}, 85)
```

**Example input / output:**

```text
Input:
- Product data with manufacturer names
- Supplier data

Output:
- Products linked to their respective suppliers
- Supplier performance metrics integrated into product records
```

**Data:**

```json
{
    "products": [
        {
            "name": "Laptop",
            "category": "Electronics",
            "manufacturer": "SupplierA"
        }
    ],
    "suppliers": [
        {
            "company_name": "SupplierA",
            "contact_details": {"email": "contact@suppliera.com"},
            "financial_health": 85
        }
    ]
}
```

**Flow:**

1. Start with Product Domain
2. Integrate Supplier Data
3. Create Product Hierarchy
4. Link to Other Domains
5. Next: Supplier Domain
6. Define Core Attributes
7. Integrate Supplier PerformanceMetrics
8. Link to Product Data
9. Follow with Customer Domain
10. Define Core Attributes
11. Link to Product Data
12. Integrate CustomerSegmentation
13. Finalize with Location Domain
14. Define Core Attributes
15. Integrate with Other Domains

