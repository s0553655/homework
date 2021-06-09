# Functional and Concurrent Programming with Scala and ZIO - SoSe 2021
#### Deadline 11.06.2021 23:59
---------------------
## Exercise 3


The aim of this exercise is to better understand implicits, package objects and get more familiar with the IO monad via ZIO. 

Reuse the project you already set up in previous exercises and create a new package for this exercise. 

Do not use any mutable variables (`var` keyword). 

Work in a separate branch and to submit this homework, create a pull request to the main branch and set me as assignee. Please do not merge the branch before I have reviewed it.


## Task 1 Implicits and Package Objects
 
In this task you will implement conversions of integers to templerature values in various systems via implicits, such as `1.fahrenheit` in order to convert 1 °F to it's respective value in Celsius.

#### A)
Create a package named `temperature` and add a package object in it. In this package object add a type alias for `Double` called `Temperature`. 

Type aliases are nothing more but an alias (another name) for an existing type and are used in order to make code more readable. See this [tutorial](https://alvinalexander.com/scala/scala-type-aliases-syntax-examples/) for more info.

### B)
Using an implicit class or implicit methods, add the following 3 methods: `celsius`, `kelvin` and `fahrenheit`, which convert a given degree in the specified system into its value in Celsius. For example `273.15.kelvin` would result in `0.0`. 

Look up the conversion formulas online.

### C)

Add a method `def avg(other:Temperature)`, that can be invoked on a `Temperature` and returns the average of the two temperatures. 

E.g. "`1.celsius avg 10.fahrenheit`"

Add 2 constants:  `freezingPoint` and `absoluteZero`, equal to the freezing point of water and absolute zero, respectively.

### D) 

In the package object, create a method `display`, that takes a `Temperature` explicitly and implicitly takes a locale ( one of `US`, `SCI` (scientific) or `Other`) and returns a string representation of the temperature. Based on the locale, the temperature is displayed in Kelvin (°K) for `SCI`, Fahrenheit (°F) for `US` and otherwise (`Other`) in Celsius (°C).

Set the default locale to `Other`. This should not be done as a simple default value to the implicit parameter of `display` itself, but as an implicit variable in the package object.

### E)

Create a class outside the `temperate` package. Set a different locale (implicitly) and invoke some of the operations, available from `temperature`.

## Task 2 Getting Started with ZIO
In this task we will transform our "synthesizer" from Exercise 2 Task 3) into ZIO.

### A)
Include the ZIO library in your project and create an application entry point. Copy over the `update`, `loop` and `whiteNoise` methods.

### B)
Create a `play` method, that wraps the `StdAudio.play` method into a ZIO (`UIO[Unit]`).

### C)
Modify the `whiteNoise` method, so it uses `zio.random` for the generation and returns a `URIO[Random, Queue[Double]]`.

### D)

Modify the `loop` method, so it has a single parameter - the queue and returns a `ZIO[Random, Throwable, Unit]`. Make sure not to change `update`, it should still return an `Option` rather than a `ZIO`.

### E)
In the `run` method of your ZIO Application, create a program, that generates white noise via the `whiteNoise` method and then plays it via `loop`. 

## Optional Task
Those tasks are optional and usually a lot harder than the normal tasks and just give additional, bonus points.

Add tests for the `whiteNoise` method using `zio-test`. (We will cover testing with ZIO soon, but you can try to learn the basics on your own, [here](https://zio.dev/docs/usecases/usecases_testing) is an example) See [zio.test.TestRandom](https://zio.dev/docs/howto/test-effects#testing-random), which has a method `TestRandom.feedDoubles`.

## Goals
- :star: For solving Task 1 A)
- :star: For solving Task 1 B)
- :star: For solving Task 1 C)
- :star: For solving Task 1 D)
- :star: For solving Task 1 E)
- :star: For solving Task 2 A) + B)
- :star: For solving Task 2 C)
- :star: For solving Task 2 D)
- :star: For solving Task 2 E)
- :star: For not using `var` anywhere

## Bonus Stars
- :star2: for solving the optional task
