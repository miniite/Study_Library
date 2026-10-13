---
type: "practice"
difficulty: "Deep"
---

# Design an Elevator system. What classes and how do you dispatch a car to a request?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Classic LLD Problems](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Classic_LLD_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked: Do the low-level design of a multi-car elevator system: (1) the class model (elevators, requests, buttons, the controller), and (2) the dispatch algorithm — given a hall request ("someone on floor 7 pressed Up"), which car do we send and how does a car decide its next move? This is the classic elevator LLD; at "deep understanding" level the interviewer wants clean OOP plus a correct scheduling algorithm (the SCAN/LOOK "elevator algorithm"), not just boxes.

What the interviewer is probing: can I (a) separate the two request types (hall/external vs. car/internal) — a very common miss; (b) model a car as a stateful mover with a direction and a set of stops; (c) implement a sane dispatch policy (nearest-car / directional / estimated-time), and articulate its trade-offs; (d) keep the design extensible and thread-safe since a real building has concurrent button presses.

#### Assumptions
/ clarifying questions I'd raise:

Building with F floors and N elevator cars, one shared controller.
Two request sources: hall panel on each floor (Up/Down button — an external/landing request with a direction) and the car panel inside each car (floor buttons — internal requests, direction implied by current position).
Cars move continuously; we model discrete floor arrivals. Capacity/weight limits, door timing, VIP/express modes are follow-ups I'll name but not fully build.
Optimise for reasonable wait time, not provably optimal scheduling (optimal multi-car dispatch is NP-hard; production systems use heuristics/ML).

Plan: enumerate classes → define a single car's motion (the LOOK algorithm) → define the controller's car-selection (dispatch) heuristic → code the core → dry-run a scenario → summarise trade-offs and failure modes.

The class model

The key insight that organises everything: a hall request must be assigned to a car by the controller; a car request just adds a stop to that specific car. So requests are first-class and typed.

Direction — enum {UP, DOWN, IDLE}.
Request — floor, direction (for hall requests), and type (HALL vs CAR). A hall request carries the desired travel direction; a car request just carries the target floor.
ElevatorCar — the stateful mover. Holds id, currentFloor, direction, and a set of pending stops. Its job: on each tick, decide the next floor and move; it exposes addStop(floor), step(), and read-only getCurrentFloor()/getDirection(). It runs the LOOK algorithm internally (keep going in the current direction, serving stops in order, until none remain that way, then reverse).
ElevatorController (dispatcher) — the brain. Receives hall requests, picks the best car via a scoring function, and forwards car requests straight to the named car. Owns the list of ElevatorCars.
Building — wiring: F floors, hall panels, and the controller. Mostly composition.
Button/Panel (optional) — model the physical UI: HallPanel(floor) produces hall requests; CarPanel(car) produces car requests. Nice for completeness; the logic lives in controller + car.

Why split the controller from the car? Single Responsibility. The car knows how to move efficiently on its own stop-set; the controller knows how to allocate a new hall request across cars. That separation also makes both independently testable and lets you swap the dispatch policy without touching motion code (Strategy pattern candidate).

See code_blocks for a compilable core and flowcharts for the class + dispatch view.

How one car moves — the LOOK (elevator) algorithm

Each car keeps two sorted structures (or one min/max view): stops above and below the current floor. The rule:

If direction is UP, keep serving the nearest stop above until there are none above; then switch to DOWN if stops remain below, else go IDLE.
Symmetric for DOWN.

This is LOOK (a refinement of SCAN/the "elevator algorithm"): the car sweeps in one direction serving everything en route, reverses only when nothing remains ahead — instead of bouncing per-request. It bounds worst-case wait and avoids starvation: a request always gets served within at most one full sweep because the sweep never skips a valid en-route stop. Using a TreeSet for stops gives O(log k) insert and O(log k) "next stop in direction" via ceiling/floor.

Crucially, a hall request's direction is respected: a car travelling UP should only pick up a hall "Up" request at floor 7 if 7 is ahead of it — otherwise the passenger going up would be carried down first. That direction match is part of both the car's stop selection and the controller's scoring.

Dispatch — which car do we send?

When a hall request arrives at (floor, dir), the controller scores every car and assigns the best. A solid, widely-used heuristic is directional nearest-car / estimated time:

For each car, compute a suitability score:

