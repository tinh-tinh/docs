---
sidebar_position: 6
---

# Error Handler

Custom your error handler function you expected.

In default, TinhTinh support default error handler to catch event when `panic` to prevent crash app. And you can custom this function to get response you expected, by pass you function when create app.

```go
app := core.CreateFactory(module, core.AppOptions{
  ErrorHandler: func(err error, ctx core.Ctx) error {
    // your code
  },
})
```

With default, the response you receive will be like:

```json
{
  "statusCode": 500,
  "error": "Error Message",
  "timestamp": "2024-11-23T14:24:06+07:00",
  "path": "/api/users",
}
```

## Built-in HttpError

Tinh Tinh support some function for you panic error with response with http statusCode.
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