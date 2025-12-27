---
name: airframe-launcher
description: Builds command-line programs using airframe-launcher. Use when creating CLI applications, parsing command-line options, defining subcommands, or when the user mentions airframe-launcher or CLI development.
---

# Building CLI Programs

Build command-line interfaces using annotations on class constructors and methods.

## Basic CLI

```scala
import wvlet.airframe.launcher.*

class MyApp(
  @option(prefix = "-p,--port", description = "port number")
  port: Int = 8080,
  @option(prefix = "-h,--help", description = "show help", isHelp = true)
  help: Boolean = false
) {
  @command(isDefault = true)
  def run: Unit = {
    println(s"Starting server on port ${port}")
  }
}

object MyApp {
  def main(args: Array[String]): Unit = {
    Launcher.execute[MyApp](args)
  }
}
```

Run: `myapp --port 9000`

## Options

```scala
class MyApp(
  @option(prefix = "-v,--verbose", description = "verbose output")
  verbose: Boolean = false,
  @option(prefix = "-c,--config", description = "config file path")
  config: String = "config.yml",
  @option(prefix = "-n,--count", description = "number of items")
  count: Int = 10
)
```

## Arguments

Use `@argument` for positional arguments:

```scala
class MyApp(
  @argument(description = "input file")
  input: String,
  @argument(description = "output file")
  output: String = "out.txt"
)
```

Multiple arguments:

```scala
class MyApp(
  @argument(description = "input files")
  files: Seq[String] = Seq.empty
)
```

## Subcommands

Define commands as methods:

```scala
class MyApp(
  @option(prefix = "-h,--help", description = "show help", isHelp = true)
  help: Boolean = false
) {
  @command(description = "start the server")
  def start(
    @option(prefix = "-p,--port", description = "port number")
    port: Int = 8080
  ): Unit = {
    println(s"Starting on port ${port}")
  }

  @command(description = "stop the server")
  def stop: Unit = {
    println("Stopping server")
  }

  @command(description = "show status")
  def status: Unit = {
    println("Server status: running")
  }
}
```

Run:
- `myapp start --port 9000`
- `myapp stop`
- `myapp status`

## Default Command

Use `isDefault = true` for the command that runs when no subcommand is specified:

```scala
@command(isDefault = true, description = "default action")
def run: Unit = {
  println("Running default command")
}
```

## Nested Commands

Add nested command modules:

```scala
class MainApp {
  @command(isDefault = true)
  def help: Unit = println("Use: myapp <command>")
}

class UserCommands {
  @command(description = "list users")
  def list: Unit = { /* ... */ }

  @command(description = "add user")
  def add(@argument name: String): Unit = { /* ... */ }
}

class ConfigCommands {
  @command(description = "show config")
  def show: Unit = { /* ... */ }
}

object MyApp {
  def main(args: Array[String]): Unit = {
    Launcher.of[MainApp]
      .addModule[UserCommands]("user", "user management commands")
      .addModule[ConfigCommands]("config", "configuration commands")
      .execute(args)
  }
}
```

Run:
- `myapp user list`
- `myapp user add alice`
- `myapp config show`

## Help Generation

Setting `isHelp = true` on an option automatically displays usage information:

```scala
@option(prefix = "-h,--help", description = "show help", isHelp = true)
help: Boolean = false
```

## Packaging with sbt-pack

Create standalone executables:

**project/plugins.sbt:**
```scala
addSbtPlugin("org.xerial.sbt" % "sbt-pack" % "(version)")
```

**build.sbt:**
```scala
enablePlugins(PackPlugin)

packMain := Map(
  "myapp" -> "com.example.MyApp"
)
```

Run `sbt pack` to generate executables in `target/pack/bin/`.
