---
type: "practice"
difficulty: "Intermediate"
---

# Design a movie ticket booking system like BookMyShow. What are the core entities?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ State Pattern](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/State_Pattern.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's being asked: Design the low-level model for a BookMyShow-style movie ticket booking system and enumerate the core entities. The interviewer wants a clean object/data model — the entities, their relationships, and the key operations (search shows, pick seats, hold, pay, confirm). At intermediate level they expect you to get the entity boundaries right (the classic trap is conflating Movie, Show, and Screen), model seat inventory correctly, and show awareness of the booking lifecycle and concurrency without going full distributed-systems.

Interviewer is probing for: Can you decompose a real product into entities with the right cardinalities? Do you separate the catalog (movies, cinemas, screens) from the scheduling (shows) from the transaction (bookings, payments)? Do you model seats as physical (belong to a screen) vs. bookable-per-show (a ShowSeat status)? Do you know where money and state live?

Clarifying questions / assumptions: Multi-city, multi-cinema (yes). Seat-level selection with categories/prices (yes). Payments via a gateway (yes). Do we handle offers/coupons and food? I'll model the core and mention extensions. I'll assume seats are assigned (not general admission), a hold/reservation window of ~8–10 minutes, and a relational store as the source of truth.

From first principles to the entity model

Step 1 — separate the three planes. The single most important design move: split the system into (a) Catalog — what exists (Movies, Cinemas, Screens, physical Seats); (b) Scheduling — when a movie plays (Show = a movie on a screen at a time); (c) Transaction — a user booking specific seats for a show and paying. Beginners collapse "Movie" and "Show," which breaks pricing, timing, and seat inventory. A Movie is the film (Inception); a Show is a specific screening (Inception, Screen 3 at PVR Forum, 7:30 PM, today). Seats are physical to a Screen, but their booked/available status is per Show.

Step 2 — the core entities.

Movie — id, title, language, duration, genre, certification, rating. Catalog-level, city-independent.
City — where cinemas live; drives search ("shows in Bengaluru").
Cinema (Theatre) — id, name, address, cityId, list of Screens.
Screen (Auditorium) — id, cinemaId, name ("Audio 3"), and a fixed seat layout.
Seat — id, screenId, rowLabel, seatNumber, category (SILVER/GOLD/RECLINER). This is the physical seat, defined once per screen.
Show — id, movieId, screenId, startTime, endTime, language, format (2D/3D/IMAX), and a base price map by seat category. The join of a movie to a screen at a time.
ShowSeat — id, showId, seatId, status (AVAILABLE / HELD / BOOKED), price, and (when held/booked) the holder's bookingId + hold expiry. This is the bookable inventory — one row per (show × seat). Its existence is why two shows on the same screen have independent seat availability.
User — id, name, email, phone.
Booking — id, userId, showId, list of ShowSeat ids, status (CREATED / PENDING_PAYMENT / CONFIRMED / CANCELLED / EXPIRED), amount, createdAt, expiresAt.
Payment — id, bookingId, amount, status (INITIATED / SUCCESS / FAILED), gateway ref, method. Kept separate from Booking so a booking can have retries.
Ticket — id, bookingId, QR/barcode, issued on CONFIRMED. (Some designs fold this into Booking.)

Nice-to-haves I'd mention but not center: Offer/Coupon, SeatLock (if not modeled via ShowSeat.status), Notification, Review/Rating.

Step 3 — relationships (cardinalities). City 1—* Cinema; Cinema 1—* Screen; Screen 1—* Seat; Movie 1—* Show; Screen 1—* Show; Show 1—* ShowSeat; Seat 1—* ShowSeat (one per show it appears in); User 1—* Booking; Booking — ShowSeat (a booking holds several show-seats); Booking 1—1 Payment (or 1—* with retries); Booking 1—1 Ticket.

Step 4 — the booking lifecycle (how the entities move).

User searches: City → Cinemas → Shows for a Movie/date.
User opens a Show → we render the seat map from ShowSeat rows (AVAILABLE vs BOOKED).
User selects seats → we hold them: flip those ShowSeat rows AVAILABLE→HELD with an expiry (~8 min) and create a Booking in PENDING_PAYMENT. The hold is what stops others from taking them.
User pays → Payment SUCCESS → Booking→CONFIRMED, ShowSeat→BOOKED, Ticket issued.
If payment fails or the hold expires → a sweeper flips ShowSeat back to AVAILABLE and Booking→EXPIRED/CANCELLED.

