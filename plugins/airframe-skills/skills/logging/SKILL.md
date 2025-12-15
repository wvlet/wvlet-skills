---
name: logging
description: Implements logging using airframe-log for Scala applications. Use when adding logging, configuring log levels, or when the user mentions airframe-log, LogSupport, or logging in Scala.
---

# Using airframe-log

## Basic Usage

Mix in `LogSupport` trait:

```scala
import wvlet.log.LogSupport

class MyApp extends LogSupport {
  info("Application started")
  debug("Debug message")
  warn("Warning message")
  error("Error occurred")
  trace("Trace message")
}
```

Or create logger manually:

```scala
import wvlet.log.Logger

class MyApp {
  private val logger = Logger.of[MyApp]
  logger.info("message")
}
```

## Log Levels

From most to least verbose: `TRACE`, `DEBUG`, `INFO`, `WARN`, `ERROR`, `OFF`

```scala
import wvlet.log.{Logger, LogLevel}

// Set default level
Logger.setDefaultLogLevel(LogLevel.DEBUG)

// Set level for specific logger
Logger("com.myapp").setLogLevel(LogLevel.DEBUG)

// Set level for package
Logger("com.myapp.service").setLogLevel(LogLevel.TRACE)
```

## Command Line Configuration

Set log level for specific classes using `-L` option:

```bash
# Set log level for a specific class
sbt "testOnly *MyTest -- -L com.example.MyClass=debug"

# Multiple class patterns
sbt "testOnly *MyTest -- -L com.example.*=trace -L com.other.Service=debug"
```

## Configuration File

Create `log.properties` in resources:

```properties
# Root logger level
_root_=info

# Package-specific levels
com.myapp=debug
com.myapp.verbose=trace
```

Enable auto-reload:

```scala
Logger.scheduleLogLevelScan  // Scans every 1 minute
```

## Log Formatters

```scala
import wvlet.log._

// Source code location (default)
Logger.setDefaultFormatter(LogFormatter.SourceCodeLogFormatter)

// Include thread name
Logger.setDefaultFormatter(LogFormatter.ThreadLogFormatter)

// Tab-separated values
Logger.setDefaultFormatter(LogFormatter.TSVLogFormatter)

// Plain message only
Logger.setDefaultFormatter(LogFormatter.BareFormatter)
```

## Custom Formatter

```scala
import wvlet.log._

object MyFormatter extends LogFormatter {
  override def formatLog(r: LogRecord): String = {
    s"[${r.level}] ${r.getMessage}"
  }
}

Logger.setDefaultFormatter(MyFormatter)
```

## File Logging with Rotation

```scala
import wvlet.log._

logger.resetHandler(new LogRotationHandler(
  fileName = "app.log",
  maxNumberOfFiles = 100,
  maxSizeInBytes = 100 * 1024 * 1024  // 100MB
))
```

## Async Logging

```scala
import wvlet.log._

val asyncHandler = new AsyncHandler(baseHandler)
logger.resetHandler(asyncHandler)
```

## Lazy Evaluation

Log messages are evaluated only when the level is enabled:

```scala
// expensiveOperation() only called if DEBUG is enabled
debug(s"Result: ${expensiveOperation()}")
```

## Structured Logging

```scala
info(s"user=${userId}, action=${action}, status=${status}")
```

## Best Practices

- Use `LogSupport` trait for simplicity
- Configure levels via `log.properties` for production
- Use appropriate levels: `error` for failures, `warn` for recoverable issues, `info` for milestones, `debug` for troubleshooting
- Leverage lazy evaluation for expensive log messages
- Use file rotation in production environments
