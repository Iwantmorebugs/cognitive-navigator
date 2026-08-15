# Business ↔ code traceability

An expansion of `§5 CONNECT BUSINESS → CODE` and `§6 USE DDD, BDD, TDD AS BRIDGES`, for when the work touches **code with a real domain layer**. This is not a second chain: it is §5's chain with more detail in each link.

```
§5                        expands here into
──────────────────        ──────────────────────────────────
BUSINESS NEED       →     the need, in one sentence
BUSINESS RULE       →     rule + explicit invariant
BEHAVIOUR/USE CASE  →     use case: actor, input, rules, result
DOMAIN MODEL        →     who owns the rule, and why that one
ARCHITECTURE        →     layers as protections, not as folders
COMPONENTS / CODE   →     each piece with the need that justifies it
TEST                →     the test as proof of the rule
```

Skip this file entirely when there is no domain layer to trace — see [EXAMPLE.md](EXAMPLE.md) for the loop on ordinary code.

**The rule governing this file:** an implementation that is technically correct but disconnected from the business reason it exists does not produce enough understanding. The user must be able to get from *"what does the business need?"* to *"where is that in the code?"* without gaps.

## The whole chain

```
┌────────────────────────────────────────────────┐
│ BUSINESS     "A shipped order cannot be        │
│              cancelled"                        │
├────────────────────────────────────────────────┤
│ RULE         Status != Shipped                 │
├────────────────────────────────────────────────┤
│ USE CASE     Cancel order                      │
├────────────────────────────────────────────────┤
│ DOMAIN       Order.Cancel()                    │
├────────────────────────────────────────────────┤
│ APPLICATION  CancelOrderHandler                │
├────────────────────────────────────────────────┤
│ PERSISTENCE  OrderRepository                   │
├────────────────────────────────────────────────┤
│ TEST         Given shipped / When cancel /     │
│              Then rejected                     │
└────────────────────────────────────────────────┘
```

Use it when the functionality is complex. A small change does not need it.

## Do not explain the pattern's name — explain the need

When an `Entity`, `Value Object`, `Aggregate`, `Domain Service`, `Domain Event`, `Repository`, `Factory`, `Specification` or `Bounded Context` shows up, do not define it. Answer in this order:

```
what business problem is there?
      ↓
what concept are we representing?
      ↓
why do we need this abstraction?
      ↓
what is it responsible for?
      ↓
what must it NOT do?
      ↓
where does it live in the code?
```

Bad:

> "We added a Repository because we use the Repository pattern."

Good:

> "We need to save orders without the domain knowing how they are stored. So we put an abstraction over persistence — which happens to be called a Repository."

```
NEED → ABSTRACTION → PATTERN → IMPLEMENTATION
```

The pattern is the answer, never the reason — this is the operational form of `§6`: *"patterns must always be explained as solutions to a problem, never as ends in themselves."* And if you cannot name the need, the abstraction is probably unnecessary.

## Layers are protections, not structure

Never present an architecture as a list of folders. Present it as *what it protects*:

```
DOMAIN  does not know about  SQL · HTTP · the ORM · the queue · the filesystem

why?
So that business rules can exist independently
of those technologies.

That is why dependencies point inwards:

  INFRASTRUCTURE → APPLICATION → DOMAIN
```

Reason first, structure second. Never the other way round.

## The use case is what justifies the handler

The user has to understand that `CancelOrderHandler` **does not exist because Clean Architecture says you should have handlers.** It exists because there is a use case to coordinate.

```
USE CASE   Cancel order
ACTOR      Customer
INPUT      OrderId
RULES      · the order exists
           · it has not shipped
           · it belongs to this customer
RESULT     the order becomes Cancelled
```

And then:

```
Use case → Handler → Domain → Persistence
```

## Rule → invariant → code → test

Important business rules are made explicit, and the chain down to the test must be visible:

```
RULE         A shipped order cannot be cancelled.
     ↓
INVARIANT    If Status == Shipped → Cancel() rejects.
     ↓
CODE         Order.Cancel()
     ↓
TEST         Given:  the order has shipped
             When:   we cancel it
             Then:   it fails, and it is still shipped
```

This is what stops a test from being *"one more test"* and makes it *"the proof of the rule we just implemented"*. When you write a meaningful test, say **which business behaviour it protects** before showing it.

## Events

Always connect to the problem. Never open with `DomainEventHandler` / `OutboxProcessor` / `MessagePublisher`:

```
BUSINESS      When an order is created, other systems must find out.
     ↓
PROBLEM       We do not want Order to depend on the external system.
     ↓
DECISION      We emit an event.
     ↓
DOMAIN        OrderCreated
     ↓
INFRA         publishing / outbox
     ↓
OUTSIDE       receives it
```

## Data

From the business concept to the model, and only then to tables. Never open with `CREATE TABLE`:

```
BUSINESS       A customer can have several orders.
     ↓
MODEL          Customer 1 ──── N Order
     ↓
PERSISTENCE    Customer table · Order table · CustomerId FK
```

## The standing question

For every meaningful architectural piece:

> **What need justifies this existing?**

If the code does not let you reconstruct that story on its own, the explanation has to.
