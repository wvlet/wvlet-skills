---
name: using-di
description: Implements dependency injection using Airframe DI for Scala applications. Use when setting up dependency injection, managing object lifecycles, binding implementations, or when the user mentions Airframe DI, Design, or Session.
---

# Using Airframe DI

## Core Concepts

- **Design**: Specifies type-to-implementation mappings (immutable)
- **Session**: Manages singleton instances and lifecycles

## Basic Usage

```scala
import wvlet.airframe._

// Define dependencies via constructor
class App(db: Database, config: Config) {
  def run(): Unit = { /* ... */ }
}

// Create design
val design: Design = newDesign
  .bindImpl[Database, MySQLDatabase]
  .bindInstance[Config](Config.load())

// Build and use
design.build[App] { app =>
  app.run()
} // Session auto-closes
```

## Binding Types

| Binding | Usage | Description |
|---------|-------|-------------|
| `.bindSingleton[A]` | `bindSingleton[A]` | Bind A to singleton instance |
| `.bindInstance[A](x)` | `bindInstance[A](new A)` | Bind to concrete instance |
| `.bindImpl[A, AImpl]` | `bindImpl[A, AImpl]` | Bind interface to implementation |
| `.bindProvider { ... }` | `bindProvider { (d: D) => new A(d) }` | Bind using provider function |

## Provider Binding

Providers can take up to 5 dependencies:

```scala
val design: Design = newDesign
  .bindProvider { (d1: D1) => new P(d1) }
  .bindProvider { (d1: D1, d2: D2) => new P(d1, d2) }
  .bindProvider { (d1: D1, d2: D2, d3: D3) => new P(d1, d2, d3) }
  // Up to 5 arguments supported
```

## Combining Designs

Designs are immutable; combine with `+`:

```scala
val combined = baseDesign + overrideDesign
// Right-hand bindings override left-hand
```

## Lifecycle Hooks

```scala
val design: Design = newDesign
  .bindSingleton[Service]
  .onInit { s => /* initialize */ }
  .onStart { s => /* on session start */ }
  .onShutdown { s => /* cleanup */ }
```

Objects implementing `AutoCloseable` have `close()` called automatically when the session ends, ensuring proper resource cleanup.

## Tagged Types

Distinguish identical types using tags:

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

Create scoped sessions with overrides:

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

## Best Practices

- Use constructor injection exclusively
- Keep application logic injection-agnostic
- Define reusable modules as Design objects
- Use tagged types to differentiate same-type dependencies
- Prefer `bindSingleton` for lazy initialization
- Use lifecycle hooks for resource management
