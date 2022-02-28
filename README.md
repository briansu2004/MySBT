# MySBT

My SBT

## Purpose

SBT is very important for the Scala development but it is under-estimated very much by many senior programmers.

I got lots of problems with different versions of ZIO, CATS Effects, FS2, NIO etc. So I feel it is important to make a good sbt repository.

## SBT For CATS Effects

Check ZIO releases at https://github.com/typelevel/cats-effect/releases

The major versions are 2.x and 3.x.

The latest version of Cats Effect 3 is 3.3.6.

The latest version of Cats Effect 2 is 2.4.1.

```
Cats Effect 3.0.0 is the first release in the 3.x lineage and the first ever major breaking change since 1.0. It represents a substantial leap forward for the library and for the ecosystem in numerous major areas, including safety, ergonomics, composability, and performance.

Even more excitingly, Cats Effect 3 represents a sturdy foundation on which we can continue to push the boundaries of what is possible within the Scala FP ecosystem! Every aspect of the library was carefully designed to foster simpler and safer usage patterns within downstream libraries. This design goal has touched almost every aspect of the API and nearly every semantic decision.

Acknowledgements

When Cats Effect 1.0.0 was released, it wasn't even clear that such a fundamentally foundational framework could exist in a space as complex as asynchronous effect systems. Over the years, we have learned that not only can such a framework exist, it is capable of fostering a thriving and innovative ecosystem for building high-performance, scalable, compositional software in an asynchronous and purely functional style. The success of Cats Effect in this space is owed in no small part to the tireless efforts of Alexandru Nedelcu, the maintainer of this project during that era. While Alex has since moved on to other endeavors, the lingering effects of his vision remain imprinted upon this library and the ecosystem as a whole.

The roots of the 3.0.0 project go back almost two and a half years. Countless people have influenced thinking and opinions on various aspects of the system. Many spirited debates and discussions were refined down to their essence and wrapped up into the whole. And of course, countless contributors have invested an enormous volume of their time into making this possible.

It is literally impossible to thank everyone who touched this project in some way. However, what follows are a list of just a few of the major contributors:

Vasil Vasilev (@vasilmkd)
Raas Ahsan (@RaasAhsan)
Fabio Labella (@SystemFw)
Tim Spence (@TimWSpence)
Michael Pilquist (@mpilquist)
Frank Thomas (@fthomas)
Ben Plommer (@bplommer)
Jakub Kozłowski (@kubukoz)
Emrys Ingersoll (@wemrysi)
Chris Davenport (@ChristopherDavenport)
So many of you have spent late nights tracking down race conditions, wrestling with algorithms and data structures that are nearly unprecedented, conceptualizing immensely complex abstractions and higher-order concepts in various instantiations and applications, engaging in enthusiastic discussion with countless users in various forums, and so much more. This triumph belongs to all of you.

I would also like to sincerely thank everyone who has, over the years, provided countless insightful discussions and opinions which have shaped the final result in ways both subtle and profound.

Overview

Cats Effect 3 represents an essentially complete rewrite of Cats Effect, starting from first principles and taking into account everything we've discovered and learned over the years. We went back to the drawing board on the most fundamental elements of the calculus and re-thought them from the ground up to be more compositional, more powerful, more expressive, and more amenable to high-performance implementations. As with any major reconstitution of a non-trivial framework, it would be impossible to enumerate every change individually. Thus, we will have to content ourselves with a few highlights.

If you have an interest in how the development of Cats Effect 3 unfolded, you are invited to peruse the following chronology. Each of the release notes linked below contains a thorough description of the major changes which arrived in the corresponding build:

Milestone 1
Milestone 2
Milestone 3
Milestone 4
Milestone 5
Release Candidate 1
Release Candidate 2
Release Candidate 3

New IO

Maybe we should have called it NIO instead of IO...

Cats Effect 3 includes a completely green field implementation of IO, designed from the ground up to support features such as tracing, fiber locals, composable cancelation, a high-performance runtime, and much more. Even on contrived synchronous benchmarks (such as flatMap and map), this new IO substantially exceeds the performance of Cats Effect 2's IO implementation, and on more realistic benchmarks which stress the fiber runtime more heavily, the performance gains can be massive.

Even more importantly, Cats Effect 3's IO puts a much stronger emphasis on ergonomics and completeness of user experience. The most commonly-noticed example of this in Cats Effect 2 was the lack of an IO.fromCompletableFuture function, taking a Java CompletableFuture[A] and producing an IO[A]. This function is fully present in Cats Effect 3, along with many other useful operations.

Additionally, IO no longer relies on confusing implicit structures such as ContextShift and Timer, nor does it have any need for tedious constructs such as Blocker. Instead, this functionality is built directly into the runtime and available out of the box in all modes of operation. For more information, see the Getting Started section of the (new and improved!) website.

Safe Typeclass Hierarchy

The typeclass hierarchy in Cats and Cats Effect represents more than just a system of abstractions which can be used to build programs in an abstracted manner. Rather, this hierarchy forms the fundamental building blocks and capabilities which give rise to an ecosystem which fits together smoothly and safely. If you spend enough time around the Typelevel ecosystem, you'll often hear people discussing "laws" and "lawfulness", or sometimes even "algebraic soundness". These are not just academic terms for flighty ideas that have little bearing on practical software. These principles, as heady and abstract as they may be, are the seeds which are tilled into the soil that bears the fruit of the ecosystem and the overall software development experience.

In Cats Effect 3, we started from the notion that every abstraction should have well-defined semantics and laws of substitutability which encode those semantics in terms of more fundamental primitives. We rigorously demanded of ourselves that we could express these semantics in their full richness directly and cleanly, across multiple compositional effects, and with multiple backing implementations.

Furthermore, we set out to resolve one of the most glaring weaknesses in the Cats Effect 2 typeclass hierarchy: the fact that Async and Sync rest at the top of the hierarchy, rather than at the bottom. In other words, every effect which exists within the Cats Effect 2 ecosystem must have the ability to capture asynchronous side-effects within Async. This is a significant handicap in reasoning and safety, not to mention the related concept of testability, since an asynchronous side-effect can feasibly be any action within your program. Thus, Concurrent for example gave you the ability to spawn cancelable fibers... as well as do literally anything else you might want using async.

In Cats Effect 3, Concurrent represents the ability to run cancelable fibers which can encode linear or non-linear dataflows... and absolutely nothing else. This along is very powerful, but it is still less powerful than "you can literally do anything you want". The Async typeclass sits at the absolute bottom of the hierarchy, where it belongs, and need not be summoned in most user code.

Compositional uncancelable

This particular semantic change may seem relatively small, but it represents an excellent example of a place in which subtle aspects of the core Cats Effect calculus can have profound impacts on the ecosystem writ large. In Cats Effect 2, uncancelable had the following signature:

def uncancelable[A](fa: F[A]): F[A]
This function provided a mechanism for masking cancelation within a lexical scope. Any effect or composition of effects passed to uncancelable would ignore the fiber cancel signal until exiting the uncancelable region. This was and is an important primitive, critical in the construction of resource-safe logic.

However, this kind of primitive is also very dangerous and limiting, since it makes it impossible to conditionally accept cancelation within the otherwise-uncancelable block. In Cats Effect 3, this weakness has been addressed with the addition of a relatively simple parameter: Poll:

def uncancelable[A](body: Poll[F] => F[A]): F[A]
To see why this is useful, consider the example of Semaphore. In order to avoid leaking permits in the face of cancelation, it is absolutely necessary to guarantee that, in all cases where we acquire a permit, we subsequently release once and only once. Thus, acquire falls into the classic resource acquisition pattern and would usually be wrapped in uncancelable (particularly if done using the conventional bracket operator).

Unfortunately, acquire also blocks its fiber (asynchronously) in the event that no permits are currently available. This is problematic for uncancelable because it could lead to deadlock, since that asynchronous blocking would not accept cancelation. Wrapping the acquire itself in a poll solves this problem by making the semaphore acquisition itself cancelable without corrupting the resource safety of the overall construction.

In the case of Semaphore, this has had immediate and profound impacts on its API. For example, the withPermit method is no longer required. Instead, a simpler permit method which returns a Resource is now possible. Additionally, this type of compositional cancelation, with support for sound nesting and unnesting of uncancelable and poll regions, has already made it possible to implement powerful generalizations and new constructions that would have never been conceivable in the past. As an example, Resource now forms an Async, meaning that combinators such as race can now be applied to Resource directly, solving a long-standing limitation of Cats Effect 2 in an incredibly general and orthogonal fashion.

We expect this simple semantic change will continue to have profound ripple effects throughout the ecosystem in the coming months and years.

Novel High-Performance Fiber Runtime

Special thanks are due to the Tokio project, who (to our knowledge) pioneered the application of coroutine-aware work-stealing task schedulers in a compositional runtime. Their designs and implementations were a substantial inspiration in the development of the Cats Effect 3 fiber runtime, and while the Cats Effect runtime has evolved and diverged substantially from this original starting point, it is absolutely undeniable that we would not be where we are today without Tokio's ground-breaking efforts.

One of the most noticeable new features in Cats Effect 3 isn't even something that most users will ever configure directly: the fiber runtime. You can read more about this general space in the blog post, Why are fibers fast?. To summarize briefly...

A fiber is nothing more than a potentially-unbounded series of effectful actions, based on IO, glued together in sequence using flatMap. Each action might be something like a synchronous effect (e.g. IO(readJdbcRow())), an asynchronous effect (e.g. IO.async_(onComplete => readFromSocket(onComplete))), an error (e.g. IO.raiseError(new ExistentialException("feelsbad"))), or many other things. The flatMap combinator represents the ability to run one action to completion, and thereafter run a subsequent action fully to completion. Thus, flatMap is very similar to the ; operator in Java or Scala, but generalized to operate on asynchronous actions just as effectively as synchronous ones, as well as a bunch of other goodies.

Any given application may have millions of fibers active at any point in time, each one representing its own sequence of actions. The job of the fiber runtime is to take all of these actions and map them down to the underlying processor threads efficiently and optimally, with a bare minimum of processing overhead and in such a fashion that applications can scale from commodity hardware and resource-constrained containers all the way up to massive single instances with hundreds of CPU cores.

In Cats Effect 3, this is accomplished with an extremely low-contention, lock-free work-stealing algorithm which is heavily optimized for the types of patterns commonly seen within IO programs. This kind of integration allows certain frequent patterns such as IO.cede (which is automatically inserted by the runtime periodically to ensure fairness), which the algorithm is able to encode in such a way that it has literally zero cost in the most common case. Various other elements of the algorithm scale astonishingly well as CPU core count increases.

Unlike typical scheduling algorithms (such as the one used within a conventional Executors.newFixedThreadPool), the Cats Effect 3 scheduler actually gets more efficient as the number of processors increases, while a conventional scheduler gets quadratically less efficient. These gains become startlingly noticeable even on modest commodity hardware. As an example, in a typical parallel scatter/gather microservice workflow, the Cats Effect 3 fiber runtime has been observed to run almost 55x more efficiently than the more conventional approaches available in other asynchronous libraries.

Full-Featured Standard Library

Going along with the theme of better ergonomics within the library as a whole, Cats Effect now provides a dramatically more complete "out of the box" experience. Numerous commonly-needed constructions such as Queue, CyclicBarrier, and such are now available in the new "standard library" (within the std module). These implementations are very high performance and production ready, and they form the foundation of complex extensional functionality in numerous downstream systems, such as Fs2.

Even simple things such as Random number generation or printing to the Console are now available without the need for third-party libraries.
```

