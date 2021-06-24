# Functional and Concurrent Programming with Scala and ZIO - SoSe 2021
#### Deadline 02.07.2021 23:59
---------------------
## Exercise 4

The aim of this exercise is to better understand ZLayer and concurrency in ZIO. 

Reuse the project you already set up in previous exercises and create a new package for this exercise. 

Do not use any mutable variables (`var` keyword). 

Work in a separate branch and to submit this homework, create a pull request to the main branch and set me as assignee. Please do not merge the branch before I have reviewed it.

## Task 1 ZEnv

Given is the following code:
```scala
object Task1 extends zio.App {

  trait Config
  trait Logging
  trait Parsing
  trait Database
  trait Serialization
  trait UserService

  val configLive: ULayer[Has[Config]] = ???
  val userServiceLive: URLayer[Has[Database] with Has[Logging] with Has[Serialization], Has[UserService]] = ???
  val parsingLive: ULayer[Has[Parsing]] = ???
  val serializationLive: URLayer[Has[Parsing], Has[Serialization]] = ???
  val databaseLive: URLayer[Has[Config], Has[Database]] = ???
  val loggingLive: ULayer[Has[Logging]] = ???

  type MyEnv = Has[Database] with Has[Logging] with Has[Serialization] with Has[UserService] with Has[Parsing] with Has[Config]

  def f(): URIO[MyEnv, Unit] = ???

  override def run(args: List[String]): URIO[zio.ZEnv, ExitCode] = f().provideCustomLayer(/*TODO*/).exitCode
}
```

Compose the `ZLayer`s into the required environment with as few operators as you cam.

## Task 2 Debugging

Given is the following stack trace:

```scala
Fiber failed.
╥
╠══╦══╗
║  ║  ║
║  ║  ╠─A checked error was not handled.
║  ║  ║ Error
║  ║  ║ 
║  ║  ║ Fiber:Id(1623345663229,4) was supposed to continue to:
║  ║  ║   a future continuation at zio.ZIO.run(ZIO.scala:1691)
║  ║  ║   a future continuation at zio.ZIO.bracket_(ZIO.scala:256)
║  ║  ║ 
║  ║  ║ Fiber:Id(1623345663229,4) execution trace:
║  ║  ║   at zio.ZIO$.effectSuspendTotal(ZIO.scala:2728)
║  ║  ║   at zio.ZIO.bracket_(ZIO.scala:256)
║  ║  ║   at zio.internal.FiberContext.evaluateNow(FiberContext.scala:627)
║  ║  ║ 
║  ║  ║ Fiber:Id(1623345663229,4) was spawned by:
║  ║  ║ 
║  ║  ║ Fiber:Id(1623345663069,1) was supposed to continue to:
║  ║  ║   a future continuation at ex4.services.StackTraceEx$.run(StackTraceEx.scala:9)
║  ║  ║   a future continuation at zio.ZIO.exitCode(ZIO.scala:574)
║  ║  ║ 
║  ║  ║ Fiber:Id(1623345663069,1) execution trace:
║  ║  ║   at zio.ZIO.zipWithPar(ZIO.scala:2242)
║  ║  ║   at ex4.services.StackTraceEx$.run(StackTraceEx.scala:8)
║  ║  ║ 
║  ║  ║ Fiber:Id(1623345663069,1) was spawned by:
║  ║  ║ 
║  ║  ║ Fiber:Id(1623345662524,0) was supposed to continue to:
║  ║  ║   a future continuation at zio.App.main(App.scala:59)
║  ║  ║   a future continuation at zio.App.main(App.scala:58)
║  ║  ║ 
║  ║  ║ Fiber:Id(1623345662524,0) ZIO Execution trace: <empty trace>
║  ║  ║ 
║  ║  ║ Fiber:Id(1623345662524,0) was spawned by: <empty trace>
║  ║  ▼
║  ║
║  ╠─An interrupt was produced by #1.
║  ║ No ZIO Trace available.
║  ▼
▼
```

Write code that could produce something similar (ignore code lines, just similar fiber state/errors).
 
## Task 3 AutoZion

In the town of AutoZion robots go about their daily tasks and live in unison. 

At the start of each day a series of tasks is posted on the job board. Whenever a task is completed, it is recorded and the robot who finished it praised. Further, both are reported on the news. 

Each robot is represented as a fiber and has a different responsibility, based on its type.

Your task will be to implement one day in the life of the citizens of AutoZion.

