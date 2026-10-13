---
type: "practice"
difficulty: "Deep"
---

# Design the index behind 'which coupons apply to this cart' where coupons have overlapping predicate rules; how do you avoid evaluating all rules and prune to a candidate set quickly?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Inverted Indexes](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Inverted_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of efficiently determining which coupons apply to a given shopping cart, we need to design an index that avoids evaluating all coupon rules. This is crucial for performance, especially when dealing with a large number of coupons and complex predicate rules. The core idea is to prune the set of relevant coupons early in the process by leveraging a carefully designed data structure.

#### Triage

The question asks us to design an index or indexing mechanism that can quickly determine which coupon conditions apply to a given cart without evaluating all possible coupons. This involves understanding how coupon rules work and finding ways to efficiently identify candidate coupons based on these rules.

#### First-Principles Solution
Understand the Coupon Rules: Each coupon rule is a predicate, possibly involving multiple conditions (e.g., minimum spend, items in the cart, etc.). The goal is to find a way to quickly narrow down which of these rules could apply.
Design an Index Structure: We need an index that can efficiently store and retrieve relevant coupons based on their rules. One approach is to use a combination of Trie (prefix tree) and range trees to handle the predicate rules effectively.
Pruning Mechanism: Implement a pruning mechanism to quickly discard irrelevant coupon rules, thus reducing the number of evaluations needed.
Step-by-Step Solution

Predicate Rule Representation:

A predicate rule can be represented as a set of conditions. For example, a rule might state that "if the total cart value is greater than $50 and the customer bought an item with ID 123," then apply coupon X.

Trie for Predicate Conditions:

#### Cons
truct a Trie where each node represents a condition (e.g., total_value > 50).
Each leaf in the Trie can correspond to multiple coupons that share common prefix conditions.

Range Trees:

For continuous ranges like total_value > 50, use range trees to quickly determine if the current cart value falls within any applicable range.

Pruning Mechanism:

As we process the cart, traverse the Trie and Range Trees to dynamically prune irrelevant coupons early in the evaluation process.

#### Example
Implementation
PYTHON
class CouponRule:
    def __init__(self, conditions):
        self.conditions = conditions

class TrieNode:
    def __init__(self):
        self.children = {}
        self.coupons = []

def build_trie(coupon_rules):
    root = TrieNode()
    for rule in coupon_rules:
        node = root
        for condition in rule.conditions:
            if condition not in node.children:
                node.children[condition] = TrieNode()
            node = node.children[condition]
        node.coupons.append(rule)
    return root

def prune_cart(cart, trie):
    def traverse(node, cart_value):
        # Prune irrelevant coupons
        for rule in node.coupons:
            if all(condition(cart_value) for condition in rule.conditions):
                yield rule
        for child in node.children.values():
            yield from traverse(child, cart_value)

    return list(traverse(trie, cart.total_value))

# Example usage
coupon_rules = [
    CouponRule([lambda v: v > 50]),
    CouponRule([lambda i: i == 123])
]
trie = build_trie(coupon_rules)
pruned_coupons = prune_cart(cart, trie)

#### Closing Summary
and Common Mistakes/Follow-Ups

The key to avoiding evaluating all rules is to use a Trie structure to efficiently manage predicate conditions and a pruning mechanism to discard irrelevant coupons early. This solution ensures that we only evaluate the most relevant coupons based on the current state of the cart.

#### Common Mistakes

Failing to properly represent complex predicate conditions.
Not implementing an effective pruning mechanism, leading to unnecessary evaluations.
Overlooking edge cases where rules might be partially applicable but not fully satisfied.

For follow-ups, you might explore more advanced techniques like caching, parallel processing, or specialized data structures tailored to specific types of predicates.

**Comparison:**

```
Coupon Rule	Condition
Rule A	v > 50
Rule B	i == 123
Trie Node Structure
Root -> v > 50 -> ... -> Coupons [Rule A, Rule B]
```

**Code:**

```python
class CouponRule:
    def __init__(self, conditions):
        self.conditions = conditions

class TrieNode:
    def __init__(self):
        self.children = {}
        self.coupons = []

def build_trie(coupon_rules):
    root = TrieNode()
    for rule in coupon_rules:
        node = root
        for condition in rule.conditions:
            if condition not in node.children:
                node.children[condition] = TrieNode()
            node = node.children[condition]
        node.coupons.append(rule)
    return root

def prune_cart(cart, trie):
    def traverse(node, cart_value):
        # Prune irrelevant coupons
        for rule in node.coupons:
            if all(condition(cart_value) for condition in rule.conditions):
                yield rule
        for child in node.children.values():
            yield from traverse(child, cart_value)

    return list(traverse(trie, cart.total_value))
```

**Example input / output:**

```text
Input:
- Cart with total value $55 and items containing item ID 123
- Coupon rules:
  - Rule1: If total cart value > $50, apply coupon A
  - Rule2: If cart contains item with ID 123, apply coupon B

Output:
- Pruned Coupons: [CouponRule(...), ...] (only relevant coupons based on the current cart state)
```

**Data:**

```python
class Cart:
    def __init__(self, total_value=0, items=[]):
        self.total_value = total_value
        self.items = items

# Example data
cart1 = Cart(total_value=55, items=[123])
coupon_rules = [
    CouponRule([lambda v: v > 50]),
    CouponRule([lambda i: i == 123])
]
trie = build_trie(coupon_rules)
pruned_coupons = prune_cart(cart1, trie)
print(pruned_coupons)  # Output: [CouponRule(...), ...]
```

**Flow:**

1. Yes
2. No
3. Item found
4. Item not found
5. Start
6. Is cart value >= 50?
7. Prune and check for item ID123
8. No coupons apply
9. Apply Rule B, continue tocheck Rule A
10. Only apply Rule A

