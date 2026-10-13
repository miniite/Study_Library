---
type: "practice"
difficulty: "Intermediate"
---

# How would you debug a Python program? Compare using print statements, pdb, and logging.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Request Logging for Models](<../../../../TOOLS/Python/Python_Language/Request_Logging_for_Models.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question asks two things: describe a systematic approach to debugging a Python program, and compare three concrete tools — print statements, the pdb debugger, and the logging module. The interviewer is probing whether you debug methodically (form a hypothesis, isolate, verify) rather than randomly sprinkling prints, and whether you know when each tool is the right one. At intermediate level they expect you to know pdb/breakpoint() fluently and to understand that logging is not just "print with extra steps" — it's the tool that survives into production.

#### Assumptions
a backend/full-stack service, Python 3.7+ (so breakpoint() exists). Clarifying questions I'd ask: "Is the bug reproducible locally, or only in production?" and "Is it a crash (exception + traceback) or wrong-but-silent behaviour?" — because those steer me to different tools (a local crash → pdb post-mortem; an intermittent prod issue → structured logging).

Plan: give a repeatable debugging method, then compare the three tools on control, cost, and where they live, then state a decision rule.

A systematic debugging method (before reaching for any tool)
Reproduce reliably. A bug you can trigger on demand is half-solved. Find the minimal input that reproduces it.
Read the traceback bottom-up. The last line is the exception; the frames above it are the call path. The traceback usually points within a few lines of the fault.
Form a hypothesis and bisect. "I think total is wrong before it reaches save()." Then check the value at the midpoint of the code path to halve the search space each time (binary search over the execution).
Inspect state at the suspect point — with whichever tool fits (below).
Fix, then verify with a regression test so the bug can't silently return.

The tool is just how you observe state at step 4. The method matters more than the tool.

The three tools compared
print

The zero-setup option: drop print(f"{total=}") (the = f-string trick auto-labels the value) and run. It's ubiquitous and needs no imports. Its weaknesses: it's manual and destructive — you edit source, re-run the whole program to reach the line, then must remember to delete every print. You only see what you thought to print before running; if you guessed the wrong variable you edit and re-run again. It writes to stdout unconditionally, so it's easy to accidentally ship a debug print to production. Fine for a quick local one-off; poor for anything iterative or long-running.

pdb (the interactive debugger)

pdb pauses execution and gives you a live REPL inside the frame. Since Python 3.7 you just write breakpoint() where you want to stop (it respects the PYTHONBREAKPOINT env var, so you can disable all breakpoints with PYTHONBREAKPOINT=0 without editing code). At the (Pdb) prompt you can: n (next line), s (step into a call), c (continue), p expr/pp (print/pretty-print any expression), l (list source), w (where — the stack), up/down (move frames), and evaluate arbitrary Python against live state. Crucially you can inspect variables you didn't anticipate, and set a conditional breakpoint (b file:42, x > 100) to stop only when things go wrong.

Its killer feature for crashes is post-mortem debugging: run python -m pdb -c continue script.py or in a REPL import pdb; pdb.pm() after an exception, and you're dropped into the frame at the moment it blew up, with all locals intact — no re-running, no guessing. The trade-off: pdb is interactive, so it's for local/dev work — you can't attach it to a running production request, and it halts execution.

logging

logging is the production-grade choice. You emit messages at severity levels (DEBUG, INFO, WARNING, ERROR, CRITICAL); a single config sets the threshold, so DEBUG chatter can exist in code but be silenced in prod by setting the level to INFO. Output routes to handlers (console, rotating file, syslog, JSON to a log aggregator) and formatters add timestamp, level, logger name, and — importantly — you can attach request IDs / user IDs so you can trace one request across a distributed system. logger.exception(...) inside an except block logs the full traceback automatically. Unlike prints, logging statements are meant to stay in the code permanently; they're not debris, they're observability. The cost is a little upfront configuration.

Decision rule
Quick, local, throwaway peek → print (or better, a temporary logger.debug).
"I don't know what's happening here, let me poke around live state / a crash" → pdb / breakpoint() / post-mortem pdb.pm().
"It fails intermittently in production and I need a durable trail" → logging with levels + structured context, plus your log aggregator.