### A) 
First off, create a package object with some convenient classes and aliases. 

Create a type alias ```MyEnv``` which will later hold our required custom ```ZIO``` environment. 

Create a trait `Robot`. Each robot has a `name` (String) and a work method with the following return type `ZIO[MyEnv, Any, Unit]`.

Create a sealed trait named `Job`. Add two case classes implementing it: `PendingJob`, which extends it with a duration (`zio.Duration`) and `CompletedJob`, which extends it with a `completedBy`(Robot) field. 

### B) 
Next implement the `JobBoard` for the robots as a `Zlayer` using the module pattern 1.0 or 2.0, as show in the lecture 9. Use a `ZIO unbounded Queue` internally for synchronization.
```scala
trait JobBoard {
  /**
  Submits a job to the job board, which can later be taken up by a robot using the take method.
  */
  def submit(job: PendingJob): UIO[Unit]

  /**
  Take a job from the job board
  */
  def take(): UIO[PendingJob]
}
```
### C) 
Implement the `CompletedJobsHub` for the robots as a `Zlayer` Use a `ZIO unbounded Hub` internally for synchronization. Whenever a robot completes a job, it broadcasts it via the `publish` method. If another robot is interested in completed jobs, it can subscribe via the `subscribe` method.
```scala
trait CompletedJobsHub {
  def subscribe: ZManaged[Any, Nothing, Dequeue[CompletedJob]]

  def publish(job: CompletedJob): UIO[Unit]
}
```

### D)
Implement the `News` service for the robots as a `Zlayer` Use a `ZIO unbounded Queue` internally for synchronization. It has two methods - `post` and `proclaim`. A robot can post some news via the `post` method and with `proclaim` the latest news can be retrieved.
```scala
trait News {
  def post(news: String): UIO[Unit]

  def proclaim(): UIO[String]
}
```

Update your `MyEnv` type alias to include all of those ZLayers.

### E)
Now it's time to implement the citizens of AutoZion - the robots. Implement each as a separate class, implementing the `Robot` trait. Each runs in a separate `zio.Fiber` and they communicate with each other based on the `ZLayer`s we already defined.

First we have the `Elder`s. They `publish` jobs to the `JobBoard` at specific intervals (`zio.Schedule`).


Next up, the `Worker`s. They `take` jobs from the `JobBoard`, execute them and once completed, `publish` them on the `CompletedJobsHub`. Each job takes `duration` to execute. Emulate the execution of the jobs by just sleeping for `duration` before taking the next job.

Next up we have the `Overseer`s. Whenever a job is completed, they `post` about it on the `News`. (including the name of the job and the robot who completed it)

The `Praiser`s are there to support the worker robots. Whenever a job is completed, they `post` a nice comment about the robot who completed it on the news. (including only the name of the robot, who completed it)

Finally, we have the `Reporter`. Whenever there are news, it `proclaim`s them (outputs the string to the console). For simplicity, it can just run synchronously in the main fiber (no need to `fork` it). 

### F)

Build a day in AutoZion as your ZIO Program. Build in a small delay before starting the `Elder`s and start the `Reporter` last synchronously(for simplicity). Make sure to have at least 2 `Worker`s and `Elder`s and at least one of each other type.

### G)
Make ZIO type parameters as concrete (e.g. `Has[News]` instead of `MyEnv` if only `News` is used in the work method) and concise (e.g `UIO[String]` for `ZIO[Any, Nothing, String]`) as possible.

## Optional Task
Those tasks are optional and usually a lot harder than the normal tasks and just give additional, bonus points.

The new `Worker`s of AutoZion have a software error! Now, while trying to complete a job, they sometimes (randomly, roughly 1 every 5 jobs) fail to do so. (implement this behaviour)

Make sure that if a job was taken and an error occurred, it is put back on the `JobBoard`.

The remedy to this is a small attachment, that reboots the `Worker`s if they shut down - `withReboot`. Once initiated, it takes 2 seconds to complete before the robot can take on jobs again.
(implement this behaviour)


## Goals
- :star: :star: For solving Task 1
- :star: :star: For solving Task 2
- :star: For solving Task 3 A)
- :star: For solving Task 3 B) 
- :star: :star: For solving Task 3 C)
- :star: For solving Task 3 D)
- :star: :star: :star: For solving Task 3 E)
- :star: For solving Task 3 F)
- :star: For solving Task 3 G)
- :star: For not using `var` anywhere

## Bonus Stars
- :star2: for solving the optional task
