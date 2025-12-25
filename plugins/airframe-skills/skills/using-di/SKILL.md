---
name: using-di
description: Implements dependency injection using Airframe DI for Scala applications. Use when setting up dependency injection, managing object lifecycles, binding implementations, or when the user mentions Airframe DI, Design, or Session.
---

# Using Airframe DI

- **Design**: Type-to-implementation mappings (immutable)
- **Session**: Manages singleton instances and lifecycles

## Basic Usage

```scala
import wvlet.airframe._

class App(db: Database, config: Config) {
  def run(): Unit = { /* ... */ }
}

val design: Design = newDesign
  .bindImpl[Database, MySQLDatabase]
  .bindInstance[Config](Config.load())

design.build[App] { app =>
  app.run()
} // Session auto-closes
```

## Binding Types

| Binding | Usage |
|---------|-------|
| `.bindSingleton[A]` | Lazy singleton |
| `.bindInstance[A](x)` | Concrete instance |
| `.bindImpl[A, AImpl]` | Interface to implementation |
| `.bindProvider { (d: D) => new A(d) }` | Provider function (up to 5 deps) |

## Combining Designs

```scala
val combined = baseDesign + overrideDesign  // Right overrides left
```

## Lifecycle Hooks

```scala
val design: Design = newDesign
  .bindSingleton[Service]
  .onInit { s => /* initialize */ }
  .onStart { s => /* on session start */ }
  .onShutdown { s => /* cleanup */ }
```

`AutoCloseable` objects have `close()` called automatically when the session ends.

## Tagged Types

```scala
import wvlet.airframe.surface.tag._

trait Prod
trait Test

class Service(
  prodConfig: Config @@ Prod,
  testConfig: Config @@ Test
)

val design: Design = newDesign
  .bindInstance[Config @@ Prod](prodConfig)
  .bindInstance[Config @@ Test](testConfig)
```

## Child Sessions

```scala
design.withSession { session =>
  val childDesign = newDesign.bindInstance[X](override)
  session.withChildSession(childDesign) { child =>
    child.build[App]
  }
}
```

## Session Modes

```scala
val design: Design = newDesign
  .withProductionMode  // Eager singletons
  .withLazyMode        // Lazy singletons (default)
```
