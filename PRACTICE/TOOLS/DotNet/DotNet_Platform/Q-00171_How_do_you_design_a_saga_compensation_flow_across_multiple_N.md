---
type: "practice"
difficulty: "Expert"
---

# How do you design a saga/compensation flow across multiple .NET services for a booking that touches inventory, payment, and notification, and how do you keep it recoverable after a crash mid-flow?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ HttpClient and Resilient Outbound Calls](<../../../../TOOLS/DotNet/DotNet_Platform/HttpClient_and_Resilient_Outbound_Calls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a recoverable saga or compensation flow in a .NET backend environment, we need to ensure that our booking process is resilient against failures by carefully managing transactions and state changes across multiple services. This involves breaking down the booking into smaller steps (units of work) and ensuring each step can be retried independently while maintaining atomicity. The key components include transactional boundaries, compensation logic, and a consistent logging mechanism.

The question is probing how to implement robust distributed transactions using sagas in a microservices architecture with .NET services. We need to assume that communication between services may fail or take time due to network latency or service unavailability. Additionally, we should consider scenarios where a service might crash mid-flow, and how the system can handle such failures gracefully.

To begin, let's outline the steps involved in booking:

Reservation: Reserve inventory.
Payment: Process payment.
Notification: Send confirmation to user.

Each of these steps is a unit of work that must be tracked by the saga coordinator. If any step fails, the compensation logic will roll back the previous successful actions, ensuring data consistency and recovery.

Here's how we would approach this from first principles:

Step 1: Define the Saga Coordinator

The saga coordinator will orchestrate the booking process across multiple services. In a .NET environment, you can use an event-driven architecture with Event Sourcing to manage these steps.

CSHARP
public class BookingSaga : ISaga<BookingSagaData>
{
    private readonly IInventoryService _inventoryService;
    private readonly IPaymentService _paymentService;
    private readonly INotificationService _notificationService;

    public BookingSaga(IInventoryService inventoryService,
                       IPaymentService paymentService,
                       INotificationService notificationService)
    {
        _inventoryService = inventoryService;
        _paymentService = paymentService;
        _notificationService = notificationService;
    }

    public async Task StartReservationAsync(int userId, int productId)
    {
        // Reserve inventory
        await _inventoryService.ReserveInventory(productId);
        // Log state change
        SagaData.InventoryReserveTime = DateTime.UtcNow;

        // Continue with next step if successful
        await ContinueWithPayment(userId, productId);
    }

    private async Task ContinueWithPayment(int userId, int productId)
    {
        try
        {
            // Process payment
            await _paymentService.ProcessPayment(productId);
            SagaData.PaymentProcessedTime = DateTime.UtcNow;

            // Continue with next step if successful
            await ContinueWithNotification(userId, productId);
        }
        catch (Exception ex)
        {
            // Log failure and rollback previous steps
            SagaData.LastError = ex.Message;
            SagaDataService.Save(SagaData);
            throw new InvalidOperationException("Payment processing failed", ex);
        }
    }

    private async Task ContinueWithNotification(int userId, int productId)
    {
        try
        {
            // Send confirmation notification
            await _notificationService.SendConfirmation(userId, productId);
            SagaData.NotificationSentTime = DateTime.UtcNow;
            SagaDataService.Save(SagaData); // Persist state change
        }
        catch (Exception ex)
        {
            // Log failure and rollback previous steps
            SagaData.LastError = ex.Message;
            SagaDataService.Save(SagaData);
            throw new InvalidOperationException("Notification failed", ex);
        }
    }

    public async Task HandleSagaCompensationAsync()
    {
        if (!string.IsNullOrEmpty(SagaData.InventoryReserveTime))
        {
            await _inventoryService.ReleaseInventory(productId, SagaData.InventoryReserveTime);
        }

        if (SagaData.PaymentProcessedTime.HasValue)
        {
            await _paymentService.ReversePayment(productId, SagaData.PaymentProcessedTime.Value);
        }

        if (SagaData.NotificationSentTime.HasValue)
        {
            await _notificationService.UnconfirmReservation(userId, productId, SagaData.NotificationSentTime.Value);
        }
    }
}
Step 2: Implement Transactional Boundaries

Each service must have transactional boundaries to ensure that all steps are completed successfully before committing any changes. This is critical for maintaining the integrity of data across multiple services.

Step 3: Log State Changes and Failures

Maintain a detailed log of state transitions and errors encountered during the saga execution. This helps in debugging and understanding the flow when something goes wrong.

