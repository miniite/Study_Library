---
type: "practice"
difficulty: "Intermediate"
---

# Design a vending-machine payment/refund flow separately from its state. Which classes own money?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ State Pattern](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/State_Pattern.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

In designing the vending machine's payment and refund flow, we will separate the concerns into distinct classes to ensure modularity and maintainability. The key idea is to have a PaymentProcessor class that manages the transaction process, interacting with the VendingMachineState to update its state accordingly.

#### Triaging the Question

The question requires us to design a vending machine's payment/refund flow in a way that it is decoupled from its state management. This ensures that changes in one component do not affect the other, making the system more robust and easier to maintain. We need to identify which classes are responsible for handling money transactions.

Key Classes and Responsibilities
VendingMachineState: Manages the current state of the vending machine (e.g., idle, waiting for payment, dispensing product).
PaymentProcessor: Processes payments and refunds, interacting with external systems like wallets or credit cards.
MoneyHandler: Abstract class for handling money-related operations.
Step-by-Step Solution

Define VendingMachineState Class

This class will have methods to transition between different states (e.g., insertCoin, dispenseProduct, cancelTransaction).

#### Example
PYTHON
class VendingMachineState:
    def insert_coin(self, coin):
        pass

    def dispense_product(self):
        pass

    def cancel_transaction(self):
        pass

Define PaymentProcessor Class

This class will handle the payment and refund process.

#### Example
PYTHON
from money_handler import MoneyHandler

class PaymentProcessor:
    def __init__(self, state: VendingMachineState, money_handler: MoneyHandler):
        self.state = state
        self.money_handler = money_handler

    def pay(self, amount):
        if self.money_handler.deposit(amount):
            self.state.insert_coin(amount)
            return True
        return False

    def refund(self, amount):
        if self.state.is_transaction_pending():
            self.money_handler.refund(amount)
            self.state.cancel_transaction()
        else:
            raise Exception("No transaction in progress to refund")

Define MoneyHandler Abstract Class

This class will provide an interface for handling money operations.

#### Example
PYTHON
from abc import ABC, abstractmethod

class MoneyHandler(ABC):
    @abstractmethod
    def deposit(self, amount) -> bool:
        pass

    @abstractmethod
    def refund(self, amount) -> None:
        pass

Concrete Implementations of MoneyHandler

Depending on the payment method (e.g., credit card, wallet), concrete implementations will be provided.

#### Example
for a simple CreditCardMoneyHandler:
PYTHON
class CreditCardMoneyHandler(MoneyHandler):
    def deposit(self, amount) -> bool:
        # Simulate processing credit card transaction
        return True

    def refund(self, amount) -> None:
        # Simulate refunding from the credit card
        pass

#### Closing Summary
and Common Mistakes/Follow-Ups

By separating the VendingMachineState from the payment/refund flow, we ensure that changes in one area do not affect the other. This design allows for better testing and easier integration with different payment methods.

#### Common Mistakes

Failing to properly decouple the classes can lead to tightly coupled code.
Not considering edge cases like invalid transactions or insufficient funds.
Overcomplicating the design by introducing unnecessary layers of abstraction.

#### Follow-Ups

How would you handle errors and retries in a real-world scenario?
What security measures would you implement for payment processing?

**Comparison:**

| Class             | Responsibilities                                                                 |
|-------------------|----------------------------------------------------------------------------------|
| VendingMachineState | Manages the state transitions and interactions with the payment processor       |
| PaymentProcessor   | Processes payments and refunds, interacts with external money handlers          |
| MoneyHandler       | Abstract class defining the interface for handling money operations              |

**Code:**

```python
class VendingMachineState:
    def insert_coin(self, coin):
        pass

    def dispense_product(self):
        pass

    def cancel_transaction(self):
        pass

from abc import ABC, abstractmethod

class MoneyHandler(ABC):
    @abstractmethod
    def deposit(self, amount) -> bool:
        pass

    @abstractmethod
    def refund(self, amount) -> None:
        pass

class CreditCardMoneyHandler(MoneyHandler):
    def deposit(self, amount) -> bool:
        # Simulate processing credit card transaction
        return True

    def refund(self, amount) -> None:
        # Simulate refunding from the credit card
        pass

class PaymentProcessor:
    def __init__(self, state: VendingMachineState, money_handler: MoneyHandler):
        self.state = state
        self.money_handler = money_handler

    def pay(self, amount):
        if self.money_handler.deposit(amount):
            self.state.insert_coin(amount)
            return True
        return False

    def refund(self, amount):
        if self.state.is_transaction_pending():
            self.money_handler.refund(amount)
            self.state.cancel_transaction()
        else:
            raise Exception("No transaction in progress to refund")
```

**Example input / output:**

```text
# Example input and output for the payment processor

# Initialize state and payment processor
state = VendingMachineState()
processor = PaymentProcessor(state, CreditCardMoneyHandler())

# Insert a coin and pay for a product
print(processor.pay(1.0))  # True

# Refund the transaction
try:
    print(processor.refund(1.0))
except Exception as e:
    print(e)  # No transaction in progress to refund
```

**Data:**

```json
{
  "state": {
    "type": "Idle",
    "coins_inserted": []
  },
  "payment_processor": {
    "state": "VendingMachineState",
    "money_handler": "CreditCardMoneyHandler"
  }
}
```

**Flow:**

1. insert_coin
2. deposit
3. refund
4. cancel_transaction
5. VendingMachineState
6. PaymentProcessor
7. CreditCardMoneyHandler

