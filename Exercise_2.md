# Functional and Concurrent Programming with Scala and ZIO - SoSe 2021
#### Deadline 28.05.2021 13:00
---------------------
## Exercise 2

The aim of this exercise is to understand simple monads and their usage, practice pattern matching and implement sequential functional data structures. In the end we will use them to create a simple audio synthesis application.

Reuse the project you already set up in `Exercise 1` and move all previous sources to a package, for example `exercise1`. And create a new package for this exercise. 

Do not use any mutable variables (`var` keyword). 

Work in a separate branch and to submit this homework, create a pull request to the main branch and set me as assignee. Please do not merge the branch before I have reviewed it.


### Task 1 Pattern Matching and Monads
 
`Ior[E, A]` represents the inclusive or relationship. It is similar to `Either[E,A]` but it can contain either an `E` or a `A` or both (`E` and `A`). Similarly to `Either`, `Ior` is right-biased, which means that `map` and `flatMap` will work on the right side, in our case the `A` value. 

Your goal is to implement this `Ior` Monad. In order to make this task a bit easier, our monad will only accept `Throwable` as `E`'s (`Ior[A]`, where `E` is implied to be `Throwable`.)

#### A)
Create a sum (or) type named `Ior[A]` (sealed trait) and add possible values for it as case classes - `Left[A](elem:Throwable)`, `Right[A](elem:A)` and `Both[A](left:Throwable, elem:A)`.

Create a companion object named `Ior` and add three methods for construction of the different values - `right`, `left`, `both`.

### B) 
Create a `unit` method in the companion object. It should take an `A` and return `Ior[A]`

### C)
Add a `flatMap` method in the `Ior[A]` trait. It accepts a `f: A => Ior[B]` function and returns an `Ior[B]`. Remember that `flatMap` is right biased, so it only works on `A`. *Hint: you will probably need nested pattern matching for this*

### D) 
Add a `map` method, which accepts an `f: A => B` and returns an `Ior[B]` by combining `flatMap` and `unit`.

### Examples
Here are some examples of how `Ior` should behave.
```scala
val a = Ior.right(2) // Right(2)
val b = a.map(x => x * 4) // Right(8)

val c = b.flatMap(_ => Ior.right("a string")) //Right("a string")

val d = c.flatMap(_ => Ior.left[String](new RuntimeException("a grave error"))) //Left(java.lang.RuntimeException: a grave error)

val e = d.map(x => x + "something") //Left(java.lang.RuntimeException: a grave error)

val both = Ior.both(new RuntimeException("not fatal"), 21) //Both(java.lang.RuntimeException: not fatal,21)
val both1 = both.map(x => x * 2) //Both(java.lang.RuntimeException: not fatal,42)
val both2 = both.flatMap(_ => Ior.left[Int](new RuntimeException("fatal error"))) //Left(java.lang.RuntimeException: fatal error)
val both3 = both.flatMap(_ => Ior.right(480)) //Both(java.lang.RuntimeException: not fatal,480)
val both4 = both.flatMap(x => Ior.both(new RuntimeException("another not fatal"), x * 3)) //Both(java.lang.RuntimeException: another not fatal,63)

```

## Task 2 Functional Sequential Collections

### A)
Implement the `Stack` trait, as described in lecture 6. Make sure that all operations except `reverse` are constant time `O(1)` and `reverse` is linear time `O(n)`. 
```scala
trait StackLike[T] {
  def push(elem: T): StackLike[T]

  def pop(): Try[StackLike[T]]

  def top(): Option[T]

  def isEmpty: Boolean

  def reverse: StackLike[T]
}
```

### B)
Implement the `Queue` trait, as described in lecture 6 using two stacks. Make sure that `enqueue`, `front` and `isEmpty` are constant time `O(1)` and dequeue is amortized constant `amortized O(1)`. 
```scala
trait QueueLike[T] {
  def enqueue(elem: T): QueueLike[T]

  def dequeue(): Try[QueueLike[T]]

  def front(): Option[T]

  def isEmpty: Boolean
}
```

