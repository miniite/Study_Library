---
type: "practice"
difficulty: "Deep"
---

# How would you design a payment gateway abstraction so swapping Razorpay for Stripe in one region is a config change, and what seams do you build to keep provider-specific quirks from leaking into the domain?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Open-closed Extensibility](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Open-closed_Extensibility.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a payment gateway abstraction that allows us to swap between providers like Razorpay and Stripe with minimal effort (essentially just a config change), we need to build a robust, modular architecture that keeps provider-specific logic isolated from the core domain. This approach ensures that changes in one area do not impact others and simplifies maintenance.

First, let's break down what the question is really asking:

Abstraction Layer: We need an abstraction layer that can dynamically switch between payment gateways.
Provider-Specific Quirks: These include differences in API responses, validation rules, error handling, etc.
Config Change: The ability to change from one provider to another should involve minimal code changes.

#### Triaging the Question

Key Components:

Payment Gateway Abstraction Layer (P GAL)
Provider-Specific Logic
Configuration Management

#### Assumptions

We are using a modern microservices architecture.
Both Razorpay and Stripe APIs follow RESTful principles.
Step-by-Step Solution from First Principles
1. Define the Payment Gateway Interface (P GI)

The P GI will define all the operations that need to be supported by any payment gateway provider, such as createPayment, capturePayment, etc.

PYTHON
class PaymentGatewayInterface:
    def create_payment(self, amount: float) -> dict:
        pass

    def capture_payment(self, payment_id: str) -> bool:
        pass

    # Other methods like refund, cancel, etc.
2. Implement the Providers

Implement specific classes for each provider that conform to the P GI. These will handle the quirks of each provider.

StripeProvider

PYTHON
class StripeProvider(PaymentGatewayInterface):
    def __init__(self, api_key: str):
        self.api_key = api_key

    def create_payment(self, amount: float) -> dict:
        # Implement Stripe's API call to create a payment
        pass

    def capture_payment(self, payment_id: str) -> bool:
        # Implement Stripe's API call to capture the payment
        pass

RazorpayProvider

PYTHON
class RazorpayProvider(PaymentGatewayInterface):
    def __init__(self, api_key: str):
        self.api_key = api_key

    def create_payment(self, amount: float) -> dict:
        # Implement Razorpay's API call to create a payment
        pass

    def capture_payment(self, payment_id: str) -> bool:
        # Implement Razorpay's API call to capture the payment
        pass
3. Create a Payment Gateway Factory

The factory will be responsible for instantiating the correct provider based on configuration.

PYTHON
class PaymentGatewayFactory:
    def get_provider(self, provider_name: str) -> PaymentGatewayInterface:
        if provider_name == 'stripe':
            return StripeProvider(api_key='your_stripe_api_key')
        elif provider_name == 'razorpay':
            return RazorpayProvider(api_key='your_razorpay_api_key')
        else:
            raise ValueError(f"Unknown provider: {provider_name}")
4. Domain Layer

The domain layer should interact with the P GI through an interface, not directly with any specific implementation.

PYTHON
class PaymentService:
    def __init__(self, payment_gateway_factory: PaymentGatewayFactory):
        self.payment_gateway_factory = payment_gateway_factory

    def process_payment(self, amount: float) -> bool:
        provider = self.payment_gateway_factory.get_provider('stripe')  # or 'razorpay'
        payment_id = provider.create_payment(amount)
        return provider.capture_payment(payment_id)
5. Configuration Management

Store the provider name in a config file or database.

YAML
payment_gateway:
  provider: stripe  # or razorpay
  api_key: your_api_key_here

#### Closing Summary
and Common Mistakes

In summary, by defining an interface for the payment gateway and creating separate providers that implement this interface, we can easily switch between different providers without affecting the core domain logic. The key is to ensure that provider-specific quirks are encapsulated within their respective classes.

#### Common Mistakes

Failing to define a clear interface (P GI) leads to tightly coupled code.
Not isolating provider-specific logic can result in leaking implementation details into higher layers.
Overcomplicating the configuration management, leading to hard-to-maintain scripts or hardcoded values.

**Comparison:**

```
Provider	API Key
Stripe	your_stripe_api_key
Razorpay	your_razorpay_api_key
```

**Code:**

```python
class PaymentGatewayInterface:
    def create_payment(self, amount: float) -> dict:
        pass

    def capture_payment(self, payment_id: str) -> bool:
        pass

class StripeProvider(PaymentGatewayInterface):
    def __init__(self, api_key: str):
        self.api_key = api_key

    def create_payment(self, amount: float) -> dict:
        # Implementation
        pass

    def capture_payment(self, payment_id: str) -> bool:
        # Implementation
        pass

class RazorpayProvider(PaymentGatewayInterface):
    def __init__(self, api_key: str):
        self.api_key = api_key

    def create_payment(self, amount: float) -> dict:
        # Implementation
        pass

    def capture_payment(self, payment_id: str) -> bool:
        # Implementation
        pass

class PaymentGatewayFactory:
    def get_provider(self, provider_name: str) -> PaymentGatewayInterface:
        if provider_name == 'stripe':
            return StripeProvider(api_key='your_stripe_api_key')
        elif provider_name == 'razorpay':
            return RazorpayProvider(api_key='your_razorpay_api_key')
        else:
            raise ValueError(f"Unknown provider: {provider_name}")

class PaymentService:
    def __init__(self, payment_gateway_factory: PaymentGatewayFactory):
        self.payment_gateway_factory = payment_gateway_factory

    def process_payment(self, amount: float) -> bool:
        provider = self.payment_gateway_factory.get_provider('stripe')  # or 'razorpay'
        payment_id = provider.create_payment(amount)
        return provider.capture_payment(payment_id)

# Configuration
payment_gateway:
  provider: stripe
  api_key: your_api_key_here
```

**Example input / output:**

```text
factory = PaymentGatewayFactory()
service = PaymentService(factory)
result = service.process_payment(100.0)  # result is True if payment was successful, False otherwise
print(result)  # Output: True or False based on the provider's response
```

**Data:**

```yaml
payment_gateway:
  provider: stripe
  api_key: your_api_key_here
```

**Flow:**

1. initiates payment request
2. creates provider instance
3. performs payment
4. returns result
5. used by service
6. Payment Service
7. Fabrication Layer
8. StripeProvider
9. API Call
10. Capture Payment Result
11. Service Response

