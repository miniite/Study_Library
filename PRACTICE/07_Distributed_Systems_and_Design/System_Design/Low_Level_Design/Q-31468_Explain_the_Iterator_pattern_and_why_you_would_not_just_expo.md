---
type: "practice"
difficulty: "Deep"
---

# Explain the Iterator pattern and why you would not just expose the internal list.

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Over-engineering vs Simplicity](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Over-engineering_vs_Simplicity/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is being asked: Two linked things — (1) explain the Iterator pattern, and (2) argue why you'd expose an iterator instead of just returning/exposing the underlying List. The second half is the real test: it probes your grasp of encapsulation, abstraction over data structures, and API design discipline. Asked by strong product companies (Amazon, Google, Microsoft, Atlassian, Flipkart, Razorpay, Postman, Zoho) at a "deep understanding" bar.

What the interviewer is probing for: Do you understand that an iterator decouples traversal from the collection's internal representation? Can you name the concrete harms of leaking the internal list (broken invariants, representation lock-in, no lazy/infinite sequences, concurrency issues)? Do you know the Java Iterator/Iterable contract, hasNext/next/remove, fail-fast behavior, and how it powers the for-each loop?

#### Assumptions
Java-centric (typical). I'll show a custom collection whose internal storage should stay hidden, and contrast "return the list" vs "return an iterator."

Plan: define the pattern → contract → concrete example → the case against exposing the internal list (5 concrete reasons) → nuance (Collections.unmodifiableList, Stream) → trade-offs → summary.

What the Iterator pattern is

Iterator is a behavioral pattern that provides a way to access the elements of an aggregate (collection) sequentially without exposing its underlying representation. It separates the traversal logic from the collection. The collection ("aggregate") knows how to create an iterator; the iterator knows how to walk the elements one at a time and track position.

Two roles:

Iterable / Aggregate: exposes a factory method iterator() that returns a fresh cursor.
Iterator: holds traversal state and exposes hasNext() (is there another element?) and next() (return it and advance). Java also defines optional remove().

In Java these are the java.lang.Iterable<T> and java.util.Iterator<T> interfaces. Any class implementing Iterable can be used directly in the enhanced for-each loop — the compiler desugars for (T x : coll) into Iterator<T> it = coll.iterator(); while (it.hasNext()) { T x = it.next(); ... }.

The contract
CODE
interface Iterable<T> { Iterator<T> iterator(); }
interface Iterator<T> {
    boolean hasNext();
    T next();                 // throws NoSuchElementException if none
    default void remove();    // optional; removes last returned element
}

Well-behaved iterators are typically fail-fast: if the backing collection is structurally modified during iteration (except via the iterator's own remove()), the next next()/hasNext() throws ConcurrentModificationException, catching bugs early rather than corrupting state silently.

Concrete example — why the internal list must stay hidden

Suppose I build a Playlist that internally stores songs in an ArrayList, but also maintains an invariant: no duplicate songs, and it caches the total duration. Two ways to let callers loop over songs:

Leak it: public List<Song> getSongs() { return songs; }
Encapsulate: implements Iterable<Song> and return an iterator (or an unmodifiable view).

Option 1 is convenient but dangerous. Here's the case against it.

Why NOT just expose the internal list

1. It breaks encapsulation and invariants. Returning the actual List reference hands the caller a live handle to my internals. They can call playlist.getSongs().add(dup) or .clear() — bypassing my no-duplicates check and leaving my cached totalDuration wrong. My class can no longer guarantee its own invariants because anyone can mutate state behind its back. The whole point of a class is to protect its data; leaking the collection defeats it.

2. It locks in the internal representation. If callers depend on getting a List, I can never change the internal storage — to a LinkedHashSet (to enforce uniqueness), a tree, a ring buffer, or a lazily loaded/paged source — without breaking every caller. An iterator (or Iterable) is an abstraction: callers only know "I can walk elements one by one," so I'm free to swap the underlying structure. This is programming to an interface, not an implementation.

3. It prevents lazy / computed / infinite sequences. A List must be fully materialized in memory. An iterator can compute elements on demand — stream results from a DB cursor, a network page, a file line-by-line, or an infinite generator (Fibonacci, natural numbers). You cannot represent "the next prime forever" as a List, but you can as an Iterator. This is huge for memory and for pipelines.

4. It enables multiple simultaneous, independent traversals. Each iterator() call returns a separate cursor with its own position, so two loops (even nested) don't interfere. Exposing a single mutable index or the raw list makes concurrent traversal error-prone.

5. It centralizes traversal concerns and safety. The iterator can enforce fail-fast concurrency checks, read-only access, or filtering/transformation as it goes — without the caller knowing. If instead you must hand out a collection, return a read-only view (Collections.unmodifiableList(songs)) so structural mutation throws, keeping your invariants intact.

Nuance — modern Java

Since Java 8, Stream and Iterator/Iterable overlap. For most application code you'd expose stream() or return an unmodifiable/immutable view rather than hand-writing an Iterator. But the reasoning is identical: give callers a read-only, representation-independent way to traverse, never a live handle to internal mutable state. The Iterator pattern is also still the right tool when you need custom/lazy/infinite traversal, or when implementing a data structure of your own (tree, graph, custom cache).

#### Trade-offs
Cost of iterator: slightly more code (one small class or a lambda-ish anonymous class), and a level of indirection vs. a raw for-loop over a list.
Benefit: encapsulation, freedom to change internals, lazy/infinite sequences, safe read-only exposure, uniform traversal (for-each, streams).
When exposing a collection is fine: internal DTOs/records with no invariants, or when you deliberately return an immutable/unmodifiable copy. Even then, returning List.copyOf(...) or an unmodifiable view is safer than the live field.

#### Common mistakes
/ follow-up probes
Returning the raw field and calling it "encapsulated" — it isn't; the reference is shared.
Forgetting fail-fast semantics; probe: "How does ConcurrentModificationException arise?" → structural modification outside the iterator during iteration (a mod-count mismatch).
Probe: "Difference between Iterator and Iterable?" → Iterable is the collection (has iterator()); Iterator is the one-shot cursor. Probe: "Internal vs external iterator?" → forEach(Consumer) (collection drives) vs hasNext/next (client drives).
Summary — what a strong answer emphasizes

The Iterator pattern provides sequential access to a collection's elements without exposing its internal structure, via a cursor with hasNext()/next(). You don't just expose the internal list because that leaks a live mutable handle — it breaks encapsulation/invariants, locks in the representation, forbids lazy/infinite sequences, and prevents safe/independent traversal. Expose an Iterable/iterator (or at least an unmodifiable view) so callers depend on the abstraction of traversal, not on your storage. Mentioning fail-fast, for-each desugaring, and Streams as the modern idiom signals senior depth.

**Comparison:**

```
Concern	Return the internal List	Return an Iterator / Iterable
Encapsulation	Broken — live mutable handle leaks	Preserved — only traversal exposed
Invariants (uniqueness, caches)	Caller can silently violate	Class stays in control
Change internal storage later	Breaks all callers	Free to swap (List→Set→tree→DB cursor)
Lazy / infinite sequences	Impossible (must materialize)	Natural (compute on demand)
Multiple independent walks	Error-prone	Each iterator() = fresh cursor
Read-only guarantee	None (unless you copy)	Iterator can refuse remove()
Interface	Purpose	Key methods
Iterable<T>	The collection (aggregate)	iterator()
Iterator<T>	One-shot traversal cursor	hasNext(), next(), remove() (optional)
```

**Code:**

```java
import java.util.*;

// Custom collection with an INVARIANT (no duplicates + cached size)
// that would be violated if we leaked the internal list.
class Playlist implements Iterable<String> {
    private final List<String> songs = new ArrayList<>();  // internal representation

    public void add(String song) {
        if (!songs.contains(song)) songs.add(song);        // invariant: unique
    }
    public int size() { return songs.size(); }

    // GOOD: expose traversal, not the list. Returns a read-only cursor.
    @Override
    public Iterator<String> iterator() {
        return new Iterator<>() {
            private int i = 0;
            public boolean hasNext() { return i < songs.size(); }
            public String next() {
                if (!hasNext()) throw new NoSuchElementException();
                return songs.get(i++);
            }
            // remove() intentionally NOT supported -> read-only traversal
        };
    }

    // If a collection MUST be returned, return an unmodifiable VIEW, never the field
    public List<String> asReadOnly() { return Collections.unmodifiableList(songs); }
}

// Lazy / infinite sequence — impossible with a List, natural with an Iterator
class Naturals implements Iterable<Long> {
    public Iterator<Long> iterator() {
        return new Iterator<>() {
            private long n = 1;
            public boolean hasNext() { return true; }   // infinite
            public Long next() { return n++; }
        };
    }
}

public class Demo {
    public static void main(String[] args) {
        Playlist p = new Playlist();
        p.add("A"); p.add("B"); p.add("A");   // dup ignored -> invariant holds

        for (String s : p) System.out.println(s);        // for-each uses iterator()

        // Leaking the list would allow: p.asReadOnly().add("X") -> throws (protected)
        try { p.asReadOnly().add("X"); }
        catch (UnsupportedOperationException e) { System.out.println("blocked mutation"); }

        // Lazy infinite traversal, take 3
        var it = new Naturals().iterator();
        System.out.println(it.next() + "," + it.next() + "," + it.next());
    }
}
```

**Example input / output:**

```text
// for (String s : p)  -> walks via iterator(); dup "A" was rejected on add
A
B

// p.asReadOnly().add("X") on an unmodifiable view:
blocked mutation

// Lazy infinite Naturals iterator, first three:
1,2,3
```

**Flow:**

1. Iterator (cursor)
2. Playlist (Iterable)
3. Client (for-each)
4. Iterator (cursor)
5. Playlist (Iterable)
6. Client (for-each)
7. loop
8. [until hasNext() == false]
9. iterator()
10. new Iterator (own position)
11. hasNext()
12. true
13. next()
14. element (advance)