Same-direction & request is ahead (car moving UP, request above it, wants UP): score = distance (small = best) — the car can pick it up en route, essentially free.
Idle car: score = distance + small penalty (it must start moving).
Opposite direction or request behind it: score = large penalty ≈ (steps to finish current sweep) + distance back — it must finish its run first.

Pick the minimum score; forward the request to that car via addStop. This approximates "minimise passenger wait" cheaply. Production systems (Otis/KONE) go further with estimated-time-of-arrival models, zoning, and even reinforcement learning, but the directional-nearest heuristic is the correct thing to present and defend in an interview.

#### Trade-offs
to state: nearest-car alone (ignoring direction) can send a car that then has to reverse, hurting everyone. Pure directional can under-utilise idle cars. ETA-based is best but needs a motion model. It's a classic latency-vs-throughput and fairness (anti-starvation) balance; I'd default to directional-nearest and mention ETA as the upgrade.

Concurrency & real-world edges (what "deep understanding" wants)
Thread safety: many buttons fire concurrently. The controller's assignment and each car's stop-set need synchronisation (a lock per car, or a thread-safe queue feeding each car's own worker thread). I'd give each car its own event loop and make the controller enqueue commands — avoids shared-mutable races.
De-duplication: two people press "Up" on floor 7 → one stop, not two.
Capacity/weight: a full car should be skippable by dispatch; model capacity and load.
Fairness/starvation: LOOK inherently avoids starvation; I'd note it and avoid greedy policies that can strand a far request forever.
Failure modes: a car out of service must be removed from dispatch and its pending hall requests reassigned; car requests inside it are lost (passengers exit). Door sensors, emergency stop, and express/VIP modes are extensions via extra states or a priority queue.
Summary — what a strong answer emphasises
Separate hall vs. car requests, and separate the controller (dispatch) from the car (motion) — SRP, testability, swappable strategy.
Each car runs LOOK/SCAN: sweep in one direction serving en-route stops, reverse only when none remain ahead — bounded wait, no starvation.
Dispatch by directional nearest-car / ETA: prefer a same-direction car that passes the floor en route; penalise opposite-direction/behind; mention ETA and RL as the production upgrade.
Name concurrency, dedup, capacity, and reassignment on failure — these turn a textbook answer into a production one.

#### Common mistakes
treating hall and car requests the same (losing the direction constraint); a naive nearest-car that ignores direction and makes cars reverse; per-request bouncing instead of sweeping; ignoring thread safety. Follow-ups: "How do you make it optimal?" (it's NP-hard for N cars → heuristics/ETA/ML) and "How do you add an express car / peak-hour zoning?" (Strategy for dispatch + per-car floor constraints).

**Comparison:**

```
Concern	Hall request (external)	Car request (internal)
Source	landing panel (Up/Down)	panel inside a car
Carries direction?	yes (desired travel dir)	no (implied by position)
Who handles it	controller picks a car	added straight to that car
Dispatch policy	Idea	Pros	Cons
Nearest-car (distance only)	send closest car	simple	may force a reverse; ignores direction
Directional nearest / en-route	prefer same-dir car passing the floor; penalise opposite	low wait, no needless reversal	idle cars can be under-used
ETA / estimated-time	model each car's finish time, pick min	best wait times	needs motion model, more compute
Zoning + ML (production)	assign zones, learn patterns	scales, peak-aware	complex, data-hungry
Complexity (per car)	Op	Cost
add a stop	TreeSet insert	O(log k)
next stop in direction	ceiling/floor	O(log k)
dispatch (pick car)	min over N cars	O(N log k)
```

**Code:**

```java
enum Direction { UP, DOWN, IDLE }
enum ReqType   { HALL, CAR }

record Request(int floor, Direction direction, ReqType type) {}

// ---- One car: stateful mover running the LOOK algorithm ----
class ElevatorCar {
    final int id;
    int currentFloor = 0;
    Direction direction = Direction.IDLE;
    private final java.util.TreeSet<Integer> upStops   = new java.util.TreeSet<>();
    private final java.util.TreeSet<Integer> downStops = new java.util.TreeSet<>();

    ElevatorCar(int id){ this.id = id; }

    synchronized void addStop(int floor){
        if (floor == currentFloor) return;
        if (floor > currentFloor) upStops.add(floor); else downStops.add(floor);
        if (direction == Direction.IDLE)
            direction = floor > currentFloor ? Direction.UP : Direction.DOWN;
    }

    // one tick: move one floor toward the next stop in the current sweep direction (LOOK)
    synchronized void step(){
        if (direction == Direction.UP) {
            Integer next = upStops.ceiling(currentFloor + 1);
            if (next != null) { currentFloor++; if (currentFloor == next) { upStops.remove(next); open(); } }
            else reverseOrIdle();
        } else if (direction == Direction.DOWN) {
            Integer next = downStops.floor(currentFloor - 1);
            if (next != null) { currentFloor--; if (currentFloor == next) { downStops.remove(next); open(); } }
            else reverseOrIdle();
        }
    }
    private void reverseOrIdle(){
        if (direction == Direction.UP   && !downStops.isEmpty()) direction = Direction.DOWN;
        else if (direction == Direction.DOWN && !upStops.isEmpty()) direction = Direction.UP;
        else direction = Direction.IDLE;
    }
    private void open(){ /* open doors, board/alight */ }

    // score for dispatch: lower is better
    synchronized int scoreFor(int floor, Direction reqDir){
        int dist = Math.abs(currentFloor - floor);
        if (direction == Direction.IDLE) return dist + 1;                 // idle: near but must start
        boolean sameDir = direction == reqDir;
        boolean ahead   = (direction == Direction.UP   && floor >= currentFloor)
                       || (direction == Direction.DOWN && floor <= currentFloor);
        if (sameDir && ahead) return dist;                               // en-route pickup: best
        return 1000 + dist;                                             // must finish sweep first
    }
}

// ---- Controller: dispatch a hall request to the best car ----
class ElevatorController {
    private final java.util.List<ElevatorCar> cars;
    ElevatorController(java.util.List<ElevatorCar> cars){ this.cars = cars; }

    // external/hall request (floor + desired direction)
    void requestHall(int floor, Direction dir){
        ElevatorCar best = cars.stream()
            .min(java.util.Comparator.comparingInt(c -> c.scoreFor(floor, dir)))
            .orElseThrow();
        best.addStop(floor);
    }
    // internal/car request: passenger inside car `id` presses a floor
    void requestCar(int carId, int floor){ cars.get(carId).addStop(floor); }

    void tick(){ for (ElevatorCar c : cars) c.step(); } // driven by a clock/thread
}
```

**Example input / output:**

```

Dispatch dry-run. Cars: C0 at floor 2 going UP; C1 at floor 8 going DOWN. Hall request: floor 7, UP.

CODE
Copy
scoreFor(floor=7, reqDir=UP):
  C0 (at 2, UP):   sameDir=UP==UP=true, ahead=(7>=2)=true  -> score = |2-7| = 5   <-- best
  C1 (at 8, DOWN): sameDir=false                            -> score = 1000+|8-7| = 1001
assign floor 7 to C0.  C0.addStop(7) -> upStops = {5,7,9}

LOOK sweep of C0 from floor 2:
  step: 2->3->4->5(stop,open)->6->7(stop,open)->8->9(stop,open)-> upStops empty
        downStops empty -> direction = IDLE
Order served: 5, 7, 9  (single upward sweep, no bouncing, hall-Up direction honoured)

Now a car request inside C0 for floor 3 while at floor 5 going UP:

CODE
Copy
C0.addStop(3): 3 < currentFloor(5) -> goes to downStops = {3}
 -> served on the RETURN sweep after the upward stops finish (LOOK), not by reversing immediately.
```

**Data:**

```text
Request (hall): { floor: 7, direction: UP,   type: HALL }   // controller must pick a car
Request (car) : { floor: 3, direction: --,   type: CAR, carId: 1 }  // added to that car only

ElevatorCar state:
  { id: 0, currentFloor: 2, direction: UP,   upStops: [5,9],   downStops: [] }
  { id: 1, currentFloor: 8, direction: DOWN, upStops: [],      downStops: [4,1] }
```

**Flow:**

1. HALL request
2. CAR request to car id
3. scoreFor each car, pick min
4. step tick
5. arrive
6. Hall Panel: floor+dir
7. ElevatorController
8. Car Panel: floor
9. best car
10. ElevatorCar 0LOOK: sweep up/down
11. ElevatorCar 1
12. ElevatorCar N
13. move 1 floor toward next stopreverse only when none ahead
14. open doors / board
15. Classes:
16. Building --> ElevatorController --> * ElevatorCar (TreeSet up/down stops, direction, floor)
17. Request(floor, direction, type: HALL|CAR)
18. Controller.requestHall() = pick car by scoreFor(); Controller.requestCar() = car.addStop()
19. Car.step() = LOOK algorithm

