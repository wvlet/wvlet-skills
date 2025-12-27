---
name: airframe-log
description: Implements logging using airframe-log for Scala applications. Use when adding logging, configuring log levels, or when the user mentions airframe-log, LogSupport, or logging in Scala.
---

# Using airframe-log

## Basic Usage

```scala
import wvlet.log.LogSupport

class MyApp extends LogSupport {
  trace("Trace message")
  debug("Debug message")
  info("Info message")
  warn("Warning message")
  error("Error message")
}
```

Or manually:

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

Logger.setDefaultLogLevel(LogLevel.DEBUG)
Logger("com.myapp").setLogLevel(LogLevel.DEBUG)
```

## Command Line Configuration

```bash
sbt "testOnly *MyTest -- -L com.example.MyClass=debug"
sbt "testOnly *MyTest -- -L com.example.*=trace -L com.other.Service=debug"
```

## Configuration File

Create `log.properties` in resources:

```properties
_root_=info
com.myapp=debug
com.myapp.verbose=trace
```

Auto-reload: `Logger.scheduleLogLevelScan`

## Log Formatters

```scala
import wvlet.log._

Logger.setDefaultFormatter(LogFormatter.SourceCodeLogFormatter)  // Default
Logger.setDefaultFormatter(LogFormatter.ThreadLogFormatter)      // Include thread
Logger.setDefaultFormatter(LogFormatter.TSVLogFormatter)         // Tab-separated
Logger.setDefaultFormatter(LogFormatter.BareFormatter)           // Plain message
```

Custom:

```scala
object MyFormatter extends LogFormatter {
  override def formatLog(r: LogRecord): String = s"[${r.level}] ${r.getMessage}"
}
Logger.setDefaultFormatter(MyFormatter)
```

## File Logging with Rotation

```scala
import wvlet.log._

logger.resetHandler(new LogRotationHandler(
  fileName = "app.log",
  maxNumberOfFiles = 100,
  maxSizeInBytes = 100 * 1024 * 1024
))
```

## Async Logging

```scala
val asyncHandler = new AsyncHandler(baseHandler)
logger.resetHandler(asyncHandler)
```
