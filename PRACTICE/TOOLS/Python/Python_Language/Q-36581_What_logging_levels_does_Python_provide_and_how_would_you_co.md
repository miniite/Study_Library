---
type: "practice"
difficulty: "Intermediate"
---

# What logging levels does Python provide, and how would you configure different handlers for console and file?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Log Filtering Scripts](<../../../../TOOLS/Python/Python_Language/Log_Filtering_Scripts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question has two parts. First: name and explain Python's logging levels. Second: configure the logging module so that console output and file output behave differently — the classic real-world need where you want everything (DEBUG and up) written to a rotating log file for later forensic analysis, but only warnings and errors printed to the console so operators aren't drowned in noise.

What the interviewer is probing at the "intermediate experience" level:

Do you know the six standard levels and their numeric ordering?
Do you understand the logger → handler → formatter → filter pipeline, and specifically that a level can be set on both the logger and each handler, and both gates must pass?
Do you use logging.getLogger(__name__) (module-scoped loggers) rather than the root logger and logging.basicConfig?
Do you know practical concerns: propagation, avoiding duplicate handlers, RotatingFileHandler/TimedRotatingFileHandler, and structured/JSON logging for cloud stacks like the ones in the tags (Razorpay, Swiggy, CRED)?

#### Assumptions
I'll state: a standard CPython 3.8+ app, single process (I'll mention the multi-process caveat), and that "handlers for console and file" means two independent sinks with different thresholds and formats.

The logging levels

Python's standard library logging module defines six severity levels, each mapped to an integer. Higher number = more severe. A handler or logger set to level X emits records whose level is >= X.

Level	Numeric	When you use it
NOTSET	0	"Inherit from parent." Default of a fresh logger; the root's effective default is WARNING.
DEBUG	10	Fine-grained diagnostic detail — variable values, query text, branch taken. Off in prod console.
INFO	20	Normal, expected events — "server started", "order 123 placed".
WARNING	30	Something unexpected but recoverable — retrying, deprecated API used, cache miss storm. This is the default level of the root logger.
ERROR	40	An operation failed — exception caught, request 500'd.
CRITICAL	50	The app itself may not continue — DB unreachable at startup, out of disk.

You emit with logger.debug(...), .info(...), .warning(...), .error(...), .critical(...), plus logger.exception(...) (which is error + automatic traceback, call it only inside an except block). You can also define custom numeric levels with logging.addLevelName, but you rarely should.

Key mental model — a record passes through two independent level gates and possibly filters:

CODE
logger.level  →  handler.level  →  (formatter) →  sink

If the logger's effective level is above the record's level, the record is discarded immediately and never reaches any handler. If it passes, it is dispatched to every handler attached to that logger and, if propagate=True (the default), bubbles up to ancestor loggers' handlers too. Each handler then applies its own level threshold. So to get "DEBUG to file, WARNING to console," you set the logger to DEBUG (the loosest gate, so nothing is dropped early) and then set the file handler to DEBUG and the console handler to WARNING.

Configuring different handlers for console and file

