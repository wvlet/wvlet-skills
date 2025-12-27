---
name: building-rpc
description: Builds RPC servers and clients using Airframe RPC with Netty backend. Use when creating RPC services, defining RPC interfaces, generating RPC clients, or when the user mentions airframe-rpc or remote procedure calls.
---

# Building Airframe RPC

Build RPC services using plain Scala methods as endpoints with automatic client generation.

## Defining RPC Interface (api project)

Define a pure trait with `@RPC` annotation. Extend `RxRouterProvider` in the companion object to enable RPC client generation with sbt-airframe:

```scala
// api/src/main/scala/myapp/api/UserApi.scala
package myapp.api

import wvlet.airframe.http.*

@RPC
trait UserApi {
  def getUser(id: Long): User
  def createUser(name: String, email: String): User
}

object UserApi extends RxRouterProvider {
  override def router: RxRouter = RxRouter.of[UserApi]
}

case class User(id: Long, name: String, email: String)
```

## Implementing the Interface (server project)

```scala
// server/src/main/scala/myapp/server/UserApiImpl.scala
package myapp.server

import myapp.api.*
import wvlet.log.LogSupport

class UserApiImpl extends UserApi with LogSupport {
  def getUser(id: Long): User = {
    info(s"Getting user: ${id}")
    // implementation
  }

  def createUser(name: String, email: String): User = {
    // implementation
  }
}
```

## Starting a Server

```scala
import wvlet.airframe.http.netty.Netty

val router = RxRouter.of[UserApiImpl]

Netty.server
  .withRouter(router)
  .withPort(8080)
  .start { server =>
    server.awaitTermination()
  }
```

## Client Generation with sbt-airframe

### Project Structure

```
project/
  plugins.sbt       # Add sbt-airframe plugin
api/
  src/main/scala/   # RPC API definitions
server/
  src/main/scala/   # Server implementation (uses generated client)
build.sbt
```

### project/plugins.sbt

```scala
addSbtPlugin("org.wvlet.airframe" % "sbt-airframe" % "(version)")
```

### build.sbt

```scala
val AIRFRAME_VERSION = "(version)"

// API project: Define RPC interfaces
lazy val api = project
  .in(file("api"))
  .settings(
    libraryDependencies ++= Seq(
      "org.wvlet.airframe" %% "airframe-http" % AIRFRAME_VERSION
    )
  )

// Server project: Generate RPC clients and run server
lazy val server = project
  .in(file("server"))
  .enablePlugins(AirframeHttpPlugin)
  .settings(
    // Format: "package.name:rpc:GeneratedClientName"
    airframeHttpClients := Seq("myapp.api:rpc:ServiceRPC"),
    libraryDependencies ++= Seq(
      "org.wvlet.airframe" %% "airframe-http-netty" % AIRFRAME_VERSION
    )
  )
  .dependsOn(api)
```

### Using Generated Clients

```scala
import myapp.api.ServiceRPC

// Sync client
val client = ServiceRPC.newRPCSyncClient(
  Http.client.newSyncClient("localhost:8080")
)
val user = client.userApi.getUser(123)

// Async client
val asyncClient = ServiceRPC.newRPCAsyncClient(
  Http.client.newAsyncClient("localhost:8080")
)
val rx: Rx[User] = asyncClient.userApi.getUser(123)
```

## Error Handling

Throw `RPCStatus` exceptions to return errors. See [rpc-status-reference.md](rpc-status-reference.md) for all status codes.

```scala
@RPC
class MyApi {
  def process(input: String): Result = {
    if (input.isEmpty) {
      throw RPCStatus.INVALID_REQUEST_U1.newException("Input cannot be empty")
    }
    // process
  }
}
```

### Catching RPC Errors

```scala
try {
  client.myApi.process(input)
} catch {
  case e: RPCException =>
    println(s"Error: ${e.status} - ${e.message}")
}
```

## Filters

```scala
class AuthFilter extends RxHttpFilter {
  def apply(request: Request, next: RxHttpEndpoint): Rx[Response] = {
    if (isValidAuth(request.authorization)) {
      next(request)
    } else {
      throw RPCStatus.UNAUTHENTICATED_U13.newException("Invalid user")
    }
  }
}

val router = RxRouter.filter[AuthFilter].andThen[UserApiImpl]
```

## Accessing Request Context

```scala
def securedMethod(): String = {
  val request = RPCContext.current.httpRequest
  request.authorization match {
    case Some(auth) if isValid(auth) => "OK"
    case _ => throw RPCStatus.PERMISSION_DENIED_U14.newException("Denied")
  }
}
```

## Dependency Injection

```scala
class UserApiImpl(db: Database) extends UserApi with LogSupport {
  def getUser(id: Long): User = db.findUser(id)
  def createUser(name: String, email: String): User = { /* ... */ }
}

val router = RxRouter.of[UserApiImpl]

val design = newDesign
  .bindSingleton[Database]
  .add(Netty.server.withRouter(router).design)

design.build[NettyServer] { server =>
  server.waitServerTermination
}
```

## Supported Types

- Primitives: `Int`, `Long`, `String`, `Double`, `Float`, `Boolean`
- Case classes
- Collections: `Seq`, `List`, `Set`, `Map`, `Option`, `Either`, `Tuple`
- `java.util.UUID`, `java.time.Instant`

Schema-on-read: Automatic type conversion (e.g., `"100"` → `100`).
