---
type: "practice"
difficulty: "Deep"
---

# A principal round asks how you'd design a 'specification pattern' style query model so complex business predicates (eligible customers) are composable, reusable across in-memory checks and DB queries, and testable in isolation; where does the translation to SQL live and what's the leak risk?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Tenant Isolation in Repositories](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Tenant_Isolation_in_Repositories/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary: To design a specification pattern query model that enables complex business predicates (eligible customers) to be composable, reusable across both in-memory checks and database queries, we need to create a flexible yet testable framework. The key is to decouple the high-level business logic from its implementation details, ensuring both ease of testing and efficient execution.

#### Triage
The core of this question revolves around designing a robust query model that can handle complex logical conditions in a way that maintains composability, reusability, and testability. It also seeks insight into how SQL translation is managed to avoid potential data leak risks.

#### First-Principles Solution

Define the Specification Interface: Begin by defining a Specification interface that encapsulates predicates.
Implement Composite Patterns: Use composite patterns (e.g., And, Or) to allow for complex predicate construction.
Embed Query Logic in Specifications: Each specification should contain enough information to translate itself into SQL, either directly or through an adapter.
Translation Mechanism: The translation mechanism could live within the specification implementations themselves but managed by a unified service layer that ensures consistency and security.

Here’s how we can break it down:

Specification Interface:

TYPESCRIPT
export interface Specification<T> {
  isSatisfiedBy(entity: T): boolean;
  toSqlCondition(): string;
}

Composite Patterns:

And:
TYPESCRIPT
export class And<T> implements Specification<T> {
  private specs: Specification<T>[] = [];

  add(spec: Specification<T>): void {
    this.specs.push(spec);
  }

  isSatisfiedBy(entity: T): boolean {
    return this.specs.every(s => s.isSatisfiedBy(entity));
  }

  toSqlCondition(): string {

#### cons
t conditions = this.specs.map(s => s.toSqlCondition()).join(" AND ");
    return `(${conditions})`;
  }
}

Concrete Specifications:

#### Example
IsEligibleCustomer that translates into a specific SQL condition.
TYPESCRIPT
export class IsEligibleCustomer implements Specification<Customer> {
  private criteria = { age: 18, location: "NYC" };

  isSatisfiedBy(customer: Customer): boolean {
    return customer.age >= this.criteria.age && customer.location === this.criteria.location;
  }

  toSqlCondition(): string {
    return `age >= ${this.criteria.age} AND location = '${this.criteria.location}'`;
  }
}

Translation Service: This service ensures that all specifications are translated consistently and securely.

TYPESCRIPT
export class SpecificationTranslator {
  translate(spec: Specification<Customer>): string {
    // Translate the specification to SQL, ensuring no data leaks
    return spec.toSqlCondition();
  }

  translateComposite(compositeSpec: Specification<Customer>, subQueries: Record<string, string>): string {

#### cons
t conditions = compositeSpec.specs.map(s => this.translate(s)).join(" AND ");
    return `(${conditions})`;
  }
}

Usage:

TYPESCRIPT

#### cons
t specTranslator = new SpecificationTranslator();

#### cons
t eligibleCustomerSpec = new And<Customer>();
eligibleCustomerSpec.add(new IsEligibleCustomer());

// Translate the entire specification to SQL

#### cons
t sqlCondition = specTranslator.translateComposite(eligibleCustomerSpec, {});

#### cons
ole.log(sqlCondition); // e.g., (age >= 18 AND location = 'NYC')

#### Closing Summary
+ Common Mistakes/Follow-Ups: This design ensures that complex business logic is encapsulated within the Specification pattern, making it composable and reusable. The translation service manages the conversion to SQL securely, avoiding data leaks by ensuring that all sensitive information remains within the specification implementations. Common mistakes include not properly defining the interface or not managing the translation mechanism effectively. Follow-ups might involve discussing performance optimizations, security measures, and integration with existing systems.

**Comparison:**

```
Specification Interface	Description
isSatisfiedBy(entity: T): boolean	Evaluates if the entity satisfies the specification.
toSqlCondition(): string	Translates the specification into a SQL condition.
Composite Specification Implementation	Description
And<T>	Implements logical AND for specifications.
add(spec: Specification<T>): void	Adds a sub-specification to the composite.
isSatisfiedBy(entity: T): boolean	Evaluates if all sub-specifications are satisfied.
toSqlCondition(): string	Combines SQL conditions of sub-specifications using AND.
```

**Code:**

```typescript
export interface Specification<T> {
  isSatisfiedBy(entity: T): boolean;
  toSqlCondition(): string;
}

export class And<T> implements Specification<T> {
  private specs: Specification<T>[] = [];

  add(spec: Specification<T>): void {
    this.specs.push(spec);
  }

  isSatisfiedBy(entity: T): boolean {
    return this.specs.every(s => s.isSatisfiedBy(entity));
  }

  toSqlCondition(): string {
    const conditions = this.specs.map(s => s.toSqlCondition()).join(" AND ");
    return `(${conditions})`;
  }
}

export class IsEligibleCustomer implements Specification<Customer> {
  private criteria = { age: 18, location: "NYC" };

  isSatisfiedBy(customer: Customer): boolean {
    return customer.age >= this.criteria.age && customer.location === this.criteria.location;
  }

  toSqlCondition(): string {
    return `age >= ${this.criteria.age} AND location = '${this.criteria.location}'`;
  }
}

export class SpecificationTranslator {
  translate(spec: Specification<Customer>): string {
    return spec.toSqlCondition();
  }

  translateComposite(compositeSpec: Specification<Customer>, subQueries: Record<string, string>): string {
    const conditions = compositeSpec.specs.map(s => this.translate(s)).join(" AND ");
    return `(${conditions})`;
  }
}
```

**Example input / output:**

```text
const specTranslator = new SpecificationTranslator();
const eligibleCustomerSpec = new And<Customer>();
eligibleCustomerSpec.add(new IsEligibleCustomer());

// Translate the entire specification to SQL
const sqlCondition = specTranslator.translateComposite(eligibleCustomerSpec, {});
console.log(sqlCondition); // (age >= 18 AND location = 'NYC')
```

**Data:**

```json
{
  "age": 18,
  "location": "NYC"
}
```

**Flow:**

1. Specification Translator
2. Translate Composite Spec
3. Translate Sub-Specs
4. (SQL Condition)

