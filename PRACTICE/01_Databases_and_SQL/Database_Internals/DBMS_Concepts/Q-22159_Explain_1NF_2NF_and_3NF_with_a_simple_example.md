---
type: "practice"
difficulty: "Intermediate"
---

# Explain 1NF, 2NF, and 3NF with a simple example.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1 · [↑ Normal Forms](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Normal_Forms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The interviewer is asking me to explain the first three normal forms — 1NF, 2NF, and 3NF — and to demonstrate each with a running example. For a Fullstack/Backend SDE at the "intermediate experience" level, the bar is not just reciting definitions. They want to see that I understand why each form exists (which anomaly it removes), the vocabulary of functional dependencies and keys that the theory is built on, and that I can carry one concrete schema from an un-normalized mess to a clean 3NF design step by step. A strong answer also mentions when you'd stop (or step back from) 3NF in practice.

Clarifying assumptions: Normalization is about eliminating redundancy and the update/insert/delete anomalies it causes. I'll assume a classic order/customer example. I'll define terms as I go (superkey, candidate key, prime attribute, partial vs. transitive dependency) so the reasoning is self-contained.

Plan: (1) Set up the vocabulary. (2) Start with an un-normalized table and normalize it into 1NF → 2NF → 3NF, showing at each step which specific anomaly is removed. (3) Summarize with a one-line rule for each form and note the practical caveats.

Vocabulary first (so the rules make sense)
Functional dependency (FD) X → Y: the value of X determines the value of Y. E.g., order_id → order_date (one order has exactly one date).
Candidate key: a minimal set of columns that uniquely identifies a row. A primary key is the chosen candidate key.
Prime attribute: an attribute that is part of some candidate key. Non-prime: everything else.
Partial dependency: a non-prime attribute depends on only part of a composite key.
Transitive dependency: a non-prime attribute depends on another non-prime attribute (which in turn depends on the key), i.e. key → A → B.
The starting point: an un-normalized table

Imagine one wide Orders table someone built without thinking:

order_id	customer_id	customer_name	items	item_prices
1	C10	Asha	Pen, Notebook	10, 40
2	C11	Ravi	Pen	10

Two problems jump out: items and item_prices are multi-valued (a list crammed into one cell), and there's redundancy waiting to happen.

1NF — First Normal Form: atomic values, no repeating groups

Rule: every cell holds a single, atomic value; there are no repeating groups or arrays inside a column, and each row is unique.

The fix is to give each item its own row:

order_id	customer_id	customer_name	item	item_price
1	C10	Asha	Pen	10
1	C10	Asha	Notebook	40
2	C11	Ravi	Pen	10

Now the table is a proper relation. The candidate key here is composite: (order_id, item) — you need both to identify a row. Why 1NF matters: without atomic values you can't index, filter (WHERE item = 'Pen'), or join reliably. But 1NF alone still has redundancy — customer_name "Asha" repeats for every item in order 1.

2NF — Second Normal Form: no partial dependencies

Rule: be in 1NF and every non-prime attribute depends on the whole candidate key, not just part of it. (2NF only bites when the key is composite; if the key is a single column, a 1NF table is automatically in 2NF.)

Look at the FDs against key (order_id, item):

order_id → customer_id, customer_name — these depend on order_id alone, i.e. part of the key. That's a partial dependency.
item → item_price — depends on item alone (assuming a fixed catalog price). Also partial.
(order_id, item) → quantity (if we had it) would be a full dependency.

To reach 2NF, split out each partial dependency into its own table:

Orders (key order_id): order_id, customer_id, customer_name Products (key item): item, item_price OrderLines (key (order_id, item)): order_id, item (plus quantity, etc.)

Anomaly removed: in 1NF, if order 1 had 5 items, "Asha" was stored 5 times (update anomaly — renaming her means 5 writes; risk of inconsistency). After 2NF, customer_name lives once per order.

3NF — Third Normal Form: no transitive dependencies

Rule: be in 2NF and no non-prime attribute depends on another non-prime attribute. Every non-key attribute must depend on the key, the whole key, and nothing but the key.

