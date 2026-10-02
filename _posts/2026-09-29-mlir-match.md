---
title: "Stop Testing the Same Pattern Twice"
description: "Building an MLIR dialect for compiling pattern matching with decision trees in MLIR"
date: 2026-09-27
---

## I Like Pattern Matching

One of my favourite languages to write in is Scala, which has functional pattern matching as a particularly nice way to inspect values without having to write a pile of type checks and casts. Additionally, you can look inside a value and match on what's actually there.

Consider the following snippet in Scala:

```Scala
x match {
    case Some(0) => A
    case Some(1) => B
    case Some(n) => C
    case None => D
}
```

This is pretty useful for developers: we have four cases, and we want to do something different depending on what `x` looks like. It also becomes trivial to type check and easy to verify its exhaustiveness (for most cases).

Languages like OCaml, Haskell and now Rust have implemented functional pattern matching as part of their languages. I wanted MLIR to have a way to represent the same idea, so I built a dialect for representing functional pattern matching directly in MLIR.

At a glance, this is pretty simple. But how would a compiler actually implement this?

A simple way to compile this is to try each case in order, just like a human reading the code:

```
                x
                │
             Some(0)?
             /      \
           yes       no
            │         │
            A      Some(1)?
                    /     \
                  yes      no
                   │        │
                   B     Some(n)?
                          /      \
                        yes       no
                         │         │
                         C         D
```

That works for sure. But notice how for each individual check or *test*, we also check that `x` is a `Some` constructor multiple times. And as patterns get larger and more nested, the amount of repeated work can grow with them by a good amount.

What we might want ideally is something more like:

```
                x
                │
             Some?
             /   \
           no     yes
           │       │
           D    inspect value
                   │
              ┌────┼────┐
              │    │    │
              0    1    _
              │    │    │
              A    B    C
```
This is the same pattern that yields the same results and behaviour as the previous example. However, the overall number of tests needed to determine the result has decreased.

Instead of asking “Is it a Some?” three times, we ask it once and carry that information forward. Once we know `x` is a `Some`, all of the remaining cases can work from that knowledge.

This is the core idea behind **decision trees**. Each internal node represents a
test on the value being matched, each branch represents a possible outcome,
and each leaf represents the actual code to be run.

## Growing Trees

The natural way to construct such a tree is to start with all of the patterns
and repeatedly ask: **what should we inspect next?**

For the example above, the obvious first test is whether `x` is a `Some` or a `None`. That immediately divides the match into two groups:

* the `Some` cases
* the `None` case

Once we've established that `x` is a `Some`, there is no reason to keep considering the `None` pattern. We can focus entirely on the value contained inside the constructor.

The remaining patterns are now:

```text
0
1
_      // i.e. anything
```

We can inspect that value and split the cases again.

At each step, the compiler is narrowing down the set of patterns that could still match. A successful test gives us more information about the value, and that information lets us ignore patterns which are no longer relevant.

However, life isn't that simple. Consider:

```text
Some((0, x))
Some((1, y))
Some((n, z))
None
```

There are now several places we could inspect. We could start with the outer constructor, or eventually inspect one of the values inside the tuple. The choice matters because different tests can produce different decision trees.

Couple this with the fact that the compiler must preserve semantics such as arm ordering and bindings, and all of a sudden constructing a decision tree systematically is a problem in its own right.

Luckily **Maranget's pattern-matching compilation algorithm** gives us a systematic way to do exactly that.

## Big Up Maranget

The first step is to give the patterns a representation that makes this process easier to reason about. Maranget uses a **pattern matrix**.

Each row of the matrix represents one arm of the match. The columns represent the individual values that still need to be inspected.

For our original example, there is only one value to match, `x`, so the matrix initially has a single column:

```text
        ┌───────────┐
    A   │  Some(0)  │
    B   │  Some(1)  │
    C   │  Some(n)  │
    D   │  None     │
        └───────────┘
```

We can now choose a column to inspect. Since there is only one, the choice is easy: we inspect the patterns in that column.

