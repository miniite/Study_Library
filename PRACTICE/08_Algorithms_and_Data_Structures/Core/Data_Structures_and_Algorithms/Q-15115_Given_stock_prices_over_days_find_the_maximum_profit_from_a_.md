---
type: "practice"
difficulty: "Intermediate"
---

# Given stock prices over days, find the maximum profit from a single buy-and-sell.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Stock Profit Problems](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Stock_Profit_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's really being asked: Given an array prices where prices[i] is the stock price on day i, find the maximum profit from exactly one buy and one later sell. The constraint that makes it non-trivial: you must buy before you sell — the buy day's index must be strictly less than the sell day's index. If no profitable trade exists, the answer is 0 (you simply don't trade). This is the classic "Best Time to Buy and Sell Stock I" (single transaction). Level is intermediate experience, so the interviewer expects the optimal O(n) one-pass solution, a clear explanation of why it works, and correct edge-case handling — not just the brute force.

What they're probing: Do I jump to the O(n²) "check every pair" or do I recognise this as a running-minimum problem? Can I articulate the greedy/DP invariant, handle the "prices only fall" case correctly (return 0), and avoid the classic bug of computing max - min globally (which can put the min after the max)?

Clarifying questions I'd ask: (1) Single transaction only, right? (Yes — that's this variant; the multi-transaction / cooldown / k-transaction versions are different problems.) (2) Can I short-sell (sell before buy)? (No.) (3) If no profit is possible, return 0? (Yes.) (4) Empty or single-element array → 0.

From first principles

The naive idea is: try every pair of days (i, j) with i < j, compute prices[j] - prices[i], take the max. That's O(n²) and correct but wasteful.

The key insight: to maximise profit when selling on day j, I want to have bought on the day with the lowest price seen so far, on or before day j. So as I scan left to right, if I track the minimum price seen up to the current day, then the best profit achievable by selling today is today's price − min-so-far. The overall answer is the maximum of that quantity across all days. Crucially, min-so-far only considers days at or before the current day, which automatically enforces the buy-before-sell constraint — I can never accidentally buy after I sell.

This is a one-pass greedy (equivalently a tiny DP where the state is "cheapest buy price so far"). It's O(n) time and O(1) space.

Why the "global max − global min" shortcut is WRONG

A tempting one-liner is max(prices) − min(prices). This fails when the minimum occurs after the maximum. Example: [9, 1, 2] — global max is 9 (day 0), global min is 1 (day 1), giving 9 − 1 = 8, but you cannot buy at 1 (day 1) and sell at 9 (day 0) because that's selling in the past. The correct answer is 2 − 1 = 1. The running-minimum approach avoids this because it only ever compares today's price against minima from earlier days.

The algorithm, step by step
Initialise minPrice = prices[0] (cheapest buy seen so far) and maxProfit = 0.
For each day from index 1 onward:
maxProfit = max(maxProfit, prices[i] − minPrice) — best profit if we sell today.
minPrice = min(minPrice, prices[i]) — update cheapest buy for future days.
Return maxProfit.

Order matters within the loop: compute the sell profit using the old minPrice (a buy strictly before today), then update minPrice. If you update minPrice first you could "buy and sell on the same day" for a spurious profit of 0 — harmless here since profit would be 0, but conceptually you should sell using a prior minimum. Both orderings give the same correct maximum in this problem, but I prefer computing profit before updating for clarity.

Edge cases
Empty array or length 0/1 → return 0 (no transaction possible).
Strictly decreasing prices e.g. [7, 6, 4, 3, 1] → maxProfit never goes positive, return 0 (don't trade — don't return a negative).
All equal prices → 0.
Prices at the boundary — the global min on the last day contributes nothing (nothing to sell to), and that's fine.
DP framing (for the interviewer who wants it)

Define minPrice[i] = min of prices[0..i]. Then answer = max over i of (prices[i] − minPrice[i-1]). This is a 1-D DP that collapses to two scalars, which is exactly the one-pass solution. The multi-transaction variants (Buy/Sell II, with cooldown, at most k) extend this into a genuine 2-state DP (hold / cash), which is a good follow-up to mention to show depth.

