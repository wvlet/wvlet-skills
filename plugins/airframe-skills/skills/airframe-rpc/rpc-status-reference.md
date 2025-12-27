# RPC Status Reference

## Success

| Status | HTTP | Description |
|--------|------|-------------|
| `SUCCESS_S0` | 200 | OK |

## User Errors (U)

Client-side errors that can be fixed by the caller.

| Status | HTTP | Description |
|--------|------|-------------|
| `USER_ERROR_U0` | 400 | Generic user error |
| `INVALID_REQUEST_U1` | 400 | Invalid request format |
| `INVALID_ARGUMENT_U2` | 400 | Invalid argument value |
| `SYNTAX_ERROR_U3` | 400 | Syntax error in request |
| `OUT_OF_RANGE_U4` | 400 | Value out of valid range |
| `NOT_FOUND_U5` | 404 | Resource not found |
| `ALREADY_EXISTS_U6` | 409 | Resource already exists |
| `NOT_SUPPORTED_U7` | 405 | Operation not supported |
| `UNIMPLEMENTED_U8` | 405 | Method not implemented |
| `UNEXPECTED_STATE_U9` | 400 | Unexpected state |
| `INCONSISTENT_STATE_U10` | 400 | Inconsistent state |
| `CANCELLED_U11` | 499 | Request cancelled by client |
| `ABORTED_U12` | 409 | Operation aborted |
| `UNAUTHENTICATED_U13` | 401 | Authentication required |
| `PERMISSION_DENIED_U14` | 403 | Permission denied |

## Internal Errors (I)

Server-side errors that the caller cannot fix.

| Status | HTTP | Description |
|--------|------|-------------|
| `INTERNAL_ERROR_I0` | 500 | Internal server error |
| `UNKNOWN_I1` | 500 | Unknown error |
| `UNAVAILABLE_I2` | 503 | Service unavailable |
| `TIMEOUT_I3` | 504 | Operation timeout |
| `DEADLINE_EXCEEDED_I4` | 504 | Deadline exceeded |
| `INTERRUPTED_I5` | 500 | Operation interrupted |
| `SERVICE_STARTING_UP_I6` | 503 | Service is starting up |
| `SERVICE_SHUTTING_DOWN_I7` | 503 | Service is shutting down |
| `DATA_LOSS_I8` | 500 | Data loss or corruption |

## Resource Exhausted (R)

Resource limit errors, typically retryable after backoff.

| Status | HTTP | Description |
|--------|------|-------------|
| `RESOURCE_EXHAUSTED_R0` | 429 | Generic resource exhausted |
| `OUT_OF_MEMORY_R1` | 429 | Out of memory |
| `EXCEEDED_RATE_LIMIT_R2` | 429 | Rate limit exceeded |
| `EXCEEDED_CPU_LIMIT_R3` | 429 | CPU limit exceeded |
| `EXCEEDED_MEMORY_LIMIT_R4` | 429 | Memory limit exceeded |
| `EXCEEDED_TIME_LIMIT_R5` | 429 | Time limit exceeded |
| `EXCEEDED_DATA_SIZE_LIMIT_R6` | 429 | Data size limit exceeded |
| `EXCEEDED_STORAGE_LIMIT_R7` | 429 | Storage limit exceeded |
| `EXCEEDED_BUDGET_R8` | 429 | Budget exceeded |

## Usage Examples

### Throwing Errors

```scala
// Not found
throw RPCStatus.NOT_FOUND_U5.newException("User not found")

// Invalid argument with context
throw RPCStatus.INVALID_ARGUMENT_U2.newException(s"Invalid email: ${email}")

// Authentication required
throw RPCStatus.UNAUTHENTICATED_U13.newException("Please login first")

// Rate limiting
throw RPCStatus.EXCEEDED_RATE_LIMIT_R2.newException("Too many requests")
```

### Catching Errors

```scala
try {
  client.api.method()
} catch {
  case e: RPCException =>
    e.status match {
      case RPCStatus.NOT_FOUND_U5 => handleNotFound()
      case RPCStatus.UNAUTHENTICATED_U13 => redirectToLogin()
      case RPCStatus.PERMISSION_DENIED_U14 => showAccessDenied()
      case s if s.isUserError => showUserError(e.message)
      case s if s.isInternalError => showServerError()
      case s if s.isResourceExhausted => retryWithBackoff()
      case _ => handleGenericError(e)
    }
}
```

### Checking Error Categories

```scala
val status: RPCStatus = e.status

status.isSuccess          // S* codes
status.isUserError        // U* codes (4xx)
status.isInternalError    // I* codes (5xx)
status.isResourceExhausted // R* codes (429)
```
