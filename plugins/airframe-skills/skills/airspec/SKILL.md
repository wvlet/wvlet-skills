---
name: airspec
description: Writes test cases using AirSpec, a functional testing framework for Scala. Use when writing tests, creating test suites, or when the user mentions AirSpec, unit tests, or test cases in Scala projects.
---

# Writing AirSpec Tests

```scala
import wvlet.airspec.*

class MyTest extends AirSpec {
  test("test description") {
    // test body
  }
}
```

## Assertions

| Assertion | Example |
|-----------|---------|
| `x shouldBe y` | `result shouldBe 42` |
| `x shouldNotBe y` | `result shouldNotBe null` |
| `x shouldBeTheSameInstanceAs y` | Reference equality |
| `x shouldBe defined` | Option is Some |
| `x shouldBe empty` | Collection/Option is empty |
| `x shouldContain y` | `list shouldContain "item"`, `"hello" shouldContain "ell"` |
| `x shouldNotContain y` | Works for collections and strings |
| `x shouldMatch { case ... => }` | Pattern matching |
| `intercept[E] { ... }` | `intercept[IllegalArgumentException] { throw ... }` |

## Test Control

- `pending("reason")` - Mark test as pending
- `skip("reason")` - Skip test execution
- `cancel("reason")` - Cancel test
- `fail("message")` - Fail immediately

## Nested Tests

```scala
test("outer") {
  val shared = setup()
  test("inner 1") { /* uses shared */ }
  test("inner 2") { /* uses shared */ }
}
```

## Dependency Injection

Use `initDesign` to configure dependencies. `AutoCloseable` objects are automatically closed after tests.

```scala
class DITest extends AirSpec {
  initDesign { design =>
    design
      .bindInstance[Database](mockDb)
      .bindSingleton[Service]
  }

  test("with dependency") { (db: Database) =>
    // db is injected
  }
}
```

Override per test:

```scala
test("custom design", design = _.bindInstance[Config](testConfig)) { (config: Config) =>
  // uses testConfig
}
```

## Lifecycle Hooks

```scala
override def beforeAll: Unit = { /* run once before all tests */ }
override def afterAll: Unit = { /* run once after all tests */ }
override def before: Unit = { /* run before each test */ }
override def after: Unit = { /* run after each test */ }
```

## Async Tests

Tests returning `Future[_]` or `Rx[_]` are automatically awaited.

## Property-Based Testing

```scala
import wvlet.airspec.*

class PropertyTest extends AirSpec with PropertyCheck {
  test("property") {
    forAll { (x: Int, y: Int) =>
      (x + y) shouldBe (y + x)
    }
  }
}
```

## Running Tests

```bash
sbt test                           # Run all tests
sbt "testOnly *MyTest"             # Run specific test class
sbt "testOnly -- pattern"          # Run tests matching pattern
sbt "testOnly -- -l debug"         # Set log level for test spec classes
sbt "testOnly -- -L com.example.MyClass=debug"  # Log level for specific class
```