Step 5 — where correctness lives. The hold + expiry + a uniqueness/atomic-update guard on ShowSeat is what prevents double-booking (covered in depth in the sibling concurrency question). Money is authoritative in Payment; seat truth is in ShowSeat; the user-facing state is Booking.status. Prices live on the Show (base by category) and are snapshotted onto ShowSeat/Booking so a later price change doesn't alter a completed booking.

Step 6 — a few real-world refinements at this level. Seat layout is best stored as structured data (rows × columns with gaps/aisles) so the UI can render it. Generating ShowSeat rows lazily (on first view) or eagerly (at show creation) is a trade-off — eager makes availability queries a simple filter; lazy saves rows for shows nobody opens. I'd generate them at show creation for simplicity and predictable queries. Pricing is per-category, sometimes per-show (weekend surge), so keep a price map on Show and snapshot it.

Summary — what a strong answer emphasises

The core entities are Movie, City, Cinema, Screen, Seat, Show, ShowSeat, User, Booking, Payment, Ticket — with the crucial separation of catalog (Movie/Cinema/Screen/Seat) vs. schedule (Show) vs. transaction (Booking/Payment), and the insight that bookable inventory is ShowSeat = Seat × Show with a status. Center the answer on: correct cardinalities, the AVAILABLE→HELD→BOOKED lifecycle with an expiry, and separating Payment from Booking. Common mistakes: merging Movie and Show, attaching availability to the physical Seat instead of ShowSeat, and forgetting the hold/expiry that underpins concurrency. Likely follow-ups: "how do you prevent double-booking?" (the concurrency question) and "how do you scale search across cities?"

**Comparison:**

```
Entity	Plane	Key attributes	Belongs to
Movie	Catalog	title, language, duration, cert	—
Cinema	Catalog	name, address	City
Screen	Catalog	name, layout	Cinema
Seat	Catalog	row, number, category	Screen
Show	Schedule	startTime, format, basePrice	Movie + Screen
ShowSeat	Inventory	status, price, holdExpiresAt	Show + Seat
Booking	Transaction	status, amount, expiresAt	User + Show
Payment	Transaction	status, gatewayRef	Booking
Ticket	Transaction	qrCode	Booking
Design decision	Option A	Option B	Choice
Availability lives on	physical Seat	ShowSeat (Seat×Show)	ShowSeat — per-show truth
Movie vs screening	one entity	Movie + Show split	split
Generate ShowSeat rows	eager at show create	lazy on first view	eager (simple queries)
Payment	field on Booking	separate entity	separate (retries)
```

**Code:**

```java
// Core entity model (intermediate LLD sketch) — catalog / schedule / transaction

enum SeatCategory { SILVER, GOLD, RECLINER }
enum ShowSeatStatus { AVAILABLE, HELD, BOOKED }
enum BookingStatus { CREATED, PENDING_PAYMENT, CONFIRMED, CANCELLED, EXPIRED }
enum PaymentStatus { INITIATED, SUCCESS, FAILED }

// ---- Catalog: what exists ----
class Movie   { String id, title, language, certification; int durationMin; }
class City    { String id, name; }
class Cinema  { String id, name, address; City city; List<Screen> screens; }
class Screen  { String id, name; Cinema cinema; List<Seat> layout; }
class Seat    { String id; Screen screen; String rowLabel; int seatNumber; SeatCategory category; }

// ---- Schedule: when a movie plays ----
class Show {
    String id; Movie movie; Screen screen;
    Instant startTime, endTime; String format;              // 2D/3D/IMAX
    Map<SeatCategory, Money> basePrice;                     // snapshot onto ShowSeat
}

// ---- Bookable inventory = Seat x Show ----
class ShowSeat {
    String id; Show show; Seat seat;
    ShowSeatStatus status = AVAILABLE;
    Money price;                 // resolved from Show.basePrice at creation
    String bookingId;            // holder, when HELD/BOOKED
    Instant holdExpiresAt;       // set on HELD
}

// ---- Transaction ----
class User    { String id, name, email, phone; }
class Booking {
    String id; User user; Show show;
    List<ShowSeat> seats; BookingStatus status = CREATED;
    Money amount; Instant createdAt, expiresAt;
}
class Payment { String id, bookingId, gatewayRef; Money amount; PaymentStatus status; String method; }
class Ticket  { String id, bookingId, qrCode; Instant issuedAt; }

// ---- Booking service: the lifecycle ----
class BookingService {
    Booking hold(User u, Show show, List<ShowSeat> chosen) {
        // (concurrency guard lives here — see sibling question)
        Instant now = Instant.now();
        for (ShowSeat ss : chosen) {                 // atomic AVAILABLE->HELD per seat
            ss.status = ShowSeatStatus.HELD;
            ss.holdExpiresAt = now.plus(Duration.ofMinutes(8));
        }
        Booking b = new Booking();
        b.user = u; b.show = show; b.seats = chosen;
        b.status = BookingStatus.PENDING_PAYMENT;
        b.amount = sum(chosen, ss -> ss.price);
        b.expiresAt = now.plus(Duration.ofMinutes(8));
        chosen.forEach(ss -> ss.bookingId = b.id);
        return b;
    }
    void confirm(Booking b, Payment p) {             // on gateway SUCCESS
        if (p.status != PaymentStatus.SUCCESS) throw new IllegalStateException();
        b.seats.forEach(ss -> ss.status = ShowSeatStatus.BOOKED);
        b.status = BookingStatus.CONFIRMED;
        // issue Ticket(s)
    }
    void expireSweep(Booking b) {                    // background, when now > expiresAt
        if (b.status == BookingStatus.PENDING_PAYMENT) {
            b.seats.forEach(ss -> { ss.status = ShowSeatStatus.AVAILABLE; ss.bookingId = null; });
            b.status = BookingStatus.EXPIRED;
        }
    }
}
```