The core objects: a StreamHandler (writes to sys.stderr/stdout, i.e. the console) and a FileHandler or, better, RotatingFileHandler (writes to disk with size-based rollover so logs don't fill the volume). Each handler gets its own Formatter and its own setLevel.

Two ways to wire it up: the programmatic API (explicit, good for illustrating understanding) and dictConfig (declarative, what you actually ship). I'll show both — see the code field.

The important, easy-to-miss details a strong candidate calls out:

Set the logger level to the lowest of the two handlers (here DEBUG). If you leave the logger at WARNING, the file handler will never see DEBUG/INFO records no matter what you set on it — the logger gate already dropped them.
Use getLogger(__name__), not the root logger. Named module loggers give you per-package control (logging.getLogger("payments").setLevel(DEBUG)) and clean names in output.
Configure once, at startup. Re-running the setup function adds duplicate handlers → every line logged twice. Guard with if not logger.handlers: or, cleaner, use dictConfig which resets configuration.
propagate=False on configured non-root loggers if you also configured the root, otherwise records hit both your handlers and the root's → duplicates.
Rotation: RotatingFileHandler(maxBytes=..., backupCount=...) for size-based, TimedRotatingFileHandler(when="midnight", backupCount=14) for daily. In containers/cloud you often skip the file and just log JSON to stdout and let the platform (CloudWatch, ELK, Loki) collect it.
Format the two sinks differently — a terse, human-readable line for the console; a verbose line with timestamp, module, line number, and thread for the file so on-call can grep.
Summary — what a strong answer emphasizes

Name all six levels with their numeric order and give WARNING-is-the-default and exception()-adds-traceback as the two "tells" of experience. Then articulate the two-gate model: logger level first (set it to the loosest, DEBUG), then per-handler setLevel — that single insight is the whole answer to "different behavior for console vs file." Show a StreamHandler at WARNING with a short format and a RotatingFileHandler at DEBUG with a verbose format, each with its own Formatter. Mention dictConfig as the production-grade way, getLogger(__name__), the duplicate-handler trap, propagation, and rotation/JSON-to-stdout for cloud deployments.

#### Common mistakes
/ follow-up probes: setting the handler level but forgetting the logger is still at WARNING (nothing hits the file); calling basicConfig after handlers exist (it's a no-op if the root already has handlers); adding handlers on every request/import (duplicate lines); using the root logger everywhere; and, at scale, ignoring that FileHandler from multiple processes can interleave/corrupt lines — the fix is QueueHandler/QueueListener or a single log-shipping sidecar.

**Comparison:**

```
Level	Numeric value	Reaches console (WARNING)	Reaches file (DEBUG)
DEBUG	10	No	Yes
INFO	20	No	Yes
WARNING	30	Yes	Yes
ERROR	40	Yes	Yes
CRITICAL	50	Yes	Yes
```

**Code:**

```

import logging from logging.handlers import RotatingFileHandler

def setup_logging(): logger = logging.getLogger("app") logger.setLevel(logging.DEBUG) # loosest gate: let everything reach the handlers logger.propagate = False # don't also hit the root logger's handlers

CODE
Copy
if logger.handlers:                   # idempotent: never add handlers twice
    return logger

# --- Console handler: only WARNING+, terse human-readable format ---
console = logging.StreamHandler()     # defaults to sys.stderr
console.setLevel(logging.WARNING)
console.setFormatter(logging.Formatter("%(levelname)s: %(message)s"))

# --- File handler: DEBUG+, verbose format, size-based rotation ---
file_handler = RotatingFileHandler(
    "app.log", maxBytes=5 * 1024 * 1024, backupCount=5  # 5 MB x 5 files
)
file_handler.setLevel(logging.DEBUG)
file_handler.setFormatter(logging.Formatter(
    "%(asctime)s %(levelname)-8s [%(name)s:%(lineno)d] %(message)s"
))

logger.addHandler(console)
logger.addHandler(file_handler)
return logger

log = setup_logging() log.debug("connecting to db host=%s", "10.0.0.5") # file only log.info("order 123 placed") # file only log.warning("payment retry 2/3") # file AND console try: 1 / 0 except ZeroDivisionError: log.exception("charge failed") # ERROR + traceback, file AND console

CODE
Copy

```python
# Production-grade equivalent using dictConfig (declarative, resets config each call)
import logging.config

logging.config.dictConfig({
    "version": 1,
    "disable_existing_loggers": False,
    "formatters": {
        "short":  {"format": "%(levelname)s: %(message)s"},
        "detail": {"format": "%(asctime)s %(levelname)-8s [%(name)s:%(lineno)d] %(message)s"},
    },
    "handlers": {
        "console": {"class": "logging.StreamHandler",
                    "level": "WARNING", "formatter": "short"},
        "file":    {"class": "logging.handlers.RotatingFileHandler",
                    "level": "DEBUG", "formatter": "detail",
                    "filename": "app.log", "maxBytes": 5_242_880, "backupCount": 5},
    },
    "loggers": {
        "app": {"handlers": ["console", "file"], "level": "DEBUG", "propagate": False},
    },
})
```

**Example input / output:**

```

Given the configuration above, the same five log calls produce different output on each sink:

Console (stderr) — WARNING threshold, terse:

CODE
Copy
WARNING: payment retry 2/3
ERROR: charge failed
Traceback (most recent call last):
  File "app.py", line 40, in <module>
    1 / 0
ZeroDivisionError: division by zero

app.log (file) — DEBUG threshold, verbose, ALL five records:

CODE
Copy
2026-07-07 10:15:02,001 DEBUG    [app:38] connecting to db host=10.0.0.5
2026-07-07 10:15:02,002 INFO     [app:39] order 123 placed
2026-07-07 10:15:02,003 WARNING  [app:40] payment retry 2/3
2026-07-07 10:15:02,004 ERROR    [app:44] charge failed
Traceback (most recent call last):
  File "app.py", line 42, in <module>
    1 / 0
ZeroDivisionError: division by zero

The DEBUG and INFO lines appear only in the file because the console handler's own level (WARNING) drops them, while the logger's level (DEBUG) let them through to the file handler.
```

**Flow:**

1. no
2. yes
3. = 30
4. < 30
5. = 10
6. logger.info('order placed')
7. logger.levelDEBUG (10)?record >= 10
8. dropped, never dispatched
9. dispatch to each handler
10. console handlerlevel WARNING (30)?
11. file handlerlevel DEBUG (10)?
12. console: short format
13. skipped on console
14. app.log: detailed format

