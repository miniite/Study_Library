---
type: "practice"
difficulty: "Deep"
---

# You must architect how a single 'add to cart' works identically across a PWA, a native app, and a voice/chat interface, all hitting the same backend intent. How do you factor the shared logic so a new channel is a thin adapter, not a full reimplementation?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Coordinated Client Upgrades](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Coordinated_Client_Upgrades.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

In this system design problem, we need to ensure that an "add to cart" functionality works consistently across various interfaces—Progressive Web App (PWA), native app, and voice/chat interface—all interacting with a common backend intent service. The key is to factor out shared logic while allowing each interface to handle its specific communication protocol or user interaction model.

#### Triaging the Question

The core of this question is about designing a modular system where different interfaces can interact with the same backend service efficiently. We need to identify and encapsulate the common business logic for adding an item to the cart, ensuring that each interface (PWA, native app, voice/chat) only implements its unique communication layer without duplicating substantial code.

#### Assumptions
Shared Business Logic: The core functionality of adding an item to a cart is consistent across all interfaces.
Backend Intent Service: There is a backend service responsible for handling the addition of items to the cart, regardless of the frontend interface.
Interfaces: Three distinct interfaces—PWA, native app, and voice/chat—are involved.
Step-by-Step Solution
Identify Common Business Logic:
Define what it means to add an item to a cart: product ID, quantity, customer session details (if applicable).
Create Backend Service API:
Design a RESTful API that the frontend can call to submit the "add to cart" request.
HTTP
POST /intent/cart/add
Abstract Interface Logic:
Develop an abstract interface for handling the addToCart operation, which will be implemented by each client.
PWA Implementation:
Use standard HTTP requests or fetch API to call the backend service.
Native App Implementation:
Implement a native network request (using URLSession in iOS or XMLHttpRequest in Android) to communicate with the backend.
Voice/Chat Interface Implementation:
Design voice commands and chatbot intents that can trigger the addToCart logic.
Detailed Solution

Define the Intent Service API

HTTP
POST /intent/cart/add
Content-Type: application/json

{
  "productId": "P001",
  "quantity": 2,
  "customerId": "C12345"
}

Abstract Interface for Add to Cart Logic

TYPESCRIPT
interface AddToCartInterface {
    addToCart(productId: string, quantity: number, customerId?: string): Promise<void>;
}

PWA Implementation

JAVASCRIPT

#### cons
t cartService = (async () => {
  return {
      async addToCart(productId, quantity, customerId) {
          try {
              await fetch('/intent/cart/add', {
                  method: 'POST',
                  headers: { 'Content-Type': 'application/json' },
                  body: JSON.stringify({ productId, quantity, customerId }),
              });
          } catch (error) {

#### cons
ole.error('Failed to add to cart:', error);
          }
      }
  };
})();

Native App Implementation

SWIFT
class CartService {
    func addToCart(productId: String, quantity: Int, customerId: String? = nil) async throws {
        let url = URL(string: "https://api.example.com/intent/cart/add")!
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")

        let jsonBody: [String: Any] = [
            "productId": productId,
            "quantity": quantity,
            "customerId": customerId ?? ""
        ]
        do {
            let data = try JSONSerialization.data(withJSONObject: jsonBody)
            request.httpBody = data

            let (data, _) = try await URLSession.shared.data(for: request)
            // Handle response if needed
        } catch {
            print("Failed to add to cart:", error)
        }
    }
}

Voice/Chat Interface Implementation

JAVASCRIPT
class VoiceIntentService {
  async handleAddToCartIntent(productId, quantity) {
      try {
          await this.cartService.addToCart(productId, quantity);
      } catch (error) {

#### cons
ole.error('Failed to process voice intent:', error);
      }
  }
}

#### cons
t chatIntentService = new VoiceIntentService();

#### Closing Summary

By abstracting the addToCart logic into a common interface and leveraging backend services for handling the actual business operation, we ensure that any new frontend channel can be easily integrated with minimal code duplication. This approach promotes scalability and maintainability.

#### Common Mistakes/Follow-Ups
Mistake 1: Directly duplicating the addToCart logic in each client instead of using an abstract interface.
Mistake 2: Not considering edge cases, such as handling network errors or authentication failures properly.

#### Follow-Up
Discussing how to implement retries, logging, and error handling for robustness.

**Comparison:**

```
Interface Type	Request Method	API Endpoint	Payload
PWA	POST	/intent/cart/add	{"productId": "P001", "quantity": 2, "customerId": "C12345"}
Native App	POST	https://api.example.com/intent/cart/add	{"productId": "P001", "quantity": 2, "customerId": "C12345"}
Voice/Chat Interface	Voice Command/Chatbot Intent	Triggered by user command	{"productId": "P001", "quantity": 2}
```

**Code:**

```

interface AddToCartInterface { addToCart(productId: string, quantity: number, customerId?: string): Promise<void>; }

CODE
Copy
```javascript
const cartService = (async () => {
  return {
      async addToCart(productId, quantity, customerId) {
          try {
              await fetch('/intent/cart/add', {
                  method: 'POST',
                  headers: { 'Content-Type': 'application/json' },
                  body: JSON.stringify({ productId, quantity, customerId }),
              });
          } catch (error) {
              console.error('Failed to add to cart:', error);
          }
      }
  };
})();
SWIFT
Copy
class CartService {
    func addToCart(productId: String, quantity: Int, customerId: String? = nil) async throws {
        let url = URL(string: "https://api.example.com/intent/cart/add")!
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")

        let jsonBody: [String: Any] = [
            "productId": productId,
            "quantity": quantity,
            "customerId": customerId ?? ""
        ]
        do {
            let data = try JSONSerialization.data(withJSONObject: jsonBody)
            request.httpBody = data

            let (data, _) = try await URLSession.shared.data(for: request)
            // Handle response if needed
        } catch {
            print("Failed to add to cart:", error)
        }
    }
}
JAVASCRIPT
Copy
class VoiceIntentService {
  async handleAddToCartIntent(productId, quantity) {
      try {
          await this.cartService.addToCart(productId, quantity);
      } catch (error) {
          console.error('Failed to process voice intent:', error);
      }
  }
}

const chatIntentService = new VoiceIntentService();
```

**Example input / output:**

```text
**Input**:
- PWA or Native App: POST request with `{"productId": "P001", "quantity": 2, "customerId": "C12345"}`.
- Voice/Chat Interface: A voice command or chatbot intent to add a product to the cart.

**Output**:
- Success response from backend indicating the item was added to the cart successfully.
- Error response if any validation fails (e.g., invalid product ID, insufficient stock).
```

**Data:**

```json
{
    "productId": "P001",
    "quantity": 2,
    "customerId": "C12345"
}
```

**Flow:**

1. User Interaction
2. PWA or Native App Call
3. Backend Service/intent/cart/add
4. Success/Error Response