**Example input / output:**

```text
Walkthrough — user books 2 seats

Search:  city=Bengaluru, movie=Inception, date=2026-07-07
  -> Cinemas: [PVR Forum, INOX Garuda]
  -> Shows @ PVR Forum Screen-3: [7:30 PM 2D, 10:15 PM IMAX]

Open show s_501 (7:30 PM) -> render ShowSeat rows:
  A1 AVAILABLE, A2 AVAILABLE, A3 BOOKED, A4 AVAILABLE ...

Select A1, A2 -> hold():
  show_seat(A1,A2): AVAILABLE -> HELD, hold_expires_at = now+8m
  booking b_900: status=PENDING_PAYMENT, amount = 2 x GOLD price = 700

Pay 700 -> payment SUCCESS -> confirm():
  show_seat(A1,A2): HELD -> BOOKED
  booking b_900: -> CONFIRMED, Ticket t_900 issued with QR

(If user never pays) after 8 min sweep:
  show_seat(A1,A2): HELD -> AVAILABLE, booking -> EXPIRED
```

**Data:**

```sql
-- Relational schema for the core entities
CREATE TABLE movie   (id PK, title, language, certification, duration_min);
CREATE TABLE city    (id PK, name);
CREATE TABLE cinema  (id PK, name, address, city_id FK->city);
CREATE TABLE screen  (id PK, cinema_id FK->cinema, name);
CREATE TABLE seat    (id PK, screen_id FK->screen, row_label, seat_number, category,
                      UNIQUE(screen_id, row_label, seat_number));
CREATE TABLE show    (id PK, movie_id FK->movie, screen_id FK->screen,
                      start_time, end_time, format);
CREATE TABLE show_seat (id PK, show_id FK->show, seat_id FK->seat,
                        status ENUM('AVAILABLE','HELD','BOOKED') DEFAULT 'AVAILABLE',
                        price, booking_id NULL, hold_expires_at NULL,
                        UNIQUE(show_id, seat_id));       -- one row per seat per show
CREATE TABLE booking (id PK, user_id FK, show_id FK, status, amount,
                      created_at, expires_at);
CREATE TABLE booking_seat (booking_id FK, show_seat_id FK, PRIMARY KEY(booking_id, show_seat_id));
CREATE TABLE payment (id PK, booking_id FK, amount, status, gateway_ref, method);
CREATE TABLE ticket  (id PK, booking_id FK, qr_code, issued_at);
```

**Flow:**

1. has
2. has
3. contains
4. screened_as
5. hosts
6. sells
7. instance_of
8. makes
9. for
10. reserves
11. paid_by
12. issues
13. CITY
14. CINEMA
15. SCREEN
16. SEAT
17. MOVIE
18. SHOW
19. SHOWSEAT
20. USER
21. BOOKING
22. PAYMENT
23. TICKET