Look at the Orders table: order_id → customer_id → customer_name. Here customer_name depends on customer_id, which is itself non-prime — a transitive dependency. So customer_name is really an attribute of the customer, not the order. If a customer places 100 orders, their name is stored 100 times (still redundant, still update-anomaly-prone).

Fix by extracting the customer:

Customers (key customer_id): customer_id, customer_name Orders (key order_id): order_id, customer_id (FK → Customers)

Now the schema is in 3NF:

Customers: customer_id (PK), customer_name
Products: item (PK), item_price
Orders: order_id (PK), customer_id (FK)
OrderLines: (order_id, item) (PK), order_id (FK), item (FK), quantity

Anomalies removed: renaming a customer is now a single-row update; you can insert a new customer before they've ordered anything (no insert anomaly); deleting an order won't accidentally erase the last record of a customer or product (no delete anomaly).

Summary — the one-liners a strong answer nails
1NF: atomic columns, no repeating groups. "One value per cell."
2NF: 1NF + no partial dependency on a composite key. "Every non-key column needs the whole key."
3NF: 2NF + no transitive dependency between non-key columns. "Non-key columns depend on nothing but the key."

What a great candidate adds: Normalization removes redundancy and thus the insert/update/delete anomalies, at the cost of more joins. In OLTP systems you normalize to ~3NF (or BCNF, a stricter variant that also handles overlapping candidate keys). In OLAP/reporting or read-heavy hot paths you may deliberately denormalize for read speed and accept controlled redundancy. Common mistakes interviewers watch for: thinking 2NF applies to single-column keys (it doesn't add anything there), confusing partial vs. transitive dependencies, and not being able to name the specific anomaly each form prevents.

**Comparison:**

```
Normal Form	Requires	Eliminates	Rule of thumb
1NF	Atomic values, unique rows, no repeating groups	Multi-valued cells / arrays	One value per cell
2NF	1NF + no partial dependency on a composite key	Redundancy from part-of-key deps	Non-key cols need the whole key
3NF	2NF + no transitive dependency (key→A→B)	Redundancy from non-key→non-key deps	Non-key cols depend on nothing but the key
```

**Code:**

```sql
-- Final 3NF schema for the order example
CREATE TABLE Customers (
    customer_id   VARCHAR(10) PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL
);

CREATE TABLE Products (
    item        VARCHAR(50) PRIMARY KEY,
    item_price  NUMERIC(10,2) NOT NULL
);

CREATE TABLE Orders (
    order_id    INT PRIMARY KEY,
    customer_id VARCHAR(10) NOT NULL REFERENCES Customers(customer_id)
);

CREATE TABLE OrderLines (
    order_id INT         NOT NULL REFERENCES Orders(order_id),
    item     VARCHAR(50) NOT NULL REFERENCES Products(item),
    quantity INT         NOT NULL DEFAULT 1,
    PRIMARY KEY (order_id, item)   -- composite candidate key
);
```

**Example input / output:**

```text
Dry-run of the normalization on the sample data:

UNF (one row): order_id=1, customer=Asha, items="Pen, Notebook", prices="10, 40"

--> 1NF (atomic rows):
(1, C10, Asha, Pen, 10)
(1, C10, Asha, Notebook, 40)   <- "Asha" now duplicated (update anomaly)

--> 2NF (remove item -> price and order_id -> customer partial deps):
Orders:     (1, C10, Asha)
Products:   (Pen, 10), (Notebook, 40)
OrderLines: (1, Pen), (1, Notebook)

--> 3NF (remove customer_id -> customer_name transitive dep):
Customers:  (C10, Asha)
Orders:     (1, C10)
Products:   (Pen, 10), (Notebook, 40)
OrderLines: (1, Pen), (1, Notebook)

Result: renaming Asha = 1 write; new customer insertable with no order; deleting order 1 does not erase customer C10 or product Pen.
```

**Flow:**

1. split to atomic rows
2. remove partial depsitem→price,order→customer
3. remove transitive depcustomer_id→customer_name
4. UNF: lists in cells(Pen, Notebook)
5. 1NF
6. 2NF
7. 3NF

