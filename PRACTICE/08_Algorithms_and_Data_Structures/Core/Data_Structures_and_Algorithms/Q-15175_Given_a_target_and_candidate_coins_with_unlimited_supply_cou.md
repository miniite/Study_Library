---
type: "practice"
difficulty: "Intermediate"
---

# Given a target and candidate coins with unlimited supply, count the number of ways to make the amount.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Dynamic Programming](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Dynamic_Programming.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to find the number of distinct combinations of given coin denominations that sum up to a specific target amount. This is a classic example where dynamic programming (DP) can be effectively applied. The core idea here is to build up solutions for smaller subproblems and use them to construct the solution for larger problems.

#### Triage

The problem asks us to count the number of ways to make change using given coin denominations, which implies we need a combinatorial approach. We are essentially looking for combinations rather than permutations since the order of coins does not matter (e.g., 2 + 1 and 1 + 2 are considered the same combination).

#### First-Principles Solution

We can use dynamic programming to solve this problem efficiently. The idea is to maintain an array dp where dp[i] represents the number of ways to make change for amount i. Initially, we set dp[0] = 1, as there's exactly one way to make a target sum of zero (using no coins).

For each coin denomination, we update the dp array by iterating over all possible amounts up to the target and adding the number of ways to form that amount using the current coin.

Step-by-Step Solution
Initialize DP Array: Create an array dp of size (target + 1) initialized with zeros.
Base Case: Set dp[0] = 1.
Iterate Over Coins: For each coin denomination, iterate over the amounts from that coin value up to the target.
Update DP Array: For each amount i, update dp[i] by adding the number of ways to make change for i - current_coin.
Full Solution

Here is a Python implementation:

PYTHON
def count_combinations(target, coins):
    # Initialize dp array with zeros
    dp = [0] * (target + 1)
    dp[0] = 1  # Base case: one way to make change for zero

    # Iterate over each coin denomination
    for coin in coins:
        # Update the dp array for all amounts from coin value up to target
        for i in range(coin, target + 1):
            dp[i] += dp[i - coin]

    return dp[target]

#### Closing Summary
and Common Mistakes/Follow-Ups

The solution uses dynamic programming to efficiently compute the number of ways to make change. The time complexity is O(n * m), where n is the target amount and m is the number of coins, which is manageable for reasonably sized inputs.

#### Common mistakes
include:

Not properly initializing the DP array.
Forgetting to add the base case dp[0] = 1.
Misunderstanding that order does not matter (e.g., using combinations rather than permutations).

#### Follow-up
questions might involve optimizing the solution for larger target amounts or coin sets, considering different constraints, or exploring alternative approaches like backtracking.

**Code:**

```python
def count_combinations(target, coins):
    dp = [0] * (target + 1)
    dp[0] = 1
    for coin in coins:
        for i in range(coin, target + 1):
            dp[i] += dp[i - coin]
    return dp[target]
```

**Example input / output:**

```text
# Example input and output
target = 5
coins = [1, 2, 5]
print(count_combinations(target, coins))  # Output: 4 (1+1+1+1+1, 1+1+1+2, 1+2+2, 5)
```

**Flow:**

1. Start
2. Initialize dp array
3. Set dp[0] = 1
4. Iterate over each coindenomination
5. For each coin, iterate overamounts from that coin valueto target
6. Update dp[i] += dp[i -current_coin]
7. Return dp[target]
8. End

