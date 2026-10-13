---
type: "practice"
difficulty: "Intermediate"
---

# How would you create a custom exception class and when is it worth doing in a backend service?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Functions Modules and Reuse](<../../../../TOOLS/Python/Python_Language/Functions_Modules_and_Reuse.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked: Two parts — (1) the mechanics: how do you define a custom exception class in Python? (2) the judgment: when is a custom exception worth introducing in a backend service versus just reusing a built-in like ValueError?

What the interviewer is probing: For a fintech/SaaS backend role they want to see that you treat exceptions as part of your API/domain design, not as an afterthought. Do you know to subclass Exception, add useful attributes, build a small hierarchy, and map errors cleanly to HTTP responses / retry logic? Do you know when a custom class earns its keep versus when it's noise?

#### Assumptions
Python 3, a service exposing an HTTP or RPC API (think payments — PhonePe, Razorpay, Zeta). I'll cover the "how" quickly then spend most of the time on the "when."

Plan: define the class properly → add attributes → build a hierarchy → show how it drives error handling/HTTP mapping → give a clear "worth it / not worth it" decision rule.

How to create a custom exception

The rule: subclass Exception (never BaseException — that's reserved for shutdown signals like KeyboardInterrupt). The minimal form is a one-liner:

PYTHON
class PaymentError(Exception):
    """Base class for all payment-domain errors."""

That already works: raise PaymentError("gateway timeout"). But a good custom exception carries structured, machine-usable data, not just a message string:

PYTHON
class PaymentDeclined(PaymentError):
    def __init__(self, order_id: str, reason_code: str):
        self.order_id = order_id
        self.reason_code = reason_code
        super().__init__(f"payment declined for {order_id}: {reason_code}")

Key points that separate a strong answer from a weak one:

Always call super().__init__(message) so str(exc) and logging work correctly.
Store fields as attributes (order_id, reason_code, http_status) so the handler can act on them without string-parsing the message.
Prefer a base class + specific subclasses so callers can catch broadly (except PaymentError) or narrowly (except PaymentDeclined). This is the single most valuable design move.
Build a small hierarchy, not a flat pile
PYTHON
class AppError(Exception):
    http_status = 500
    code = "internal_error"

class NotFoundError(AppError):
    http_status = 404
    code = "not_found"

class ValidationError(AppError):
    http_status = 400
    code = "validation_failed"

class InsufficientBalance(AppError):
    http_status = 402
    code = "insufficient_balance"

Now one generic boundary handler can turn any AppError into a proper JSON response, while unexpected Exceptions fall through to a 500 — you get consistent error contracts for free.

Why it's powerful: it drives control flow and the API contract

A custom hierarchy lets each layer of the service speak in domain terms and lets the top layer translate uniformly:

PYTHON
# domain layer raises meaning
if account.balance < amount:
    raise InsufficientBalance(account_id=account.id)

# HTTP boundary translates meaning -> response, in ONE place
@app.errorhandler(AppError)
def handle(err: AppError):
    return jsonify(error=err.code, message=str(err)), err.http_status

It also enables retry classification: mark exceptions as retryable or not, so a worker knows a GatewayTimeout is worth retrying but a PaymentDeclined is terminal.

When is it worth doing?

Worth it when:

The error is a domain concept your callers must branch on — InsufficientBalance, DuplicateTransaction, KYCPending. Callers need except InsufficientBalance to do something specific; a generic ValueError forces brittle message-matching.
You're building a library / shared module — a custom base (class MyLibError(Exception)) lets consumers except MyLibError to catch everything from your library without knowing internals. This is a near-universal best practice for reusable code.
You need to attach structured data — order id, reason code, retryable flag, HTTP status — that a handler acts on programmatically.
You want a clean, uniform API error contract — one base class → one boundary handler → consistent status codes and error bodies.

Not worth it (reuse a built-in) when:

A built-in already means exactly the right thing and no caller needs to distinguish it — bad int → ValueError, missing key → KeyError, bad file → FileNotFoundError. Inventing MyValueError adds nothing.
The error is internal and never crosses a boundary — if nobody catches it specifically and it just becomes a 500, a custom type is ceremony.
You'd create one class per error site — that's over-engineering; group errors into a small hierarchy instead of a class explosion.

Rule of thumb: Create a custom exception when a caller (or the boundary) needs to catch it specifically or read structured data off it; otherwise reuse a built-in. Distinct type is the feature — if no code will ever except ThatType or read its attributes, you don't need the type.

Summary — what a strong answer emphasizes

Subclass Exception, call super().__init__, attach structured attributes, and organize errors into a small hierarchy with a shared base class. That base class is what pays off: broad except for library boundaries, narrow except for specific recovery, and a single place to map errors → HTTP status/JSON. Introduce a custom exception when the error is a domain concept callers branch on or when you're authoring a shared module; reuse ValueError/KeyError/etc. when a built-in already conveys the meaning and nothing catches it specifically.

#### Common mistakes
subclassing BaseException; forgetting super().__init__ (breaks str()/logging); a flat set of unrelated classes with no common base; a class-per-error explosion; stuffing data into the message string instead of attributes.

#### Follow-up
probes: "How do you preserve the original cause?" → raise NewError(...) from original (sets __cause__, keeps the chained traceback). "How do you make some errors retryable?" → a boolean class attribute or a RetryableError marker base class.

**Comparison:**

```
Situation	Custom exception?	Why
Caller must branch on it (InsufficientBalance, KYCPending)	Yes	Enables except ThatType; no message-string matching
Authoring a shared library/module	Yes	Consumers except MyLibError to catch all of it
Need structured data (order id, retryable, http_status)	Yes	Handler acts on attributes programmatically
Bad int / missing key / missing file, nobody catches it specially	No	Built-in ValueError/KeyError/FileNotFoundError already fits
Purely internal error that just becomes a 500	No	Custom type is ceremony with no consumer
One class per error site	No	Over-engineering — group into a hierarchy
```

**Code:**

```python
# 1) Base + specific subclasses carrying structured data and HTTP mapping
class AppError(Exception):
    http_status = 500
    code = "internal_error"
    retryable = False

class ValidationError(AppError):
    http_status, code = 400, "validation_failed"

class InsufficientBalance(AppError):
    http_status, code = 402, "insufficient_balance"
    def __init__(self, account_id: str):
        self.account_id = account_id
        super().__init__(f"insufficient balance on account {account_id}")

class GatewayTimeout(AppError):
    http_status, code, retryable = 504, "gateway_timeout", True

# 2) Domain layer raises meaning
def debit(account, amount):
    if account.balance < amount:
        raise InsufficientBalance(account.id)

# 3) ONE boundary handler maps any AppError -> response
@app.errorhandler(AppError)
def handle_app_error(err):
    return jsonify(error=err.code, message=str(err)), err.http_status

# 4) Preserve the original cause when wrapping
try:
    gateway.charge(order)
except requests.Timeout as e:
    raise GatewayTimeout(f"charge timed out for {order.id}") from e
```

**Example input / output:**

```

Request/response driven by the custom hierarchy:

CODE
Copy
POST /v1/payments  { "account": "acc_9271", "amount": 5000 }
  -> debit() raises InsufficientBalance(account_id="acc_9271")
  -> boundary handler reads err.http_status=402, err.code="insufficient_balance"
<- HTTP 402
   { "error": "insufficient_balance",
     "message": "insufficient balance on account acc_9271" }

# Retry classification in a worker:
try: charge()
except AppError as e:
    if e.retryable: enqueue_retry()   # GatewayTimeout -> retried
    else: mark_failed()               # PaymentDeclined -> terminal
```

**Data:**

```json
// Uniform error body produced by the single boundary handler
{
  "error": "insufficient_balance",
  "message": "insufficient balance on account acc_9271",
  "request_id": "req_5f3a1c"
}
```

**Flow:**

1. No
2. No
3. Yes
4. Yes
5. Exception occurs
6. Does a caller need tocatch THIS specificallyor read structured data?
7. Is it your sharedlibrary boundary?
8. Reuse a built-inValueError / KeyError / ...
9. Custom base classso consumers can except it
10. Subclass Exception,super().__init__,add attributes,put in small hierarchy

