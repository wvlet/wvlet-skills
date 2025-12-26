---
name: building-rest-http
description: Builds REST HTTP servers using airframe-http with Netty backend. Use when creating HTTP endpoints, REST APIs, RPC services, HTTP clients, or when the user mentions airframe-http, web servers, or API development.
---

# Building REST HTTP Servers

Build REST HTTP servers with Netty backend, automatic JSON/MessagePack serialization, and Airframe DI integration.

## Defining Endpoints

```scala
import wvlet.airframe.http.*

class MyApi {
  @Endpoint(method = HttpMethod.GET, path = "/v1/user/:name")
  def getUser(name: String): User = User(name)

  @Endpoint(method = HttpMethod.POST, path = "/v1/user")
  def createUser(request: CreateUserRequest): User = User(request.name)

  @Endpoint(method = HttpMethod.DELETE, path = "/v1/user/:id")
  def deleteUser(id: Long): Unit = { /* ... */ }
}
```

## Path Parameters

| Pattern | Description | Example |
|---------|-------------|---------|
| `:arg` | Single segment match | `/user/:id` matches `/user/123` |
| `*arg` | Tail match (end of path only) | `/files/*path` matches `/files/a/b/c` |

## Request Context

```scala
@Endpoint(method = HttpMethod.GET, path = "/info")
def getInfo(): Info = {
  val request = RPCContext.current.httpRequest
  Info(request.userAgent, request.header.get("X-Custom"))
}
```

## Custom Response

Return `Response` directly to customize headers and body:

```scala
@Endpoint(method = HttpMethod.GET, path = "/v1/download/:id")
def download(id: String): Response = {
  val content = getFileContent(id)
  Response(HttpStatus.Ok_200)
    .withHeader("Content-Disposition", s"attachment; filename=${id}.csv")
    .withHeader("X-Custom-Header", "value")
    .withContent(content)
}
```

## Starting a Server

```scala
import wvlet.airframe.http.netty.Netty

val router = RxRouter.of[MyApi]

Netty.server
  .withPort(8080)
  .withRouter(router)
  .start { server =>
    server.awaitTermination()
  }
```

## Combining Multiple APIs

```scala
val router = RxRouter.of[UserApi].andThen[ProductApi].andThen[OrderApi]
```

## Dependency Injection Integration

```scala
val design = Netty.server
  .withRouter(router)
  .withPort(8080)
  .design

design.build[NettyServer] { server =>
  server.waitServerTermination
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

val router = RxRouter.filter[AuthFilter].andThen[MyApi]
```

### Thread-Local Storage in Filters

```scala
// In filter
RPCContext.current.setThreadLocal("user_id", userId)

// In endpoint
val userId = RPCContext.current.getThreadLocal[String]("user_id")
```

## Error Handling

```scala
// Simple error
throw Http.serverException(HttpStatus.NotFound_404)

// With response body
case class ErrorResponse(code: Int, message: String)
throw Http.serverException(request, HttpStatus.Forbidden_403,
  ErrorResponse(100, "Access denied"))

// RPC status (auto-maps to HTTP status)
throw RPCStatus.INVALID_ARGUMENT_U3.newException("Invalid input")
```

## Static Content

```scala
@Endpoint(path = "/static/*path")
def serveStatic(path: String) =
  StaticContent.fromResource(basePath = "/public", path)
```

## HTTP Client

```scala
// Sync client
val client = Http.client.newSyncClient("http://localhost:8080")
val response = client.sendSafe(Http.GET("/v1/user/alice"))
val user = client.readAs[User](Http.GET("/v1/user/alice"))

// POST with JSON body
client.call[CreateUserRequest, User](
  Http.POST("/v1/user"),
  CreateUserRequest("bob")
)
```

### Client Configuration

```scala
Http.client
  .withRequestFilter(_.withAuthorization("Bearer token"))
  .withRetryContext(_.withMaxRetry(3))
  .withConnectionTimeout(Duration(30, TimeUnit.SECONDS))
  .newSyncClient("http://localhost:8080")
```

### Async Client

```scala
val client = Http.client.newAsyncClient("http://localhost:8080")
val rx: Rx[Response] = client.send(Http.GET("/v1/info"))
```

## Server Configuration

```scala
Netty.server
  .withName("my-server")
  .withPort(8080)
  .withRouter(router)
  .withExtraLogEntries { () =>
    ListMap("version" -> "1.0")
  }
  .noLogging  // Disable access logging
  .start { server => ... }
```

## MessagePack Support

Automatic MessagePack serialization when clients send:
- `Content-Type: application/msgpack` for requests
- `Accept: application/msgpack` for responses