A mature engineer's honest answer: I use logging as the always-on baseline (so production is observable), pdb when I need to interrogate live state during development, and print only for the most trivial throwaway checks — and I never let a print reach production.

Summary — what a strong answer emphasizes

Lead with method (reproduce → read traceback → bisect → inspect → fix + regression test), then position the three tools by control and lifespan: print = zero-setup but manual/destructive/local; pdb = interactive, inspect-anything, post-mortem for crashes, but dev-only; logging = level-gated, routable, permanent, the only one that works in production. The senior signal is knowing breakpoint()/pdb.pm(), and knowing that logging is observability, not "fancier print."

#### Common mistakes
/ follow-up probes: saying "I just add prints" with no method; not knowing breakpoint() (using the older import pdb; pdb.set_trace() is fine but dated); thinking logging is only about verbosity, not routing/levels/context. A likely follow-up — "how do you debug something that only happens in prod?" — answer: you can't attach pdb there, so you lean on rich logging (correlation IDs, DEBUG toggled per-request), metrics, and reproducing locally from the logged inputs.

**Comparison:**

```
Dimension	print	pdb / breakpoint()	logging
Setup cost	None	None (breakpoint())	One-time config
Interactivity	None (static output)	Full REPL: inspect/step/eval	None (records only)
See un-anticipated vars?	No (re-edit + re-run)	Yes, any expression live	Only what you logged
Severity levels / filtering	No	N/A	Yes (DEBUG…CRITICAL)
Routing (file/syslog/JSON)	stdout only	N/A	Yes, via handlers
Meant to stay in code?	No (delete after)	No (remove after)	Yes (permanent)
Works in production?	Risky/noisy	No (halts, interactive)	Yes — the right tool
Post-mortem on crash?	No	Yes (pdb.pm())	Captures traceback (.exception)
```

**Code:**

```python
# 1) print — quick + destructive (note the f-string '=' auto-label trick)
def total_price(items):
    total = sum(i["price"] for i in items)
    print(f"{total=}")            # temporary; must remember to delete
    return total

# 2) pdb — interactive: drop a breakpoint, then poke at live state
def total_price(items):
    total = 0
    for i in items:
        breakpoint()              # (Pdb) prompt: n, s, c, p i, pp items, w, b, up/down
        total += i["price"]
    return total
# Post-mortem after a crash (no re-run needed):
#   $ python -m pdb -c continue app.py
#   or in a REPL after the exception:  import pdb; pdb.pm()

# 3) logging — production-grade, level-gated, permanent
import logging

logging.basicConfig(
    level=logging.INFO,           # DEBUG lines exist in code but are silenced in prod
    format="%(asctime)s %(levelname)s %(name)s %(message)s",
)
logger = logging.getLogger(__name__)

def total_price(items):
    logger.debug("computing total for %d items", len(items))  # lazy %-format, not f-string
    try:
        total = sum(i["price"] for i in items)
    except KeyError:
        logger.exception("item missing 'price' field")         # logs full traceback
        raise
    logger.info("total=%s", total)
    return total
```

**Example input / output:**

```text
# --- pdb session (interactive) ---
$ python app.py
> app.py(5)total_price()
-> total += i["price"]
(Pdb) p i
{'name': 'pen', 'price': 10}
(Pdb) pp items
[{'name': 'pen', 'price': 10}, {'name': 'ink', 'prce': 5}]   # typo 'prce' spotted!
(Pdb) c        # continue -> next iteration crashes on KeyError, root cause found

# --- logging output (production, level=INFO so DEBUG is hidden) ---
2026-07-07 11:02:14,551 INFO  app total=15
2026-07-07 11:02:19,004 ERROR app item missing 'price' field
Traceback (most recent call last):
  ...
KeyError: 'price'
```

**Flow:**

1. No, only in prod
2. Yes
3. Crash
4. Silent
5. Yes, one-off
6. No, need to explore
7. Bug observed
8. Reproducible locally?
9. Rely on logging:levels + correlation IDs,reproduce from logged inputs
10. Crash w/ tracebackor silent wrong value?
11. pdb post-mortem: pdb.pm()land in the failing frame
12. Know exactly whichvalue to check?
13. quick print / logger.debug
14. breakpoint(): step, inspect,bisect the execution
15. Fix + add regression test