## Task 3 Application: Karplus-Strong Algorithm
Inspired heavily by [Josh Hug's Data Structures Course HW1](https://sp19.datastructur.es/materials/hw/hw1/hw1)

The [Karplus-Strong Algorithm](https://en.wikipedia.org/wiki/Karplus%E2%80%93Strong_string_synthesis) is a method of replicating the sound of a plucked string used in audio synthesis. It can be easily implemented using a queue like the one we implemented in the previous task.


The Karplus-Algorithm is simply the following three steps:
1. Replace every item in a queue with random noise (double values between `-0.5` and `0.5`).
2. Remove the front double in the queue and average it with the next double (hint: use `dequeue()` and `peek()`) multiplied by an energy decay factor of `0.996` (we’ll call this entire quantity newDouble). Then, add newDouble to the queue.

3. Play the double (newDouble) that you dequeued in step 2. Go back to step 2 (and repeat forever).


### A)
Create a new package called `lib` and create a class called `StdAudio.java` by copying it from the  [Princeton StdAudio.java](https://introcs.cs.princeton.edu/java/stdlib/StdAudio.java). This enables us to play a `double` value using the `StdAudio.play` method. 

 For example `StdAudio.play(0.333)` will tell the diaphragm of your speaker to extend itself to 1/3rd of its total reach, `StdAudio.play(-0.9)` will tell it to stretch its little heart backwards almost as far as it can reach. Movement of the speaker diaphragm displaces air, and if you displace air in nice patterns, these disruptions will be interpreted by your consciousness as pleasing thanks to billions of years of evolution. See [this page](https://electronics.howstuffworks.com/speaker6.htm) for more. 
 If you simply do `StdAudio.play(0.9)` and never play anything again, the diaphragm shown in the image would just be sitting still 9/10ths of the way forwards.
 [Explanation copied from Josh Hug's Task](https://sp19.datastructur.es/materials/hw/hw1/hw1#why-it-works)

### B )
Create a function `def whiteNoise(frequency:Int=440, volume:Double = 1.0): Queue[Double]` that returns a queue containing a total of `frequency` elements of random values between `.5` and `-.5` multiplied by `volume`. Frequency must be greater than zero and volume is between `0` and `1`. This is part 1) of the Kaplus-Strong algorithm description.

For this task you can use `val r = new Random()` to get random numbers.

### C) 
Create a function called `update` that performs the Karplus-Strong update, part 2) of the algorithm description. This function should be pure - it takes a `queue` and returns a new `queue`, the next step. 


### D)
Create a curried function called `loop` that takes a `queue` and a function `f:Double => Unit`, which can be used to play audio. It takes the `queue`, passes it to `update`, plays the front element of the queue and calls itself indefinitely.  
Make sure it is tail recursive, by marking it with `@tailrec` ([Info on tail recursion](https://www.scala-exercises.org/scala_tutorial/tail_recursion)), otherwise you will get a `StackOverflowException`.

This is part 3) of the Kaplus-Strong algorithm description.
### E)
In your main method, call `whiteNoise` to generate a starting queue. Call `loop` with the queue alongside the  `StdAudio.play` function. You should now hear something that sounds like an `A` string.


## Optional Task
Those tasks are optional and usually a lot harder than the normal tasks and just give additional, bonus points.

Implement the true `Ior` by making it generic - `Ior[E, A]` instead of just an `Ior[A]` with implied `E` of type `Throwable`. Also implement all of its operations described in Task 1.


## Goals
- :star: For solving Task 1 A)
- :star: For solving Task 1 B)
- :star: For solving Task 1 C)
- :star: For solving Task 1 D)
- :star::star: For solving Task 2 A)
- :star::star: For solving Task 2 B)
- :star: For solving Task 3 A)
- :star: For solving Task 3 B)
- :star: For solving Task 3 C)
- :star: For solving Task 3 D)
- :star: For solving Task 3 E)
- :star: For not using `var` anywhere
- :star: For having tests

## Bonus Stars
- :star2: for solving the optional task
