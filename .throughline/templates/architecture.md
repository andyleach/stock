# <Architecture Area>

## Purpose & scope
<What this piece is and the boundary of its responsibility. Scope each architecture doc to **one distinct architectural concept**. Do not lump several unrelated concepts (e.g. authentication, caching, and background jobs) into a single item — each deserves its own doc. Equally, don't over-decompose: avoid splitting one coherent concept into many trivially small fragments. The right grain is a concept a developer would reason about as a single unit.>

## Where it's used
<Which features/areas depend on it — the cross-feature picture.>

## Key abstractions / contracts

The main interfaces, base classes, or contracts and what they promise. One row each; link the name to where it's defined.

| Abstraction | Type | What it promises |
|---|---|---|
| [`<Name>`](<path>) | interface / base class / trait / protocol / contract | <the guarantee it makes and what implementers must uphold> |

## Invariants & guarantees
<What must always hold for this area to be correct — ordering, idempotency, consistency, uniqueness, thread-safety, and the like. State each as a rule a developer must not break. Omit if there are none worth calling out.>

## Design decisions & trade-offs
<Why it's built this way: the notable decisions, the alternatives that were rejected, and what each choice costs or buys. Often the highest-value part of the doc — capture the reasoning a future maintainer would otherwise have to reverse-engineer.>

## How to extend
<How a developer plugs into or extends this.>

## Key files

Link each file so a reader can jump straight to it.

| File | Responsibility |
|---|---|
| [`<path>`](<path>) | <what it defines> |

## Diagram (optional)
<A Mermaid or ASCII diagram if it aids understanding.>