Complexity
Time: O(n) — single pass.
Space: O(1) — two scalars.
Beats brute force O(n²) and needs no extra data structures.
What a strong answer emphasises

Recognise it as a running-minimum / one-pass problem, not a nested-loop pair search. State the invariant crisply: "the best profit selling today is today's price minus the cheapest price on any earlier day." Handle the return-0-when-no-profit contract and the min-after-max trap explicitly. Deliver O(n)/O(1) code. Common follow-ups: "what if you can make multiple transactions?" (sum all positive consecutive differences — greedy), "at most k transactions?" (DP), "add a transaction fee or cooldown?" (state-machine DP). Being able to pivot to those state-machine DPs signals you understand the whole family.

**Comparison:**

```
Approach	Time	Space	Notes
Brute force (all pairs i<j)	O(n²)	O(1)	Correct but slow
Running minimum (one pass)	O(n)	O(1)	Optimal; enforces buy-before-sell automatically
max(prices) - min(prices)	O(n)	O(1)	WRONG — breaks when min comes after max
Variant	Approach
Single transaction (this Q)	Running minimum, O(n)/O(1)
Unlimited transactions	Sum positive consecutive diffs, O(n)
At most k transactions	2-D DP over (day, transactions, hold/cash)
With cooldown / fee	State-machine DP (hold/cash/cooldown states)
```

**Code:**

```

// Best Time to Buy and Sell Stock I — single transaction. O(n) time, O(1) space. class Solution { public int maxProfit(int[] prices) { if (prices == null || prices.length < 2) return 0; int minPrice = prices[0]; // cheapest buy seen so far int maxProfit = 0; // never negative: 0 means "don't trade" for (int i = 1; i < prices.length; i++) { // best profit if we SELL today (buy was on some earlier, cheaper day) maxProfit = Math.max(maxProfit, prices[i] - minPrice); // update cheapest buy for future days minPrice = Math.min(minPrice, prices[i]); } return maxProfit; } }

CODE
Copy

```python
# Python equivalent
def max_profit(prices: list[int]) -> int:
    if len(prices) < 2:
        return 0
    min_price = prices[0]
    max_profit = 0
    for p in prices[1:]:
        max_profit = max(max_profit, p - min_price)
        min_price = min(min_price, p)
    return max_profit
JAVA
Copy
// Follow-up: UNLIMITED transactions (Buy/Sell II) — sum every positive up-move.
int maxProfitMany(int[] prices) {
    int profit = 0;
    for (int i = 1; i < prices.length; i++)
        if (prices[i] > prices[i - 1]) profit += prices[i] - prices[i - 1];
    return profit;
}
```

**Example input / output:**

```

Worked examples:

CODE
Copy
Input : [7, 1, 5, 3, 6, 4]   Output : 5   (buy at 1 on day 1, sell at 6 on day 4)
Input : [7, 6, 4, 3, 1]      Output : 0   (prices only fall -> don't trade)
Input : [9, 1, 2]            Output : 1   (buy 1, sell 2; NOT 9-1=8, since 9 is before 1)
Input : [2, 4, 1]            Output : 2   (buy 2, sell 4; the later 1 is useless)
Input : []                   Output : 0

Dry run on [7, 1, 5, 3, 6, 4]:

CODE
Copy
start: minPrice=7, maxProfit=0
i=1 (1): profit=1-7=-6 -> maxProfit=0 ; minPrice=min(7,1)=1
i=2 (5): profit=5-1=4  -> maxProfit=4 ; minPrice=1
i=3 (3): profit=3-1=2  -> maxProfit=4 ; minPrice=1
i=4 (6): profit=6-1=5  -> maxProfit=5 ; minPrice=1
i=5 (4): profit=4-1=3  -> maxProfit=5 ; minPrice=1
result: 5
```

**Flow:**

1. each i
2. done
3. minPrice = prices[0]; maxProfit= 0
4. i from 1 to n-1
5. maxProfit = max(maxProfit,prices[i] - minPrice)
6. minPrice = min(minPrice,prices[i])
7. return maxProfit (0 if neverprofitable)

