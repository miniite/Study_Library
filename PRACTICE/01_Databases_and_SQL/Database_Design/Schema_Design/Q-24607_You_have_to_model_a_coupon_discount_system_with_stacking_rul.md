---
type: "practice"
difficulty: "Deep"
---

# You have to model a coupon/discount system with stacking rules and eligibility; how do you keep the relationship complexity from exploding the schema?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Modeling Relationships and Schemas](<../../../../01_Databases_and_SQL/Database_Design/Schema_Design/Modeling_Relationships_and_Schemas.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an efficient and maintainable schema for a coupon/discount system that includes stacking rules and eligibility, it's crucial to manage complexity carefully. ER diagrams help visualize relationships between entities, but in this case, we need to ensure the schema isn't overly complex. We'll start by identifying the key entities: Coupon, Discount, Customer, Order, and Product. The core challenge is managing stacking rules and ensuring that discounts are applied correctly without causing a combinatorial explosion.

First, let's define the basic entities:

Coupon: A discount offer with conditions.
Discount: A specific application of a coupon to an order or product.
Customer: A user who can apply coupons.
Order: An instance where products are purchased by a customer.
Product: Items that can be bought.

**Code:**

```
MERMAID
Copy
erDiagram
    Customer ||--o{ Order : places
    Order }|--|{ Discount : applies
    Product ||--o{ Order : contains
    Coupon ||--o{ Discount : defines
```

