---
type: "practice"
difficulty: "Deep"
---

# How do you design gateway routing for a monorepo of 40 microservices where routes are declared close to each service (service-owned config) but assembled into one gateway; how do you prevent one team's bad config from breaking others?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Control Plane and Data Plane Design](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Routing_and_BFF_Patterns/Control_Plane_and_Data_Plane_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Designing an effective API Gateway for a monorepo of 40 microservices where routes are declared close to each service but assembled into a single gateway is crucial for maintaining system integrity and ensuring that one team's bad configuration does not impact others. The key idea here is to leverage modular, decoupled configurations with robust validation and fallback mechanisms.

The core question revolves around how we can ensure:

Modular Configuration: Each microservice owns its routing configuration.
Centralized Assembly: All routes are assembled into a single gateway.
Isolation of Configurations: Bad configurations from one service do not disrupt others.

#### Triaging the Question

The interviewer is probing your understanding of:

Modular Design Principles: Ensuring that each microservice owns its configuration without interference.
Centralization and Assembly: How to logically combine decentralized configurations into a unified gateway.
Error Handling and Fallbacks: Implementing strategies to mitigate the impact of misconfigured services.

#### Assumptions
Each service has a local route configuration file.
Services are written in a modern language with access to external configuration systems (e.g., environment variables, config files).
The API Gateway can dynamically load and validate these configurations before applying them.
Step-by-Step Solution
1. Service-Oriented Configuration

Each microservice will have its own route definition file, typically in YAML or JSON format:

YAML
# Example service route configuration
routes:
  - path: /users
    method: GET
    handler: UserController.listUsers

This ensures that each team is responsible for their service's routing logic.

2. Dynamic Configuration Loading

The API Gateway will dynamically load these configurations from the respective services and assemble them into a unified configuration object:

PYTHON
# Pseudocode for dynamic config loading
def load_service_configs():
    configs = []
    for service in services:  # Assume 'services' is a list of microservice instances
        config = service.get_route_config()
        if validate(config):
            configs.append(config)
    return assemble_config(configs)

def assemble_config(configs):
    gateway_config = {}
    for config in configs:
        merge_routes(gateway_config, config)
    return gateway_config

def validate(config):
    # Validate the configuration using predefined rules
    pass

def merge_routes(destination, source):
    for route in source['routes']:
        if route not in destination['routes']:
            destination['routes'].append(route)
3. Validation and Error Handling

Before applying any configurations, the gateway should validate them:

PYTHON
# Pseudocode for validation
def validate(config):
    if 'routes' not in config or not config['routes']:
        return False
    for route in config['routes']:
        if 'path' not in route or 'method' not in route or 'handler' not in route:
            return False
    return True

If a configuration fails validation, it should be ignored and logged as an error.

4. Fallback Mechanisms

Implement fallbacks to prevent failures from one service from breaking the gateway:

PYTHON
# Pseudocode for fallback handling
def handle_exception(e):
    if isinstance(e, ConfigurationValidationError):
        log.error(f"Failed to load config: {e}")
        return default_gateway_config

default_gateway_config = {
    "routes": [
        {"path": "/health", "method": "GET", "handler": "HealthController.healthCheck"}
    ]
}

This ensures that even if a service's configuration is bad, the gateway can still function with a predefined set of routes.

#### Closing Summary
and Common Mistakes

In summary, designing an API Gateway for a monorepo of 40 microservices involves modular configurations owned by each service, centralized assembly in the gateway, robust validation, and fallback mechanisms to ensure system resilience. Common mistakes include:

Ignoring Validation: Failing to validate loaded configurations can lead to runtime errors.
Insufficient Error Handling: Not implementing proper error handling can result in cascading failures.
Overcomplicating the Assembly Process: Over-engineering the configuration assembly process can introduce unnecessary complexity.

#### Follow-Up
Questions
How would you handle cases where a service's configuration is missing some required fields?
What strategies do you have for automatically detecting and fixing misconfigurations in real-time?
Can you discuss how this solution scales with more microservices?

**Comparison:**

```
Service	Routes
UserService	/users (GET)
HealthService	/health (GET)
```

**Code:**

```python
# Example pseudocode for dynamic config loading
def load_service_configs():
    configs = []
    for service in services:
        config = service.get_route_config()
        if validate(config):
            configs.append(config)
    return assemble_config(configs)

def assemble_config(configs):
    gateway_config = {}
    for config in configs:
        merge_routes(gateway_config, config)
    return gateway_config

def validate(config):
    if 'routes' not in config or not config['routes']:
        return False
    for route in config['routes']:
        if 'path' not in route or 'method' not in route or 'handler' not in route:
            return False
    return True

def merge_routes(destination, source):
    for route in source['routes']:
        if route not in destination['routes']:
            destination['routes'].append(route)
```

**Example input / output:**

```text
# Example input-output for dynamic configuration loading and assembly
services = [
    {
        'name': 'UserService',
        'config': {
            'routes': [
                {'path': '/users', 'method': 'GET', 'handler': 'UserController.listUsers'}
            ]
        }
    },
    {
        'name': 'HealthService',
        'config': {
            'routes': [
                {'path': '/health', 'method': 'GET', 'handler': 'HealthController.healthCheck'}
            ]
        }
    }
]

gateway_config = load_service_configs(services)
print(gateway_config)
```

**Data:**

```yaml
# Example service route configuration
services:
  - name: UserService
    config:
      routes:
        - path: /users
          method: GET
          handler: UserController.listUsers
  - name: HealthService
    config:
      routes:
        - path: /health
          method: GET
          handler: HealthController.healthCheck
```

**Flow:**

1. Load Service Configs
2. Validate Configurations
3. Valid
4. Asemble Gateway Config
5. Invalid
6. Log Error, Use Default Config

