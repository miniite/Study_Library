---
type: "practice"
difficulty: "Basic"
---

# Design a Chess-clock or a general Timer/Stopwatch class. What is the interface?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Classic LLD Problems](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Classic_LLD_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is being asked? Design a small, well-behaved class that measures elapsed time — a Stopwatch/Timer — and then extend it to a chess clock (two players, each with their own countdown, only one running at a time, whoever's clock hits zero loses). This is a "basic concept" LLD warm-up, so the interviewer is probing: do I pick the right clock source, do I get the state machine right (IDLE → RUNNING → PAUSED → STOPPED), is the interface minimal and clear, and do I avoid the two classic beginner bugs. It is not about algorithms; it's about clean state and correct time math.

Clarifying questions I'd ask:

Do we need pause/resume, or just start/stop? (I'll support pause/resume — it's the interesting part.)
What granularity — milliseconds is fine for a chess clock; nanoseconds if we're profiling code.
Should the class be thread-safe? For a UI chess clock, no (one UI thread); I'll mention how to make it safe.
Chess-clock rules: two players, increment/Fischer time (add a few seconds per move)? I'll add that as an option.

The two bugs to avoid (this is what they're really testing):

Never use wall-clock time (System.currentTimeMillis()) to measure elapsed durations. Wall clocks can jump backwards (NTP sync, DST, user changing the clock), producing negative or wrong elapsed times. Use a monotonic source: System.nanoTime() in Java, performance.now() in JS, time.monotonic() in Python. Monotonic clocks only move forward.
Accumulate elapsed time across pauses. A stopwatch that's paused and resumed must add up each running segment. The trick: keep a running accumulated total, and on each start record startTick; on pause/stop, add now - startTick into accumulated.
1. Stopwatch — interface and state machine

The class has three fields: a state, an accumulated elapsed (of all past running segments), and the startTick of the current running segment. Its state machine:

CODE
IDLE --start--> RUNNING --pause--> PAUSED --start--> RUNNING --stop--> STOPPED

Interface (the minimal, clear surface):

Method	Meaning
start()	begin (or resume) counting; records startTick
pause()	freeze; folds current segment into accumulated
stop()	pause + mark terminal
reset()	back to IDLE, accumulated = 0
elapsed()	total running time so far (works while running or paused)

The one subtle method is elapsed(): it must return accumulated + (running ? now - startTick : 0) so it's correct live, without mutating state.

2. Chess clock — composed from two countdown timers

A chess clock is two countdown clocks plus a turn pointer. Only the active player's clock ticks. Pressing your button = pause my clock, add my increment, start opponent's clock. When a clock's remaining() reaches zero, that player flags (loses).

I model each side as a CountdownClock (a Stopwatch with an initial budget: remaining = budget - elapsed()), and the ChessClock owns two of them plus activePlayer. This composition — reuse the stopwatch, don't reinvent it — is exactly the design instinct the interviewer wants.

Chess-clock interface:

Method	Meaning
press()	end my turn: pause mine (+increment), start opponent's
remaining(player)	milliseconds left for a player
flagged()	which player (if any) has run out of time
pause()/resume()	referee pauses the whole game
3. Edge cases and follow-ups I'd volunteer
Idempotency / illegal transitions: calling start() when already RUNNING should be a no-op (or throw) — never reset startTick, or you'd lose the running segment. Same for double-pause().
Zero / negative budget: a countdown that's already at 0 stays flagged.
Fischer vs Bronstein increment: Fischer adds a fixed bonus on every move; Bronstein only refunds up to the time you actually used. Easy to add as a strategy.
Thread safety: if a background timer thread reads remaining() while the UI thread calls press(), guard the state mutations with a lock (or synchronized), or use an actor/single-thread event loop. For a basic UI clock, a single thread avoids all of it.
Precision vs display: measure in nanos/millis internally, but the display should round to whole seconds and update via a repaint timer — don't confuse "measurement precision" with "UI refresh rate".
Summary — what a strong answer emphasizes

Use a monotonic clock (System.nanoTime), model an explicit state machine (IDLE/RUNNING/PAUSED/STOPPED), and compute elapsed() as accumulated + (running ? now - startTick : 0) so it stays correct across pause/resume and reads correctly while live. Build the chess clock by composing two countdown stopwatches with a turn pointer, where press() pauses mine (plus increment) and starts the opponent's, and reaching zero flags a loss. The whole answer hinges on the two gotchas: never measure durations with wall-clock time, and always accumulate paused segments. Follow-up probes: handle illegal transitions idempotently, add Fischer/Bronstein increment, and note the thread-safety story.

**Comparison:**

```
Clock source	Monotonic?	Use for	Risk if misused
System.nanoTime() / time.monotonic() / performance.now()	Yes	measuring elapsed/durations	none — correct choice
System.currentTimeMillis() / wall clock	No	timestamps ("when did X happen")	jumps on NTP/DST → negative or wrong elapsed
State	Legal next states
IDLE	RUNNING (start)
RUNNING	PAUSED (pause), STOPPED (stop)
PAUSED	RUNNING (start/resume), STOPPED, IDLE (reset)
STOPPED	IDLE (reset)
```

**Code:**

```java
// ---------- Stopwatch ----------
public class Stopwatch {
    enum State { IDLE, RUNNING, PAUSED, STOPPED }
    private State state = State.IDLE;
    private long accumulatedNanos = 0;   // sum of all past running segments
    private long startTick = 0;          // System.nanoTime() at last start

    public void start() {                // start or resume
        if (state == State.RUNNING) return;          // idempotent, don't lose the segment
        startTick = System.nanoTime();               // MONOTONIC source
        state = State.RUNNING;
    }
    public void pause() {
        if (state != State.RUNNING) return;
        accumulatedNanos += System.nanoTime() - startTick;  // fold segment in
        state = State.PAUSED;
    }
    public void stop()  { pause(); state = State.STOPPED; }
    public void reset() { accumulatedNanos = 0; state = State.IDLE; }

    public long elapsedMillis() {        // correct while running OR paused
        long live = (state == State.RUNNING) ? System.nanoTime() - startTick : 0;
        return (accumulatedNanos + live) / 1_000_000;
    }
    public State state() { return state; }
}

// ---------- Countdown = Stopwatch with a budget ----------
class CountdownClock extends Stopwatch {
    private final long budgetMillis;
    CountdownClock(long budgetMillis) { this.budgetMillis = budgetMillis; }
    long remaining() { return Math.max(0, budgetMillis - elapsedMillis()); }
    boolean flagged() { return remaining() == 0; }
}

// ---------- Chess clock = two countdowns + turn ----------
public class ChessClock {
    private final CountdownClock[] clocks = new CountdownClock[2];
    private final long incrementMillis;   // Fischer increment
    private int active = 0;

    public ChessClock(long budgetMillis, long incrementMillis) {
        clocks[0] = new CountdownClock(budgetMillis);
        clocks[1] = new CountdownClock(budgetMillis);
        this.incrementMillis = incrementMillis;
    }
    public void startGame() { clocks[active].start(); }

    public void press() {                 // end my move, start opponent's
        if (clocks[active].flagged()) return;         // already lost
        clocks[active].pause();
        // (Fischer increment would extend this player's budget; omitted for brevity)
        active ^= 1;                                   // switch turn
        clocks[active].start();
    }
    public long remaining(int player) { return clocks[player].remaining(); }
    public Integer flagged() {            // returns loser index or null
        if (clocks[0].flagged()) return 0;
        if (clocks[1].flagged()) return 1;
        return null;
    }
}
```

**Example input / output:**

```text
Stopwatch dry-run (pause/resume accumulation):
  t=0ms   start()        -> RUNNING, startTick=0
  t=300ms elapsedMillis()-> 300   (accumulated 0 + live 300)
  t=300ms pause()        -> PAUSED, accumulated=300
  t=800ms elapsedMillis()-> 300   (paused: no live time added)   <-- key correctness point
  t=800ms start()        -> RUNNING, startTick=800
  t=1000ms elapsedMillis()-> 500  (accumulated 300 + live 200)
  double start() at RUNNING -> no-op (startTick unchanged, segment preserved)

ChessClock dry-run (budget=5000ms, increment=0):
  startGame()            -> P0 clock running, P1 paused
  after 2000ms, P0.press()-> P0 remaining=3000, now P1 running
  after 4000ms, P1.press()-> P1 remaining=1000, now P0 running
  ... P0 lets 3001ms pass without pressing
  P0.remaining()=0 -> flagged() returns 0  (Player 0 loses on time)
```

**Flow:**

1. start()
2. pause() / accumulatesegment
3. start() / new startTick
4. stop()
5. stop()
6. reset()
7. reset()
8. IDLE
9. RUNNING
10. PAUSED
11. STOPPED

