---
type: "practice"
difficulty: "Other"
---

# Drive and Cancel a Stepwise Coroutine

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Coroutines](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Coroutines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a coroutine — a routine that runs in stages. At each stage the coroutine awaits a value: it hands an awaitable out to a driver, the driver "resolves" that awaitable and feeds the resolved value back in, and the coroutine then proceeds to its next stage. Eventually the coroutine finishes by returning a final value, or it may fail by raising its own error.

Build a driver that runs such a coroutine to completion and also supports cancellation.

Cancellation may be requested while the coroutine is still running — that is, while it is suspended at one of its await points. When cancellation is requested before the coroutine has finished, the driver must inject a cancellation signal — the exact string "Cancelled" (not an error object) — into the coroutine at its current suspension point, as though that await had raised "Cancelled". Then:

If the coroutine catches this signal (for example with a try/except or try/catch that recovers) and goes on to yield another value or return normally, the run succeeds and produces that next value.
If the coroutine does not catch it, the signal propagates out and the run fails with "Cancelled".

Once cancellation has driven the coroutine to completion, no further stages run. Cancelling a coroutine that has already finished does nothing.

If cancellation is never requested (or is requested only after the coroutine has already finished), the driver runs the coroutine normally: it succeeds with the coroutine's returned value, or fails with the coroutine's own raised error.

Report the outcome as either a resolved value or a rejected error.

Deterministic model (for a reproducible specification). Model each await as resolving to the value the coroutine produced (identity resolution), and describe the cancellation timing as a 1-based index k: cancellation is requested at the coroutine's k-th suspension, or never. A native generator exposes exactly the two operations the driver needs: resume with a value (.next(v) / .send(v)) and inject a signal (.throw(e)). In a language without generators, a coroutine is modeled as an explicit state machine exposing the same two operations — which is precisely how compilers lower generators.

Input: a coroutine and a cancel index k (a positive integer, or never).

Output: { resolved: value } or { rejected: error }.

#### Cons
traints:

The coroutine performs a finite number of stages, 0 <= stages <= 10^4.
Cancellation is injected at most once.
k is a positive integer (1-based suspension index) or never.
Yielded/returned values and error payloads are arbitrary.

#### Example
s & test cases
TEXT
Input: coroutine returns 42 immediately (0 stages); cancel at k = 1 -> Output: { resolved: 42 }
Why: the coroutine finishes before any suspension, so there is nothing to cancel; cancelling a finished coroutine is a no-op.

Input: coroutine awaits one value then raises its own error "Boom"; cancel = never -> Output: { rejected: "Boom" }
Why: no cancellation occurs, so the coroutine's own error propagates out as a rejection.

Input: coroutine awaits two values then returns "Done"; cancel at k = 1 -> Output: { rejected: "Cancelled" }
Why: the "Cancelled" signal injected at the first suspension is not caught, so it propagates and the run rejects.

Input: coroutine wraps its awaits in try/catch that returns "Recovered" on cancellation; cancel at k = 1 -> Output: { resolved: "Recovered" }
Why: the coroutine catches the injected signal and completes normally, so the run resolves with the recovery value.

Input: coroutine awaits three values then returns their sum; cancel = never -> Output: { resolved: 6 }
Why: with no cancellation the driver simply resolves each await (1, 2, 3) and the coroutine returns normally.

Input: coroutine awaits two values then returns "Late"; cancel at k = 5 -> Output: { resolved: "Late" }
Why: the coroutine finishes before reaching the 5th suspension, so cancellation never fires (edge case: cancel index beyond the coroutine's length).

#### Approach

The heart of the solution is a resume/inject driver loop over a coroutine that supports two operations: resume with a resolved value and inject a signal. Native generators provide these directly — .next(v)/.send(v) and .throw(e) — so JavaScript and Python drive a real generator. Languages without generators model the coroutine as an explicit state machine (a program counter plus captured locals) exposing the identical resume/raise interface; this is exactly how a compiler lowers a generator, so the driver algorithm is unchanged. The loop: start the coroutine; on each YIELD (suspension) either inject "Cancelled" — but only once, at the target suspension index — or resume with the resolved value; a RETURN resolves the run, while a THROW (the coroutine's own error, or an uncaught injected "Cancelled") rejects it. If the coroutine catches the injected signal and continues, its next return/yield is what the run resolves with. The underlying pattern is coroutine/continuation driving with one-shot exception injection (cooperative cancellation).

Complexity

Time: O(s) where s is the number of stages actually executed — the driver performs one resume or one inject per stage, each O(1). Space: O(1) auxiliary in the driver (a counter and a flag), beyond the coroutine's own captured state.

Python
PYTHON
class Cancelled(Exception):
    """The signal injected into a coroutine on cancellation; its message is 'Cancelled'."""
    def __init__(self):
        super().__init__("Cancelled")

def run_cancellable(coro, cancel_at=None):
    """
    Drive a coroutine (generator) that yields awaitable values. Each yielded value
    is resolved (to itself) and sent back in via .send(); cancellation is injected
    via .throw() at the coroutine's cancel_at-th suspension (1-based), or never.
    Returns ('resolved', value) or ('rejected', error).
    """
    awaited = 0
    cancelled = False
    action, payload = 'send', None            # first .send() must prime with None
    while True:
        try:
            if action == 'send':
                yielded = coro.send(payload)
            else:                             # 'throw' -> inject the signal
                yielded = coro.throw(Cancelled())
        except StopIteration as stop:         # coroutine returned normally
            return ('resolved', stop.value)
        except Cancelled:                     # injected signal was NOT caught
            return ('rejected', 'Cancelled')
        except Exception as err:              # coroutine raised its own error
            return ('rejected', err)
        # coroutine is suspended at an await point holding `yielded`
        if action == 'send':
            awaited += 1
        if not cancelled and cancel_at is not None and awaited == cancel_at:
            cancelled = True
            action, payload = 'throw', None   # inject "Cancelled" exactly once
        else:
            action, payload = 'send', yielded # resolve the awaitable to itself

# --- sample coroutines used by the worked cases ---
def returns_immediately():
    return 42
    yield            # makes this a generator (never reached)

def raises_own_error():
    msg = yield "hi"
    raise RuntimeError("Boom: " + msg)

def two_steps_then_done():
    yield 1
    yield 2
    return "Done"

def swallows_cancel():
    try:
        yield 1
        yield 2
        return "unreached"
    except Cancelled:
        return "Recovered"
JavaScript
JAVASCRIPT
// The signal injected into a coroutine on cancellation: the exact string
// "Cancelled" (not an Error object), per spec.

#### cons
t CANCELLED = "Cancelled";

/**
 * Drive a coroutine (generator) that yields awaitable values. Each yielded value
 * is resolved (to itself) and fed back via .next(); cancellation is injected via
 * .throw() at the coroutine's `cancelAt`-th suspension (1-based), or null=never.
 * Returns { status: "resolved" | "rejected", value }.
 */
function runCancellable(coro, cancelAt = null) {
    let awaited = 0;
    let cancelled = false;
    let action = "next";          // "next" -> resume with value, "throw" -> inject
    let payload;

    while (true) {
        let step;
        try {
            step = action === "next" ? coro.next(payload) : coro.throw(CANCELLED);
        } catch (err) {
            // Uncaught cancellation, or the coroutine's own error, propagated out.
            return { status: "rejected", value: err };
        }
        if (step.done) {
            return { status: "resolved", value: step.value };
        }
        // Coroutine is suspended at an await point holding step.value.
        if (action === "next") awaited++;
        if (!cancelled && cancelAt !== null && awaited === cancelAt) {
            cancelled = true;
            action = "throw";      // inject "Cancelled" exactly once
        } else {
            action = "next";
            payload = step.value;  // resolve the awaitable to itself
        }
    }
}

// --- sample coroutines used by the worked cases ---
function* returnsImmediately() { return 42; }
function* raisesOwnError()     { const m = yield "hi"; throw "Boom: " + m; }
function* twoStepsThenDone()   { yield 1; yield 2; return "Done"; }
function* swallowsCancel() {
    try { yield 1; yield 2; return "unreached"; }
    catch (e) { return "Recovered"; }        // e === "Cancelled"
}
Java
JAVA
class CancellableDriver {
    enum Kind { YIELD, RETURN, THROW }

    static final class Result {
        final Kind kind; final long value; final String error;
        private Result(Kind k, long v, String e) { kind = k; value = v; error = e; }
        static Result yield (long v) { return new Result(Kind.YIELD,  v, null); }
        static Result ret   (long v) { return new Result(Kind.RETURN, v, null); }
        static Result thr   (String e) { return new Result(Kind.THROW, 0, e);   }
    }

    // A coroutine exposes the same two operations a native generator does.
    interface Coroutine {
        Result resume(long sent);
        Result raise(String signal);
    }

    static final class Outcome {
        final boolean resolved; final long value; final String error;
        Outcome(boolean r, long v, String e) { resolved = r; value = v; error = e; }
    }

    // Drive to completion; inject "Cancelled" once at the cancelAt-th suspension
    // (cancelAt <= 0 means never).
    static Outcome runCancellable(Coroutine c, int cancelAt) {
        int awaited = 0; boolean cancelled = false;
        Result r = c.resume(0);                          // start
        while (true) {
            if (r.kind == Kind.RETURN) return new Outcome(true,  r.value, null);
            if (r.kind == Kind.THROW)  return new Outcome(false, 0,       r.error);
            awaited++;                                   // YIELD: suspended
            if (!cancelled && cancelAt > 0 && awaited == cancelAt) {
                cancelled = true;
                r = c.raise("Cancelled");
            } else {
                r = c.resume(r.value);                   // resolve awaitable to itself
            }
        }
    }

    // Sample coroutine as an explicit state machine.
    static final class SampleCoroutine implements Coroutine {
        final int steps; int pc = 0;
        final boolean throwsOwn; final String ownError; final long finalValue;
        final boolean catchesCancel;                     final long recoverValue;
        SampleCoroutine(int steps, boolean throwsOwn, String ownError, long finalValue,
                        boolean catchesCancel, long recoverValue) {
            this.steps = steps; this.throwsOwn = throwsOwn; this.ownError = ownError;
            this.finalValue = finalValue; this.catchesCancel = catchesCancel;
            this.recoverValue = recoverValue;
        }
        public Result resume(long sent) {
            if (pc < steps) { pc++; return Result.yield(pc); }
            return throwsOwn ? Result.thr(ownError) : Result.ret(finalValue);
        }
        public Result raise(String signal) {
            return catchesCancel ? Result.ret(recoverValue) : Result.thr("Cancelled");
        }
    }
}
C++
CPP
#include <string>
#include <utility>

enum class Kind { YIELD, RETURN, THROW };

struct Result {
    Kind kind;
    long value = 0;      // carried on YIELD / RETURN
    std::string error;   // carried on THROW
    static Result Yield (long v)               { return {Kind::YIELD,  v, ""}; }
    static Result Return(long v)               { return {Kind::RETURN, v, ""}; }
    static Result Throw (const std::string& e) { return {Kind::THROW,  0, e};  }
};

// A coroutine exposes exactly the two operations a native generator does:
// resume-with-value and inject-a-signal.
struct Coroutine {
    virtual ~Coroutine() = default;
    virtual Result resume(long sent) = 0;
    virtual Result raise(const std::string& signal) = 0;
};

struct Outcome { bool resolved; long value; std::string error; };

// Drive the coroutine to completion, injecting "Cancelled" once at the
// cancelAt-th suspension (cancelAt <= 0 means never).
Outcome runCancellable(Coroutine& c, int cancelAt) {
    int awaited = 0; bool cancelled = false;
    Result r = c.resume(0);                          // start
    while (true) {
        if (r.kind == Kind::RETURN) return {true,  r.value, ""};
        if (r.kind == Kind::THROW)  return {false, 0,       r.error};
        ++awaited;                                   // YIELD: suspended
        if (!cancelled && cancelAt > 0 && awaited == cancelAt) {
            cancelled = true;
            r = c.raise("Cancelled");                // inject exactly once
        } else {
            r = c.resume(r.value);                   // resolve awaitable to itself
        }
    }
}

// Sample coroutine as an explicit state machine.
struct SampleCoroutine : Coroutine {
    int steps, pc = 0;
    bool throwsOwn;     std::string ownError; long finalValue;
    bool catchesCancel;                       long recoverValue;
    SampleCoroutine(int steps, bool throwsOwn, std::string ownError, long finalValue,
                    bool catchesCancel, long recoverValue)
        : steps(steps), throwsOwn(throwsOwn), ownError(std::move(ownError)),
          finalValue(finalValue), catchesCancel(catchesCancel), recoverValue(recoverValue) {}

    Result resume(long /*sent*/) override {
        if (pc < steps) { ++pc; return Result::Yield(pc); }
        return throwsOwn ? Result::Throw(ownError) : Result::Return(finalValue);
    }
    Result raise(const std::string& /*signal*/) override {
        return catchesCancel ? Result::Return(recoverValue) : Result::Throw("Cancelled");
    }
};
C
C
#include <stdbool.h>
#include <stddef.h>

/* A coroutine step reports one of three outcomes. */
typedef enum { KIND_YIELD, KIND_RETURN, KIND_THROW } Kind;

typedef struct {
    Kind kind;
    long value;         /* carried on YIELD / RETURN */

#### cons
t char *error;  /* carried on THROW          */
} Result;

/* A sample coroutine modeled as an explicit state machine (a program counter
   plus its captured behavior). It performs `steps` awaits, then either returns
   `finalValue` or throws `ownError`. If `catchesCancel` is true, an injected
   "Cancelled" signal is swallowed and it returns `recoverValue` instead. */
typedef struct {
    int steps, pc;
    bool throwsOwn;   const char *ownError;  long finalValue;
    bool catchesCancel;                      long recoverValue;
} Coro;

/* Resume the coroutine with a resolved value (the value itself is unused here,
   since awaits resolve to their own token in the deterministic model). */
static Result coro_resume(Coro *c, long sent) {
    (void)sent;
    Result r;
    if (c->pc < c->steps) {                 /* suspend at the next await point  */
        c->pc += 1;
        r.kind = KIND_YIELD; r.value = c->pc; r.error = NULL;
        return r;
    }
    if (c->throwsOwn) { r.kind = KIND_THROW;  r.value = 0;             r.error = c->ownError; }
    else              { r.kind = KIND_RETURN; r.value = c->finalValue; r.error = NULL;        }
    return r;
}

/* Inject a signal into the coroutine at its current suspension. */
static Result coro_raise(Coro *c, const char *signal) {
    (void)signal;
    Result r;
    if (c->catchesCancel) { r.kind = KIND_RETURN; r.value = c->recoverValue; r.error = NULL;        }
    else                  { r.kind = KIND_THROW;  r.value = 0;               r.error = "Cancelled"; }
    return r;
}

typedef struct { bool resolved; long value; const char *error; } Outcome;

/* Drive the coroutine to completion, injecting "Cancelled" exactly once at the
   cancelAt-th suspension (cancelAt <= 0 means never). */
Outcome runCancellable(Coro *c, int cancelAt) {
    int awaited = 0; bool cancelled = false;
    Outcome out;
    Result r = coro_resume(c, 0);           /* start the coroutine */
    for (;;) {
        if (r.kind == KIND_RETURN) { out.resolved = true;  out.value = r.value; out.error = NULL;   return out; }
        if (r.kind == KIND_THROW)  { out.resolved = false; out.value = 0;       out.error = r.error; return out; }
        /* KIND_YIELD: suspended at an await point */
        awaited += 1;
        if (!cancelled && cancelAt > 0 && awaited == cancelAt) {
            cancelled = true;
            r = coro_raise(c, "Cancelled");
        } else {
            r = coro_resume(c, r.value);    /* resolve the awaitable to itself   */
        }
    }
}

#### Follow-up
questions
Allow cancellation to be requested by multiple independent observers, where only the first request takes effect and later ones are no-ops.
Make awaits genuinely asynchronous — resolve each yielded value through a real promise/future and race every await against the cancel signal (the classic promise-driven form of this problem).
Add a per-stage timeout that auto-injects cancellation if any single await exceeds a time budget.
Guarantee that resources acquired by the coroutine are always released (finally/cleanup blocks run) even when the injected cancellation propagates uncaught.

