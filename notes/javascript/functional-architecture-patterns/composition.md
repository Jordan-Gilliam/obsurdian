# Composition

## Abstract Algebra

### Group Definition
A group is a set, G, together with an operation ⋅ (called the group law of G) that combines any two elements a and b to form another element, denoted a ⋅ b or ab. To qualify as a group, the set and operation, (G, ⋅), must satisfy four requirements known as the group axioms:

**Closure**
For all a, b in G, the result of the operation,
`a ⋅ b`, is also in `G`.

**Associativity**
For all `a`, `b` and `c` in `G`:
``(a ⋅ b) ⋅ c = a ⋅ (b ⋅ c)``

**Identity element**
There exists an element `e` in `G` such that, for every element `a` in `G`, the equations
`e ⋅ a = a` and `a ⋅ e = a` hold. 
Such an element is unique (see below), and thus one speaks of the identity element.

**Inverse element**
For each `a` in `G`, there exists an element `b` in `G` such that `a ⋅ b = e` and `b ⋅ a = e`, where `e` is the identity element. For each `a`, the `b` is unique and it is commonly denoted `a−1` (or `−a`, if the operation is denoted "+").

The result of the group operation may depend on the order of the operands. In other words, the result of combining element `a` with element `b` need not yield the same result as combining element `b` with element `a`; the equation

`a ⋅ b = b ⋅ a`

### Guiding Principle
use Composition (mostly)

#js #functional-programming #architecture #pattern