# Functional and Concurrent Programming with Scala and ZIO - SoSe 2021
#### Deadline 30.04.2021 13:00
---------------------
## Exercise 1

The aim of this exercise is to get comfortable with the scala syntax and try out some concepts already introduced in the lecture.

Reuse the project you already set up in `Exercise 0`. Implement all functions as functions of an `Object` and add simple tests for them using `ScalaTest`. Do not use any mutable variables (`var` keyword).

Also have a look at [Scala Worksheets](https://blog.jetbrains.com/scala/2012/12/04/scala-worksheet/), which provide an interactive shell ([REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop)), which can be used to ease development and aid understanding of what is going on in the code.

### Task 1
Implement the following methods using `fold` or `reduce` operations.
```scala
def max(arr: Array[Int]): Int = ???
```
```scala
def min(arr: Array[Int]): Int = ???
```
```scala
def sum(arr: Array[Int]): Int = ???
```

### Task 2
In the card game of Vingt-et-Un (Twenty-One) the aim is to come as close to `21` as possible, without going over it. It is played with a standard deck of cards (2 to King), where aces have a value of either 1 or 11 and all other picture cards (Jack, Queen, King) have a fixed value of 10.

#### A) 
Each card is represented as a a string: Number cards `"2"-"10"`, Ace - `"A"`, Jack - `"J"`, Queen - `"Q"`, King - `"K"`. 

Implement a function named `parse`, that takes a card (`String`) and returns an `Int`. When encountering an ace it should return `11`.

Implement another function, named `parseAll`, that takes an array of strings and calls `parse` for each element. *Hint: use `map`*

#### B)
Implement a function `values`, that takes a card (`Int`) and returns its possible values (`Array[Int]`). In case of aces an array containing 2 elements is returned (11 and 1). In all other cases, an array contains a single element is returned (with the value of the passed card/Integer).

#### C)
Implement a higher order curried function `determineHandValue`, that determines the `hand value` (return type `Int`). 
It should be curried and 
- accept a strategy function (`Array[Int] => Int`), which maps the `values` of a card to a single value 
- and the hand (`Array[Int]`).

Call `values` on each card of the hand, followed by the strategy function and in the end compute the `sum` using the function you already defined in Task 1. Note that the strategy function has no implementation. It is just a specification of a function, which will be provided in the next subtask.

Create a function `isBust` that takes the `hand value` and returns `true` if is greater than 21 or `false` otherwise.

#### D)
Partially apply `determineHandValue` by taking a strategy in order to form two function - `optimisticF`, by applying `max` as the strategy and `pessimisticF` by applying `min` as the strategy. Those functions now each take the `values` of a card and produce a single value (either the higher or lower in case of aces). 

### E)
Create a function `determineBestHandValue` which takes a hand and computes its value using `optimisticF` and if this results in a bust returns `pessimisticF` instead.

## Optional Task
Those tasks are optional and usually a lot harder than the normal tasks and just give additional, bonus points.

`determineBestHand` is a simplification. The hand `[A, K]` has two possible values `21[A(11)+K(10)]` or `11[A(1) + K(10)]`,
but the hand `[A, A, 9]` has three: `31[A(11) + A(11) + 9]`, `21[A(11) + K(10)]`, or `11[A(1) +A(1) + 9]`

`determineBestHandValue`, as described in Task 2, in the pessimistic case, would output 11
and in the optimistic 31, however it will never consider the actual best case - 21.

Update `determineBestHandValue`so it also covers that case.

*Hint: A way to solve this is using the flatten function does. We will learn this in future lectures.*

## Goals
- :star: For solving Task 1
- :star: For solving Task 2 A)
- :star: For solving Task 2 B)
- :star: For solving Task 2 C)
- :star: For solving Task 2 D)
- :star: For solving Task 2 E)
- :star: For reasonable currying in Task 2 C)
- :star: For correct partial application in Task 2 D)
- :star: For not using `var` anywhere
- :star: For having tests

## Bonus Stars
- :star2: for solving the optional task