## SBT For FS2

Check FS2 releases at https://github.com/typelevel/fs2/releases

The latest version of FS2 for Cats Effect 3 is 3.2.5, which supports Cats Effect 3 and is cross built for Scala 2.12, 2.13, and 3.0.

The latest version of FS2 for Cats Effect 2 is 2.5.10, which supports Cats Effect 2 and is similarly cross built for various Scala versions.

```
val catsVersion = "2.5.3"

libraryDependencies += "org.typelevel" %% "cats-effect" % catsVersion

val fs2Version = "2.5.10"

libraryDependencies += "co.fs2" %% "fs2-core" % fs2Version
libraryDependencies += "co.fs2" %% "fs2-io" % fs2Version // Node.js only
libraryDependencies += "co.fs2" %% "fs2-scodec" % fs2Version
```

## SBT For Circe

Check Circe releases at https://github.com/circe/circe/releases

```
val circeVersion = "0.13.0"

libraryDependencies ++= Seq(
  "io.circe" %% "circe-core" % circeVersion,
  "io.circe" %% "circe-generic" % circeVersion,
  "io.circe" %% "circe-parser" % circeVersion,
```

## SBT For ZIO

Check ZIO releases at https://github.com/zio/zio/releases

The major versions are 1.x and 2.x.

The latest version of ZIO 2 is 2.0.0-RC2.

The latest version of ZIO 1 is 1.0.13.

This is with very old version of v1 but a particular piece of code needs it to work.

```
libraryDependencies ++= Seq(
  "dev.zio" %% "zio" % "1.0.0-RC17",
  "dev.zio" %% "zio-streams" % "1.0.0-RC17",
  "com.propensive" %% "magnolia" % "0.12.6"
)
```

This is for ZIO v2.

```
libraryDependencies += "dev.zio" %% "zio" % "2.0.0-M4"
libraryDependencies += "dev.zio" %% "zio-streams" % "2.0.0-M4"
```

## SBT for NIO

```
lazy val zioVersion = "1.0.13"

libraryDependencies ++= Seq(
  "dev.zio"                %% "zio"                     % zioVersion,
  "dev.zio"                %% "zio-streams"             % zioVersion,
  //"dev.zio"                %% "zio-blocking"            % zioVersion,
  "org.scala-lang.modules" %% "scala-collection-compat" % "2.5.0",
  "dev.zio"                %% "zio-nio"                 % "2.0.0-RC2",
  "dev.zio"                %% "zio-test"                % zioVersion % Test,
  "dev.zio"                %% "zio-test-sbt"            % zioVersion % Test
)
```
