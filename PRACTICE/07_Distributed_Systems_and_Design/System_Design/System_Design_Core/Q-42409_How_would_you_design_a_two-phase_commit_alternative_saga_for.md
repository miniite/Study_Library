---
type: "practice"
difficulty: "Expert"
---

# How would you design a two-phase commit alternative (saga) for a multi-service booking that reserves a flight, a hotel, and a car — and what are your compensating actions when step 2 fails?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Payment Reconciliation](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Payment_Reconciliation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary:

Designing a two-phase commit alternative (saga) for a multi-service booking system that reserves a flight, hotel, and car involves ensuring atomicity through compensatory actions in case of failures during step 2. This design ensures the system can handle complex operations with retries and fallback mechanisms.

#### Triage

The question is asking us to design a saga pattern implementation for a multi-service booking process. A saga consists of multiple steps (in this case, reserving flight, hotel, and car), each potentially involving different services. The key challenge is ensuring that if any step fails in phase 2, we can revert the state changes made during phase 1.

#### Assumptions
We have three main services: Flight Booking Service, Hotel Booking Service, and Car Booking Service.
Each service has its own database to manage bookings and maintain transactional integrity.
The system supports retries for each step until a successful outcome is achieved or a maximum retry limit is hit.
Step-by-Step Solution:

Phase 1: Prepare Phase

Flight Reservation: Call the Flight Booking Service API with the flight details and store a reservation ID in your local database.
Hotel Reservation: Call the Hotel Booking Service API with the hotel details and store a reservation ID in your local database.
Car Reservation: Call the Car Booking Service API with the car details and store a reservation ID in your local database.

Phase 2: Confirm Phase

If all reservations are successful, confirm each booking by calling the corresponding services to finalize them (e.g., payment, seat allocation).

Compensatory Actions for Step 2 Failure:

If any step fails during phase 2, we need to revert the changes made in phase 1.

#### Example
If the hotel reservation fails after the car and flight reservations have been confirmed:
Flight: Undo seat allocation by calling a "revert" method on the Flight Booking Service API.
Car: Release the allocated vehicle by calling its "revert" method.
Ensure that each service has a rollback mechanism to handle such failures.
Detailed Steps:

Phase 1: Prepare Phase

JAVASCRIPT
// Pseudocode for Phase 1

async function prepareReservations(flightDetails, hotelDetails, carDetails) {

#### cons
t flightReservationId = await callFlightService(flightDetails);

#### cons
t hotelReservationId = await callHotelService(hotelDetails);

#### cons
t carReservationId = await callCarService(carDetails);

    // Store reservation IDs in local database
    storeReservationIds(flightReservationId, hotelReservationId, carReservationId);

    return { flightReservationId, hotelReservationId, carReservationId };
}

Phase 2: Confirm Phase

JAVASCRIPT
async function confirmReservations(reservationIds) {

#### cons
t { flightReservationId, hotelReservationId, carReservationId } = reservationIds;

    // Confirm each booking by calling the respective services
    await callFlightServiceToConfirm(flightReservationId);
    await callHotelServiceToConfirm(hotelReservationId);
    await callCarServiceToConfirm(carReservationId);

    return true;
}

Compensatory Actions for Step 2 Failure

JAVASCRIPT
async function handlePhase2Failure(reservationIds, failedStep) {

#### cons
t { flightReservationId, hotelReservationId, carReservationId } = reservationIds;

    switch (failedStep) {
        case 'hotel':
            // Revert hotel booking
            await callHotelServiceToRevert(hotelReservationId);
            break;
        case 'car':
            // Revert car booking
            await callCarServiceToRevert(carReservationId);
            break;
        default:

#### cons
ole.error('Unknown step failure');
    }

    // Attempt to revert flight if necessary
    if (failedStep !== 'flight') {
        await callFlightServiceToRevert(flightReservationId);
    }
}

#### Closing Summary
and Common Mistakes/Follow-Ups:

#### Closing Summary
By implementing the saga pattern, we ensure that our multi-service booking system can handle failures gracefully by reverting changes made in earlier steps. This approach requires careful design of compensatory actions for each step.

#### Common Mistakes
Failing to implement rollback logic on services.
Not handling all failure scenarios properly (e.g., what if both flight and hotel fail, but only one car is reserved?).
Overlooking the need for a consistent retry mechanism.

#### Follow-Ups
How would you handle retries and timeouts in this system?
What about cases where multiple services might be down simultaneously?
Discuss how you would test such a system to ensure it works as expected.

**Comparison:**

```
Step	Service	Action	Rollback Action
1	Flight	Reserve	Revert
2	Hotel	Confirm	Revert
3	Car	Confirm	Release
```

**Code:**

```javascript
async function prepareReservations(flightDetails, hotelDetails, carDetails) {
    const flightReservationId = await callFlightService(flightDetails);
    const hotelReservationId = await callHotelService(hotelDetails);
    const carReservationId = await callCarService(carDetails);

    // Store reservation IDs in local database
    storeReservationIds(flightReservationId, hotelReservationId, carReservationId);

    return { flightReservationId, hotelReservationId, carReservationId };
}

async function confirmReservations(reservationIds) {
    const { flightReservationId, hotelReservationId, carReservationId } = reservationIds;

    // Confirm each booking by calling the respective services
    await callFlightServiceToConfirm(flightReservationId);
    await callHotelServiceToConfirm(hotelReservationId);
    await callCarServiceToConfirm(carReservationId);

    return true;
}

async function handlePhase2Failure(reservationIds, failedStep) {
    const { flightReservationId, hotelReservationId, carReservationId } = reservationIds;

    switch (failedStep) {
        case 'hotel':
            // Revert hotel booking
            await callHotelServiceToRevert(hotelReservationId);
            break;
        case 'car':
            // Revert car booking
            await callCarServiceToRevert(carReservationId);
            break;
        default:
            console.error('Unknown step failure');
    }

    // Attempt to revert flight if necessary
    if (failedStep !== 'flight') {
        await callFlightServiceToRevert(flightReservationId);
    }
}
```

**Example input / output:**

```text
Input:
- flightDetails
- hotelDetails
- carDetails

Output (Phase 1):
{
    "flightReservationId": "FR123",
    "hotelReservationId": "HR456",
    "carReservationId": "CR789"
}

Output (Phase 2 - Successful):
true

Output (Phase 2 - Failure on Hotel Booking):
false
```

**Data:**

```json
{
  "flightDetails": {
    "flightNumber": "CA123",
    "departureTime": "2023-10-01T14:00:00Z"
  },
  "hotelDetails": {
    "location": "New York City",
    "checkInDate": "2023-10-01",
    "checkOutDate": "2023-10-05"
  },
  "carDetails": {
    "pickupLocation": "JFK Airport",
    "returnLocation": "Times Square",
    "pickUpTime": "2023-10-01T16:00:00Z"
  }
}
```

**Flow:**

1. Yes
2. No
3. Hotel
4. Car
5. Prepare Phase
6. Is All Successful?
7. Confirm Phase
8. Handle Failure
9. Commit Transactions
10. Which Step Failed?
11. Revert Hotel Booking
12. Release Car Booking
13. Revert Flight Booking
14. Revert Flight Booking

