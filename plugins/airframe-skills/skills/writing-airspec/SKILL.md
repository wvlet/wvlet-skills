---
name: writing-airspec
description: Writes test cases using AirSpec, a functional testing framework for Scala. Use when writing tests, creating test suites, or when the user mentions AirSpec, unit tests, or test cases in Scala projects.
---

# Writing AirSpec Tests

## Basic Test Structure

```scala
import wvlet.airspec.*

class MyTest extends AirSpec {
  test("test description") {
    // test body
  }
}
```

## Assertions

| Assertion | Description |
|-----------|-------------|
| `x shouldBe y` | Equality check |
| `x shouldNotBe y` | Inequality check |
| `x shouldBeTheSameInstanceAs y` | Reference equality |
| `x shouldBe defined` | Option is Some |
| `x shouldBe empty` | Collection/Option is empty |
| `x shouldContain y` | Collection/String contains element/substring |
| `x shouldNotContain y` | Collection does not contain |
| `x shouldMatch { case ... => }` | Pattern matching |
| `intercept[E] { ... }` | Expect exception of type E |

**String containment example:**

```scala
test("string contains substring") {
  val message = "Hello, World!"
  message shouldContain "World"
  message shouldNotContain "Goodbye"
}
```

## Test Control

- `pending("reason")` - Mark test as pending
- `skip("reason")` - Skip test execution
- `cancel("reason")` - Cancel test
- `fail("message")` - Fail immediately

## Nested Tests

Tests can be nested to share context:

```scala
class NestedTest extends AirSpec {
  test("outer") {
    val shared = setup()
    test("inner 1") {
      // uses shared
    }
    test("inner 2") {
      // uses shared
    }
  }
}
```

## Dependency Injection

Use `initDesign` to configure dependencies. Injected objects implementing `AutoCloseable` have their `close()` method automatically called after tests complete.

```scala
class DITest extends AirSpec {
  initDesign { design =>
    design
      .bindInstance[Database](mockDb)
      .bindSingleton[Service]
  }

  test("with dependency") { (db: Database) =>
    // db is injected
    // db.close() will be called automatically after tests
  }
}
```

Override design per test:

```scala
test("custom design", design = _.bindInstance[Config](testConfig)) { (config: Config) =>
  // uses testConfig
}
```

## Lifecycle Hooks

```scala
class LifecycleTest extends AirSpec {
  override def beforeAll: Unit = { /* run once before all tests */ }
  override def afterAll: Unit = { /* run once after all tests */ }
  override def before: Unit = { /* run before each test */ }
  override def after: Unit = { /* run after each test */ }
}
```

## Async Tests

Tests returning `Future[_]` or `Rx[_]` are automatically awaited:

```scala
test("async with Future") {
  someFutureOperation().map { result =>
    result shouldBe expected
  } // returns Future[_]
}

test("async with Rx") {
  someRxOperation().map { result =>
    result shouldBe expected
  } // returns Rx[_]
}
```

## Property-Based Testing

Mix in `PropertyCheck` for ScalaCheck integration:

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
# Run all tests
sbt test

# Run specific test class
sbt "testOnly *MyTest"

# Run tests matching pattern
sbt "testOnly -- pattern"

# Set log level for test classes
sbt "testOnly -- -l debug"

# Set log level for specific class
sbt "testOnly -- -L com.example.MyClass=debug"

# Multiple class patterns
sbt "testOnly -- -L com.example.*=trace -L com.other.Service=debug"
```

## Best Practices

- Use descriptive test names that explain the expected behavior
- One assertion per test when possible
- Use `initDesign` for test dependencies
- Group related tests using nesting
- Use `pending` for tests not yet implemented
