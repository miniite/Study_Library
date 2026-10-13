---
type: "practice"
difficulty: "Intermediate"
---

# Design a system to merge a guest cart with the user's cart after they log in.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Multi-tenant Migrations](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Multi-tenant_Migrations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a system that merges a guest cart with the user's cart after logging in, we need to focus on several key aspects: ensuring data integrity, optimizing performance, and maintaining a scalable architecture. The core idea is to implement a seamless transition from a temporary guest session to a persistent user session while preserving all items and metadata.

The question is essentially asking how we can efficiently merge two carts (guest and user) upon login, considering potential complexities such as race conditions, data consistency, and performance optimization. We'll make some key assumptions: that the cart contains product IDs, quantities, prices, and any associated metadata like timestamps or user preferences.

First-principles reasoning will guide us to break down this problem into manageable components:

Authentication: Verify the user's credentials.
Cart Retrieval: Fetch both the guest cart (if applicable) and the user’s existing cart from a database.
Merge Logic: Combine the items from both carts, resolving conflicts if necessary (e.g., same product ID but different quantities).
Update Persistently: Save the merged cart back to storage.

Let's walk through each step in detail:

Authentication:

We need a robust authentication system that can handle various login methods like email/password, social logins, or API keys.
Ensure that user sessions are managed securely using techniques such as JWT (JSON Web Tokens) with proper validation and expiry.

Cart Retrieval:

For the guest cart, we store it temporarily in a session object or a cookie until a user logs in.
The user's existing cart is stored persistently in a database with unique identifiers for each user session.

Merge Logic:

We need to check if there are any duplicate product IDs between the two carts.
If duplicates exist, we decide how to handle them (e.g., summing quantities or retaining one version only).
Metadata like timestamps and preferences can also be merged appropriately.

Update Persistently:

After merging, the updated cart should be saved back to the database in a single transaction to ensure data consistency.
Use optimistic locking mechanisms if multiple users might access the same cart concurrently.

Here’s a high-level flow of our system:

User logs in with credentials.
Authentication service validates user and retrieves their session ID or unique identifier.
System checks for an associated guest cart and user cart using the session ID.
Both carts are fetched from the database.
Merge logic processes both carts, resolving any conflicts.
Merged cart is saved back to the database in a single transaction.

Closing this summary, common mistakes often include not considering race conditions between multiple users accessing the same cart simultaneously or failing to handle metadata and product details effectively during the merge process.

**Comparison:**

```
Product ID	Quantity in Guest Cart	Quantity in User Cart	Merged Quantity
101	2	None	2
102	None	5	5
103	1	None	1
```

**Code:**

```python
# Pseudocode for merging carts

def merge_guest_and_user_cart(guest_cart_items, user_cart_id):
    # Fetch user's existing cart
    user_cart = db.get_cart(user_cart_id)

    # Merge items from guest cart into user cart
    merged_items = {}
    for item in guest_cart_items:
        product_id = item['product_id']
        if product_id in user_cart['items']:
            # Handle duplicate (e.g., sum quantities)
            user_cart['items'][product_id]['quantity'] += item['quantity']
        else:
            # Add new item
            merged_items[product_id] = item

    # Update the user's cart with merged items
    user_cart['items'].update(merged_items)

    # Save updated cart to database
    db.update_cart(user_cart, user_cart_id)
```

**Example input / output:**

```text
# Input from guest cart
guest_cart_items = [
    {"product_id": 101, "quantity": 2},
    {"product_id": 103, "quantity": 1}
]

# User's existing cart ID and items
user_cart_id = "uX123456789"
user_cart = db.get_cart(user_cart_id)

# Merging the carts
merged_cart_items = merge_guest_and_user_cart(guest_cart_items, user_cart_id)
print(merged_cart_items)  # Output: { ... }
```

**Data:**

```json
{
  "guest_cart": [
    {"product_id": 101, "quantity": 2},
    {"product_id": 103, "quantity": 1}
  ],
  "user_cart": {
    "items": {
      "product_id": [102, 5],
      "timestamp": ["2023-10-04T12:00:00Z", 1696488000]
    }
  }
}
```

**Flow:**

1. User Logs In
2. Authenticate User
3. Fetch Guest Cart (if exists)
4. Fetch User's Existing Cart
5. Merge Carts
6. Save Merged Cart

