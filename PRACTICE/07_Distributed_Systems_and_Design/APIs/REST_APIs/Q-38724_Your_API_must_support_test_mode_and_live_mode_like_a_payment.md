---
type: "practice"
difficulty: "Deep"
---

# Your API must support 'test mode' and 'live mode' like a payments provider. How do you isolate data, keys, and side effects between modes while sharing one codebase and endpoint surface?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Authorization Design](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Authorization_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview
To address the requirement of supporting both "test" and "live" modes in a REST API without duplicating code or endpoints, we need to ensure data isolation, key management, and control over side effects. The core idea is to design the system so that it can dynamically switch between environments based on configuration settings.

#### Triage
The question is probing how you manage different operational modes (test vs live) in a single codebase while ensuring security and data integrity. This involves understanding environment-specific configurations, key management, and side effects like logging or rate limiting.

Environment Configuration:

Use environment variables to distinguish between test and live environments.

#### Example
.env files:
PLAINTEXT
# .env.test
API_KEY=test_api_key
ENV=test

# .env.live
API_KEY=live_api_key
ENV=live

Data Isolation:

Database Schema Design: Use a database schema that supports both environments.
SQL
-- Sample table for transactions
CREATE TABLE IF NOT EXISTS transactions (
    id SERIAL PRIMARY KEY,
    amount DECIMAL(10, 2),
    status VARCHAR(50),
    environment VARCHAR(50) CHECK (environment IN ('test', 'live'))
);

Key Management:

Use environment-specific API keys or secrets.

#### Example
code snippet:
PYTHON
import os

def get_api_key():
    if os.getenv('ENV') == 'test':
        return os.environ.get('TEST_API_KEY')
    else:
        return os.environ.get('LIVE_API_KEY')

print(get_api_key())

Side Effect Management:

Configure logging levels based on the environment.

#### Example
configuration for a logging.config file:
INI
[loggers]
keys=root

[handlers]
keys=consoleHandler

[formatters]
keys=simpleFormatter

[logger_root]
level=DEBUG if ENV == 'test' else ERROR
handlers=consoleHandler

[handler_consoleHandler]
class=StreamHandler
level=NOTSET
formatter=simpleFormatter
args=(sys.stdout,)

[formatter_simpleFormatter]
format=%(asctime)s - %(name)s - %(levelname)s - %(message)s
datefmt=

Rate Limiting and Security:

Implement rate limiting with environment-specific thresholds.

#### Example
using Flask:
PYTHON
from flask_limiter import Limiter

limiter = Limiter(key_func=get_remote_address, default_limits=["100 per day", "20 per hour"] if os.getenv('ENV') == 'test' else ["500 per day"])

@app.route('/api/endpoint')
@limiter.limit("20 per minute")  # Adjust based on environment
def api_endpoint():
    return jsonify({"message": "Hello, World!"})

#### Closing Summary
In summary, the key to managing different modes is through careful configuration and dynamic runtime decisions. By using environment variables for keys, schema design for data isolation, logging configurations for side effects, and rate limiting, we can ensure that a single codebase supports both test and live environments securely.

#### Common Mistakes/Follow-Ups

Failing to properly isolate sensitive information like API keys.
Not configuring different logging levels or side effects based on the environment.
Overlooking the need for clear documentation of how the system handles different modes.

**Comparison:**

```
Environment	Configurations	Example
Test	API Key = test_api_key	.env.test
Live	API Key = live_api_key	.env.live
```

**Code:**

```

import os

def get_api_key(): if os.getenv('ENV') == 'test': return os.environ.get('TEST_API_KEY') else: return os.environ.get('LIVE_API_KEY')

print(get_api_key())

CODE
Copy

```ini
[loggers]
keys=root

[handlers]
keys=consoleHandler

[formatters]
keys=simpleFormatter

[logger_root]
level=DEBUG if ENV == 'test' else ERROR
handlers=consoleHandler

[handler_consoleHandler]
class=StreamHandler
level=NOTSET
formatter=simpleFormatter
args=(sys.stdout,)
```

**Example input / output:**

```text
from flask_limiter import Limiter

limiter = Limiter(key_func=get_remote_address, default_limits=["100 per day", "20 per hour"] if os.getenv('ENV') == 'test' else ["500 per day"])

@app.route('/api/endpoint')
@limiter.limit("20 per minute")  # Adjust based on environment
def api_endpoint():
    return jsonify({"message": "Hello, World!"})
```

**Data:**

```sql
-- Sample table for transactions
CREATE TABLE IF NOT EXISTS transactions (
    id SERIAL PRIMARY KEY,
    amount DECIMAL(10, 2),
    status VARCHAR(50),
    environment VARCHAR(50) CHECK (environment IN ('test', 'live'))
);
```

**Flow:**

1. Environment Configuration
2. Get API Key
3. Logging Configuration
4. Rate Limiting Configuration