Looking down it, we find two constructors: `Some` and `None`. We can therefore specialise the matrix for each of those possibilities.

For the `Some` case, we keep the rows that can match a `Some`, remove the constructor that we have just established, and expose its argument as a new value to match:

```text
        ┌───────────┐
    A   │     0     │
    B   │     1     │
    C   │     n     │
        └───────────┘
```

The `None` case is different. `None` has no arguments, so once we have established that the value is `None`, the corresponding arm can be selected immediately:

```text
    D → D
```

This is what **specialisation** does: it transforms the original matrix into a smaller matrix corresponding to one particular result of a test.

Patterns which do not match that result disappear. Patterns which do match are simplified to whatever remains to be checked.

Wildcards are slightly different. A wildcard matches any constructor, so when we specialise on a particular constructor, that row is retained and the wildcard is replaced by wildcards for the constructor's arguments.

For example, suppose we had:

```text
Some(0)
Some(_)
None
```

and specialised on `Some`. The resulting matrix would be:

```text
        ┌───────────┐
    A   │     0     │
    B   │     _     │
        └───────────┘
```

The wildcard is still applicable because it places no restriction on the value inside `Some`.

This becomes more interesting with constructors containing multiple values. Consider:

```text
Some((0, x))
Some((1, y))
Some((n, z))
None
```

After specialising on `Some`, we are left with the tuple:

```text
        ┌───────────────┐
    A   │     0, x      │
    B   │     1, y      │
    C   │     n, z      │
        └───────────────┘
```

The tuple itself can then be specialised, exposing its two fields as separate values:

```text
              first       second
        ┌─────────────┬─────────────┐
    A   │      0      │      x      │
    B   │      1      │      y      │
    C   │      n      │      z      │
        └─────────────┴─────────────┘
```

Now there are two columns. The first column represents the first tuple element and the second represents the second tuple element. Each is a value that can independently contribute information to the match.

This gives us a recursive process:

```text
pattern matrix
      │
      ▼
  choose column
      │
      ▼
  inspect patterns
      │
      ▼
   specialise
      │
   ┌──┼──┐
   ▼  ▼  ▼
  M₁  M₂  M₃
   │  │  │
   └──┴──┘
      │
      ▼
 compile each
 recursively
```

Each specialised matrix represents a smaller matching problem. We keep applying the same process until we reach a matrix where the next action is known.

There is one important detail here: once a pattern matrix has multiple columns, **which column should we choose?**

For the tuple above, choosing the first column gives us a useful distinction between `0` and `1`. But in a larger pattern there may be several columns that could be inspected, and choosing between them can produce very different trees. A good compiler therefore needs a heuristic for selecting the next column rather than simply choosing one arbitrarily.

This is an important part of Maranget's work. His 2008 paper explores heuristics based on **necessity**: roughly, whether inspecting a particular column is actually useful for distinguishing the remaining patterns, and how much that choice can reduce the resulting decision tree. Different choices can trade off the number of tests performed against the size and shape of the generated tree.

The basic algorithm can therefore be thought of as three repeating steps:

```text
1. Choose a column to inspect.

2. Specialise the matrix for the possible results of that test.

3. Recursively compile each specialised matrix.
```

Eventually, a row becomes the first applicable match. At that point there is no more pattern structure to inspect and the compiler can produce the corresponding action.

For our original match, the process gives us something like:

```text
                    x
                    │
               Some / None
               /         \
            Some          D
              │
          inspect payload
          /      |      \
         0       1       _
         │       │       │
         A       B       C
```

The important difference from the hand-written version is that we did not construct this tree by separately compiling four patterns. We started with the entire matrix and repeatedly transformed the remaining matching problem.

That gives us a concrete compilation procedure: represent the patterns as a matrix, select a useful value to inspect, specialise the matrix based on that test, and recursively compile the resulting problems.

This is the machinery I need before I can start talking about how to represent the same information in MLIR.


## Woo Baby It's Dialect Time
