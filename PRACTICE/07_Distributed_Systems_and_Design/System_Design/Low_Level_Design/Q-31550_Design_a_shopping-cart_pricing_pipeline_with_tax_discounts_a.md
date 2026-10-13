---
type: "practice"
difficulty: "Intermediate"
---

# Design a shopping-cart pricing pipeline with tax discounts and shipping. In what order do they apply?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Rule Engines and Pricing](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Rule_Engines_and_Pricing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

In designing a shopping cart pricing pipeline that includes tax, discounts, and shipping, we need to ensure that these operations are applied in a specific order to avoid double charging or incorrect calculations. The pipeline should apply discounts first, then taxes, followed by calculating the final price and applying any shipping costs.

#### Triaging the Question

The question is asking for the correct sequence of operations when implementing a pricing pipeline for an e-commerce system. Specifically, we need to determine the order in which discounting, tax calculations, and shipping should be applied to ensure accurate final prices. The interviewer might be probing your understanding of the dependencies between these operations and how they interact.

#### First-Principles Solution
Discount Application: Apply all applicable discounts to the item prices first.
Tax Calculation: Calculate taxes on the discounted item prices.
Shipping Cost: Add shipping cost based on the total price after applying discounts and taxes.
Detailed Reasoning

Discounts First:

Discounts should be applied first because they reduce the base price of items in the cart. Applying discounts before taxes ensures that taxes are calculated on the correct item prices.

Taxes Next:

After all discounts have been applied, calculate the taxes. This step is crucial as it affects the final price after applying any quantity-based or item-specific discounts.

Shipping Cost Last:

Finally, add shipping cost to the total post-tax amount. Shipping costs are typically a flat rate or based on the total weight/quantity of items and should be added last because they do not depend on individual discount application but rather on the final subtotal.

#### Example
Worked Input-Output

Assume we have the following input data:

JSON
{
  "cart": [
    { "itemId": 1, "price": 50.00, "quantity": 2 },
    { "itemId": 2, "price": 75.00, "quantity": 1 }
  ],
  "discounts": [
    { "id": 1, "type": "percentage", "value": 10 }, // 10% off
    { "id": 2, "type": "fixedAmount", "value": 5 }  // $5 off
  ],
  "taxRate": 8.0, // 8% tax
  "shippingCost": 5.0
}
Step-by-Step Calculation

Apply Discounts:

Item 1: (2 \times (50 - 10%) = 90)
Item 2: (75 - 5 = 70)

Calculate Total Discounted Price:

Total after discounts: (90 + 70 = 160)

Apply Tax:

Total after tax: (160 \times (1 + \frac{8}{100}) = 172.8)

Add Shipping Cost:

Final total price: (172.8 + 5 = 177.8)

#### Closing Summary

The correct order for applying operations in the shopping cart pricing pipeline is discounts first, followed by taxes, and finally adding shipping costs. This ensures that all calculations are based on the most accurate subtotal.

#### Common Mistakes
and Follow-Ups
Incorrect Order: Applying shipping cost before tax or discount can lead to incorrect final prices.
Omitting Tax Calculation: Not applying taxes after discounts can result in an incomplete pricing pipeline.
Double Charging: If both discount and tax apply, double-check that the discount is applied first so it reduces the base price correctly.

If asked further, you could discuss how to handle different types of discounts (percentage vs. fixed amount) or more complex scenarios such as tiered discounts.

**Comparison:**

| Item ID | Price (USD) | Quantity | Total (USD) |
|---------|-------------|----------|-------------|
| 1       | 50.00       | 2        | 90.00       |
| 2       | 75.00       | 1        | 70.00       |
| Total   |             |          | 160.00      |
| Discounted Total (USD) | Tax Rate (%) | Tax Amount (USD) | Total after Tax (USD) | Shipping Cost (USD) | Final Price (USD) |
|------------------------|--------------|------------------|-----------------------|--------------------|-------------------|
| 160.00                 | 8            | 12.80            | 172.80                | 5                  | 177.80            |

**Code:**

```python
def calculate_total(cart_items, discounts, tax_rate, shipping_cost):
    # Apply discounts
    total_price = sum(item['price'] * item['quantity'] for item in cart_items)
    for discount in discounts:
        if discount['type'] == 'percentage':
            total_price *= (1 - discount['value'])
        elif discount['type'] == 'fixedAmount':
            total_price -= discount['value']

    # Apply tax
    total_price += total_price * (tax_rate / 100)

    # Add shipping cost
    total_price += shipping_cost

    return round(total_price, 2)

cart_items = [
    {'itemId': 1, 'price': 50.00, 'quantity': 2},
    {'itemId': 2, 'price': 75.00, 'quantity': 1}
]
discounts = [{'id': 1, 'type': 'percentage', 'value': 10}, {'id': 2, 'type': 'fixedAmount', 'value': 5}]
tax_rate = 8
shipping_cost = 5

total_price = calculate_total(cart_items, discounts, tax_rate, shipping_cost)
print(total_price)  # Output: 177.80
```

**Example input / output:**

```text
cart_items = [
    {'itemId': 1, 'price': 50.00, 'quantity': 2},
    {'itemId': 2, 'price': 75.00, 'quantity': 1}
]
discounts = [{'id': 1, 'type': 'percentage', 'value': 10}, {'id': 2, 'type': 'fixedAmount', 'value': 5}]
tax_rate = 8
shipping_cost = 5

total_price = calculate_total(cart_items, discounts, tax_rate, shipping_cost)
print(total_price)  # Output: 177.80
```

**Data:**

```json
{
  "cart": [
    { "itemId": 1, "price": 50.00, "quantity": 2 },
    { "itemId": 2, "price": 75.00, "quantity": 1 }
  ],
  "discounts": [
    { "id": 1, "type": "percentage", "value": 10 }, // 10% off
    { "id": 2, "type": "fixedAmount", "value": 5 }  // $5 off
  ],
  "taxRate": 8.0, // 8% tax
  "shippingCost": 5.0
}
```

**Flow:**

1. Taxes
2. Discounts
3. Cart
4. Discounts
5. Tax
6. Total Price
7. Shipping Cost
8. Final Price
9. Item 1
10. Item 2
11. Discount 1 (Percentage)
12. Discount 2 (Fixed Amount)
13. Tax Rate
14. Tax Calculation

