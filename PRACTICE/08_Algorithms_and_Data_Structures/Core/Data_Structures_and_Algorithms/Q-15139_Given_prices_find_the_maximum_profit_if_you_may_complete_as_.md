---
type: "practice"
difficulty: "Intermediate"
---

# Given prices, find the maximum profit if you may complete as many buy-sell transactions as you like.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Stock Profit Problems](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Stock_Profit_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To solve this problem, we need to identify all the local minima and maxima in the array of prices. By buying at each local minimum and selling at the next local maximum, we can maximize our profit. This greedy approach works because every transaction increases the overall profit.

#### Triaging the Question

The question is asking us to determine the maximum profit from multiple transactions with an unlimited number of buy-sell operations allowed. The interviewer might be probing your understanding of algorithmic thinking and the ability to work through a problem that involves multiple decisions at different points in time. We need to assume that we can make as many transactions as desired, but each transaction must consist of one buy followed by one sell.

#### First-Principles Solution
Understanding the Problem: The key insight here is that we should capture every upward movement in price because each such move increases our profit.
Greedy Algorithm: At any point where a local minimum (a price at which the next price is higher) meets a local maximum, we can make a transaction and add the difference to our profit.

#### Implementation
We iterate through the array of prices, keeping track of the current minimum price. Whenever we encounter a price that is lower than the previous one, it indicates a potential buying opportunity. The next rising edge represents a selling opportunity.

Here’s how you can implement this in code:

PYTHON
def maxProfit(prices):
    if not prices:
        return 0

    profit = 0
    min_price = float('inf')

    for price in prices:
        # Update the minimum price seen so far.
        if price < min_price:
            min_price = price

        # If we find a local maximum, sell and add to profit.
        elif price > min_price:
            profit += (price - min_price)
            min_price = price  # Reset min_price for potential new transactions

    return profit

#### Closing Summary
+ Common Mistakes/Follow-Ups

The solution is efficient with a time complexity of (O(n)) and space complexity of (O(1)). It ensures we capture every upward trend in the prices. A common mistake could be to miss updating the min_price or incorrectly managing transaction boundaries.

**Code:**

```python
def maxProfit(prices):
    if not prices:
        return 0

    profit = 0
    min_price = float('inf')

    for price in prices:
        if price < min_price:
            min_price = price

        elif price > min_price:
            profit += (price - min_price)
            min_price = price

    return profit
Self-Check

The code is correct and covers all edge cases, including an empty list of prices.
```

