---
type: "practice"
difficulty: "Deep"
---

# How would you design a dependency-injection container for a Python service so tests can swap real DB/HTTP clients for fakes without monkeypatching everywhere?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Safe User-authored Configuration](<../../../../TOOLS/Python/Python_Language/Safe_User-authored_Configuration/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a dependency injection (DI) container in Python that allows swapping real database and HTTP clients with mocks or fakes during testing, we need to ensure our application has clear interfaces and loose coupling between dependencies and services. This approach not only facilitates unit testing but also makes the system more modular and easier to maintain.

#### Triage

The question is about creating a flexible and testable architecture in Python where you can easily replace real database and HTTP clients with mock implementations during tests without resorting to monkeypatching. The key here is to ensure that services depend on interfaces, not concrete classes, making it straightforward to inject mocks or fakes.

#### First-Principles Solution
Define Interfaces: Create abstract base classes (ABC) for database and HTTP clients.
Register Dependencies in a Container: Use a DI container like injector or dependency_injector.
Inject Dependencies into Services: Ensure services accept interfaces as parameters rather than concrete implementations.
Register Mocks for Tests: Replace real dependencies with mocks in test cases.
Detailed Steps

Define Interfaces

PYTHON
from abc import ABC, abstractmethod

class DatabaseClient(ABC):
    @abstractmethod
    def get_data(self, key: str) -> str:
        pass

class HttpClient(ABC):
    @abstractmethod
    def fetch_url(self, url: str) -> str:
        pass

Register Dependencies in a Container Using dependency_injector, we can register these interfaces and their implementations.

Inject Dependencies into Services

PYTHON
from dependency_injector import containers, providers

class Service:
    def __init__(self, db_client: DatabaseClient, http_client: HttpClient):
        self.db_client = db_client
        self.http_client = http_client

    def process_data(self) -> str:
        # Fetch data from the database and HTTP client
        db_data = self.db_client.get_data("key")
        http_response = self.http_client.fetch_url("http://example.com")
        return f"Processed: {db_data} and {http_response}"

Register Mocks for Tests

PYTHON
from unittest.mock import MagicMock

def test_process_data():
    db_mock = MagicMock(spec=DatabaseClient)
    http_mock = MagicMock(spec=HttpClient)

    # Set up mock responses
    db_mock.get_data.return_value = "Mocked Data"
    http_mock.fetch_url.return_value = "Mocked Response"

    container = containers.DynamicContainer()
    container.db_client = providers.Singleton(db_mock)
    container.http_client = providers.Singleton(http_mock)

    service = container.service()

    result = service.process_data()

    assert result == "Processed: Mocked Data and Mocked Response"

#### Closing Summary

By defining clear interfaces, using a DI container to manage dependencies, and injecting these dependencies into services via constructor parameters, we achieve loose coupling and testability. This approach allows us to easily swap out real implementations for mocks or fakes during tests, making the system more robust and easier to maintain.

#### Common Mistakes/Follow-Ups

Mistake: Using concrete classes directly in service constructors instead of interfaces.

#### Follow-Up
Ensure all services accept interfaces as parameters.

Mistake: Not using a DI container or not setting up mocks properly.

#### Follow-Up
Verify that dependencies are correctly registered and injected.

**Code:**

```python
from abc import ABC, abstractmethod

class DatabaseClient(ABC):
    @abstractmethod
    def get_data(self, key: str) -> str:
        pass

class HttpClient(ABC):
    @abstractmethod
    def fetch_url(self, url: str) -> str:
        pass

from dependency_injector import containers, providers

class Service:
    def __init__(self, db_client: DatabaseClient, http_client: HttpClient):
        self.db_client = db_client
        self.http_client = http_client

    def process_data(self) -> str:
        # Fetch data from the database and HTTP client
        db_data = self.db_client.get_data("key")
        http_response = self.http_client.fetch_url("http://example.com")
        return f"Processed: {db_data} and {http_response}"

def test_process_data():
    db_mock = MagicMock(spec=DatabaseClient)
    http_mock = MagicMock(spec=HttpClient)

    # Set up mock responses
    db_mock.get_data.return_value = "Mocked Data"
    http_mock.fetch_url.return_value = "Mocked Response"

    container = containers.DynamicContainer()
    container.db_client = providers.Singleton(db_mock)
    container.http_client = providers.Singleton(http_mock)

    service = container.service()

    result = service.process_data()

    assert result == "Processed: Mocked Data and Mocked Response"
```

**Example input / output:**

```text
# Input
service = container.service()

result = service.process_data()
print(result)

# Output
"Processed: Mocked Data and Mocked Response"
```

**Data:**

```json
{
  "db_client": {
    "get_data": {"key": "key", "return_value": "Mocked Data"}
  },
  "http_client": {
    "fetch_url": {"url": "http://example.com", "return_value": "Mocked Response"}
  }
}
```