Step 4: Handle Crashes Mid-Flow

If a crash occurs mid-flow, the saga should be able to resume from where it left off or revert to previous states using compensation logic. Ensure that each service has an IDempotent operation mechanism to handle retries without causing duplicate actions.

#### Closing Summary
and Common Mistakes/Follow-Ups

The key takeaway is that designing recoverable sagas in .NET requires careful orchestration of distributed transactions, robust error handling, and consistent logging. Common mistakes include not handling failures correctly or neglecting the compensatory logic for recovering from partial successes. Always ensure that your saga can gracefully handle various failure scenarios by rolling back to a consistent state.

Common follow-ups might involve discussing additional considerations such as performance optimization, dead letter queues, and scaling strategies in distributed systems.

**Comparison:**

```
Step	Action	Logs
1	Reserve Inventory	InventoryReserveTime
2	Process Payment	PaymentProcessedTime
3	Send Confirmation	NotificationSentTime, LastError
Failure Scenarios	Recovery Strategy
Unsuccessful Reservation	Rollback to original state via ReleaseInventory
Failed Payment	Reverse payment using ReversePayment method
Sent Notification	Unconfirm reservation with UnconfirmReservation
```

**Code:**

```csharp
public class BookingSaga : ISaga<BookingSagaData>
{
    private readonly IInventoryService _inventoryService;
    private readonly IPaymentService _paymentService;
    private readonly INotificationService _notificationService;

    public BookingSaga(IInventoryService inventoryService,
                       IPaymentService paymentService,
                       INotificationService notificationService)
    {
        _inventoryService = inventoryService;
        _paymentService = paymentService;
        _notificationService = notificationService;
    }

    public async Task StartReservationAsync(int userId, int productId)
    {
        // Reserve inventory
        await _inventoryService.ReserveInventory(productId);
        SagaData.InventoryReserveTime = DateTime.UtcNow;

        await ContinueWithPayment(userId, productId);
    }

    private async Task ContinueWithPayment(int userId, int productId)
    {
        try
        {
            // Process payment
            await _paymentService.ProcessPayment(productId);
            SagaData.PaymentProcessedTime = DateTime.UtcNow;

            await ContinueWithNotification(userId, productId);
        }
        catch (Exception ex)
        {
            SagaData.LastError = ex.Message;
            SagaDataService.Save(SagaData);
            throw new InvalidOperationException("Payment processing failed", ex);
        }
    }

    private async Task ContinueWithNotification(int userId, int productId)
    {
        try
        {
            // Send confirmation notification
            await _notificationService.SendConfirmation(userId, productId);
            SagaData.NotificationSentTime = DateTime.UtcNow;
            SagaDataService.Save(SagaData);
        }
        catch (Exception ex)
        {
            SagaData.LastError = ex.Message;
            SagaDataService.Save(SagaData);
            throw new InvalidOperationException("Notification failed", ex);
        }
    }

    public async Task HandleSagaCompensationAsync()
    {
        if (!string.IsNullOrEmpty(SagaData.InventoryReserveTime))
        {
            await _inventoryService.ReleaseInventory(productId, SagaData.InventoryReserveTime);
        }

        if (SagaData.PaymentProcessedTime.HasValue)
        {
            await _paymentService.ReversePayment(productId, SagaData.PaymentProcessedTime.Value);
        }

        if (SagaData.NotificationSentTime.HasValue)
        {
            await _notificationService.UnconfirmReservation(userId, productId, SagaData.NotificationSentTime.Value);
        }
    }
}
```

**Example input / output:**

```text
Input:
- StartReservationAsync(123, 456)

Output (Dry-run):
- InventoryService.ReserveInventory(456)
- SagaData.InventoryReserveTime = DateTime.UtcNow

Try:
- PaymentService.ProcessPayment(456)
- SagaData.PaymentProcessedTime = DateTime.UtcNow
- NotificationService.SendConfirmation(123, 456)

Error Handling (in case of an exception):
- SagaData.LastError = ex.Message
```

**Data:**

```json
{
  "userId": 123,
  "productId": 456,
  "InventoryReserveTime": null,
  "PaymentProcessedTime": null,
  "NotificationSentTime": null,
  "LastError": ""
}
```

**Flow:**

1. Yes
2. Yes
3. No
4. Start Reservation
5. Success?
6. Continue to Payment
7. Success?
8. Send Confirmation
9. Complete Booking
10. Compensation for Reservation
11. Rollback Inventory and SaveSaga Data

