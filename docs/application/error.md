---
sidebar_position: 6
---

# Error Handler

Customize your error handler function as you expect.

By default, Tinh Tinh supports a default error handler to catch events when `panic` occurs, preventing the app from crashing. You can customize this function to get the response you expect by passing your function when creating the app.

```go
app := core.CreateFactory(module, core.AppOptions{
  ErrorHandler: func(err error, ctx core.Ctx) error {
    // your code
  },
})
```

By default, the response you receive will look like:

```json
{
  "statusCode": 500,
  "error": "Error Message",
  "timestamp": "2024-11-23T14:24:06+07:00",
  "path": "/api/users"
}
```

## Built-in HttpError

Tinh Tinh supports some functions for you to panic with an HTTP response status code:
- BadRequest
- Unauthorized
- NotFound
- Forbidden
- NotAcceptable
- RequestTimeout
- Conflict
- Gone
- HttpVersionNotSupported
- PayloadTooLarge
- UnsupportedMediaType
- UnprocessableEntity
- InternalServerError
- NotImplemented
- ImATeapot
- MethodNotAllowed
- BadGateway
- ServiceUnavailable
- GatewayTimeout
- PreconditionFailed

Example:

```go
import (
	"github.com/tinh-tinh/tinhtinh/v2/common/exception"
	"github.com/tinh-tinh/tinhtinh/v2/core"
)

// something

ctrl.Get("", func(ctx core.Ctx) error {
  return exception.BadRequest("bad request")
})
```