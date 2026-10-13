---
type: "practice"
difficulty: "Other"
---

# In-Memory Account Ledger with Guarded Transfers

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Design](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Design a data structure that manages a fixed set of accounts and processes money operations against them. There are n accounts labeled 1 through n. You are given a 0-indexed integer array funds, where funds[i] is the starting amount held by the account labeled i + 1.

Your class must support three operations, each of which returns a boolean indicating whether the operation was valid and therefore applied:

transfer(from, to, amount) — move amount units from the account labeled from to the account labeled to. Valid only if both labels are within [1, n] and the from account currently holds at least amount. Return true if valid (and perform the move), false otherwise.
deposit(acct, amount) — add amount units to the account labeled acct. Valid only if acct is within [1, n]. Return true if valid (and perform the deposit), false otherwise.
withdraw(acct, amount) — remove amount units from the account labeled acct. Valid only if acct is within [1, n] and the account currently holds at least amount. Return true if valid (and perform the withdrawal), false otherwise.

An operation that is not valid must leave every balance unchanged. Because amounts and balances can be large, use a 64-bit integer type for the money values.

Class interface

Ledger(long[] funds) — initialize with the starting balances.
boolean transfer(int from, int to, long amount)
boolean deposit(int acct, long amount)
boolean withdraw(int acct, long amount)

#### Cons
traints

n == funds.length
1 <= n, acct, from, to <= 10^5
0 <= funds[i], amount <= 10^12
At most 10^4 calls will be made to each of transfer, deposit, and withdraw.

#### Example
s & test cases
TEXT
- Input: `Ledger([15, 200, 40, 80])`, then `withdraw(3, 40)` -> Output: `true`
  Why: account 3 holds 40, so withdrawing 40 is valid; account 3 becomes 0 (exact-balance withdrawal).
- Input: (continuing) `transfer(4, 1, 30)` -> Output: `true`
  Why: account 4 holds 80 >= 30; account 4 becomes 50 and account 1 becomes 45 (normal transfer).
- Input: (continuing) `deposit(3, 25)` -> Output: `true`
  Why: account 3 is valid; its balance becomes 0 + 25 = 25 (deposit always valid for an in-range account).
- Input: (continuing) `transfer(2, 3, 250)` -> Output: `false`
  Why: account 2 holds only 200 < 250, so the transfer is invalid and no balance changes (insufficient funds).
- Input: (continuing) `withdraw(9, 10)` -> Output: `false`
  Why: account 9 does not exist (n = 4), so the operation is invalid (out-of-range account).
- Input: `Ledger([0])`, then `withdraw(1, 1)` -> Output: `false`
  Why: the only account holds 0, so withdrawing 1 is invalid (min-balance edge case).

#### Approach

Store the balances in a plain array indexed by acct - 1, since accounts are densely labeled 1..n; this gives O(1) lookup and update with no hashing needed. Every operation is gated by the same two validity rules: (1) all referenced account labels must lie in [1, n], and (2) any operation that removes money from an account (withdraw, and the source side of transfer) requires the source balance to be at least the requested amount. Check these guards before mutating anything, so an invalid call leaves all balances untouched — this is what makes the operations safe to apply in place. Use a 64-bit integer type because balances can reach 10^12 and repeated deposits could push a single account well beyond 32-bit range. The design is a simple simulation over an array with careful precondition checks (a classic "design/data-structure with guarded mutations" pattern).

Complexity

Time: O(1) per operation — each of transfer, deposit, and withdraw performs a constant number of range checks and array updates; construction is O(n) to copy the initial balances. Space: O(n) to store one balance per account. Justification: direct 0-indexed array access by acct - 1 needs no search or auxiliary structure.

Python
PYTHON
class Ledger:
    def __init__(self, funds):
        self.funds = list(funds)
        self.n = len(funds)

    def _in_range(self, acct: int) -> bool:
        return 1 <= acct <= self.n

    def transfer(self, from_acct: int, to_acct: int, amount: int) -> bool:
        if not self._in_range(from_acct) or not self._in_range(to_acct):
            return False
        if self.funds[from_acct - 1] < amount:
            return False
        self.funds[from_acct - 1] -= amount
        self.funds[to_acct - 1] += amount
        return True

    def deposit(self, acct: int, amount: int) -> bool:
        if not self._in_range(acct):
            return False
        self.funds[acct - 1] += amount
        return True

    def withdraw(self, acct: int, amount: int) -> bool:
        if not self._in_range(acct) or self.funds[acct - 1] < amount:
            return False
        self.funds[acct - 1] -= amount
        return True
JavaScript
JAVASCRIPT
class Ledger {

#### cons
tructor(funds) {
        this.funds = funds.slice();
        this.n = funds.length;
    }

    _inRange(acct) {
        return acct >= 1 && acct <= this.n;
    }

    transfer(from, to, amount) {
        if (!this._inRange(from) || !this._inRange(to) || this.funds[from - 1] < amount) return false;
        this.funds[from - 1] -= amount;
        this.funds[to - 1] += amount;
        return true;
    }

    deposit(acct, amount) {
        if (!this._inRange(acct)) return false;
        this.funds[acct - 1] += amount;
        return true;
    }

    withdraw(acct, amount) {
        if (!this._inRange(acct) || this.funds[acct - 1] < amount) return false;
        this.funds[acct - 1] -= amount;
        return true;
    }
}
Java
JAVA
class Ledger {
    private final long[] funds;
    private final int n;

    public Ledger(long[] funds) {
        this.funds = funds;
        this.n = funds.length;
    }

    private boolean inRange(int acct) {
        return acct >= 1 && acct <= n;
    }

    public boolean transfer(int from, int to, long amount) {
        if (!inRange(from) || !inRange(to) || funds[from - 1] < amount) return false;
        funds[from - 1] -= amount;
        funds[to - 1] += amount;
        return true;
    }

    public boolean deposit(int acct, long amount) {
        if (!inRange(acct)) return false;
        funds[acct - 1] += amount;
        return true;
    }

    public boolean withdraw(int acct, long amount) {
        if (!inRange(acct) || funds[acct - 1] < amount) return false;
        funds[acct - 1] -= amount;
        return true;
    }
}
C++
CPP
#include <vector>
using namespace std;

class Ledger {
    vector<long long> funds;
    int n;
    bool inRange(int acct) const { return acct >= 1 && acct <= n; }

 public:
    Ledger(vector<long long>& initial) : funds(initial), n((int)initial.size()) {}

    bool transfer(int from, int to, long long amount) {
        if (!inRange(from) || !inRange(to) || funds[from - 1] < amount) return false;
        funds[from - 1] -= amount;
        funds[to - 1] += amount;
        return true;
    }

    bool deposit(int acct, long long amount) {
        if (!inRange(acct)) return false;
        funds[acct - 1] += amount;
        return true;
    }

    bool withdraw(int acct, long long amount) {
        if (!inRange(acct) || funds[acct - 1] < amount) return false;
        funds[acct - 1] -= amount;
        return true;
    }
};
C
C
#include <stdbool.h>
#include <stdlib.h>

typedef struct {
    long long* funds;
    int n;
} Ledger;

Ledger* ledgerCreate(long long* funds, int fundsSize) {
    Ledger* obj = (Ledger*)malloc(sizeof(Ledger));
    obj->n = fundsSize;
    obj->funds = (long long*)malloc(sizeof(long long) * fundsSize);
    for (int i = 0; i < fundsSize; i++) obj->funds[i] = funds[i];
    return obj;
}

static bool inRange(Ledger* obj, int acct) {
    return acct >= 1 && acct <= obj->n;
}

bool ledgerTransfer(Ledger* obj, int from, int to, long long amount) {
    if (!inRange(obj, from) || !inRange(obj, to)) return false;
    if (obj->funds[from - 1] < amount) return false;
    obj->funds[from - 1] -= amount;
    obj->funds[to - 1] += amount;
    return true;
}

bool ledgerDeposit(Ledger* obj, int acct, long long amount) {
    if (!inRange(obj, acct)) return false;
    obj->funds[acct - 1] += amount;
    return true;
}

bool ledgerWithdraw(Ledger* obj, int acct, long long amount) {
    if (!inRange(obj, acct)) return false;
    if (obj->funds[acct - 1] < amount) return false;
    obj->funds[acct - 1] -= amount;
    return true;
}

void ledgerFree(Ledger* obj) {
    free(obj->funds);
    free(obj);
}

#### Follow-up
questions
How would you support dynamically adding or removing accounts at runtime (labels no longer dense)?
Make the operations thread-safe for concurrent transfers without deadlocking on cross-account transfers.
Add an operation to query the total money in the system in O(1), or to roll back the last k operations.
What changes if a transfer must be atomic across two ledgers/shards, or if overdraft up to a limit is allowed?

