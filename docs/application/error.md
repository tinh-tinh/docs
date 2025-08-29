---
sidebar_position: 6
---

# Error Handler

Tinh Tinh centralizes error handling so that panics and returned errors from handlers/middleware are converted into HTTP responses in a consistent way. This document explains the default behavior (what the framework does out-of-the-box), how to provide a custom error handler, and recommended patterns based on the tinhtinh repository.

Relevant types / entry points
- ErrorHandler type (core): func(err error, ctx core.Ctx) error
- Default handler (core.ErrorHandlerDefault) — used when you don't provide a custom one.
- Configure via core.AppOptions.ErrorHandler when creating the app (core.CreateFactory).

---

## Default behavior

The default error handler converts an error into an HTTP-friendly JSON response. It uses the exception adapter (common/exception.AdapterHttpError) so errors produced by the common/exception helpers are respected (status & message). The default response body looks like:

```json
{
  "statusCode": 500,
  "error": "Error Message",
  "timestamp": "2024-11-23T14:24:06+07:00",
  "path": "/api/users"
}
```

The default handler sets the HTTP status code from the mapped exception instance and returns JSON containing:
- statusCode: integer HTTP status
- error: message string (or message derived from the exception)
- timestamp: RFC3339 timestamp of the error handling moment
- path: request path

Implementation note: core.ErrorHandlerDefault uses exception.AdapterHttpError(err) to convert generic errors into structured Http values and returns ctx.Status(...).JSON(...).

---

## When the error handler is invoked

- Any returned error from a route handler or middleware.
- Any panic recovered by the framework (panic(exception.ThrowHttp(...)) or plain panic).
- The framework will call the configured ErrorHandler(err, ctx) for these cases.

---

## Configure a custom error handler

You can supply your own ErrorHandler via AppOptions when creating the app factory:

```go
app := core.CreateFactory(module, core.AppOptions{
    ErrorHandler: func(err error, ctx core.Ctx) error {
        // custom handling
    },
})
```

The function should return an error (usually the result of ctx.Status(...).JSON(...) or another ctx response helper). The framework expects the handler to produce a response; returning nil means "no response produced here".

---

## Recommended custom handlers

1) Simple custom JSON envelope

```go
app := core.CreateFactory(module, core.AppOptions{
    ErrorHandler: func(err error, ctx core.Ctx) error {
        // map exceptions to structured HTTP error (same helper used by default)
        h := exception.AdapterHttpError(err)
        body := core.Map{
            "ok":      false,
            "message": h.Msg,
            "code":    h.Status,
            "path":    ctx.Req().URL.Path,
            "time":    time.Now().Format(time.RFC3339),
        }
        return ctx.Status(h.Status).JSON(body)
    },
})
```

2) Logging + error details (avoid leaking sensitive info in production)

```go
app := core.CreateFactory(module, core.AppOptions{
    ErrorHandler: func(err error, ctx core.Ctx) error {
        // log stack / details for internal monitoring
        log.Printf("err: %v, path: %s, method: %s", err, ctx.Req().URL.Path, ctx.Req().Method)

        // map to HTTP error if it's an encoded Http exception; fallback to 500
        httpErr := exception.AdapterHttpError(err)
        // in prod you might hide internal messages and return a generic message
        msg := httpErr.Msg
        if httpErr.Status == 500 {
            msg = "internal server error"
        }
        return ctx.Status(httpErr.Status).JSON(core.Map{"error": msg})
    },
})
```

3) Special handling for validation / known error types

If you use the common/exception helpers (e.g., exception.BadRequest, exception.TooManyRequests), those produce errors whose Error() is JSON. AdapterHttpError will parse them and set Status/Msg for you — use that in the handler to set appropriate HTTP status and body.

---

## Throwing/producing HTTP errors in handlers

Use the provided helpers in common/exception to create structured HTTP errors that the adapter understands. Examples:

```go
return exception.BadRequest("invalid payload")
panic(exception.Unauthorized("missing token"))
```

These helpers create an error whose .Error() is JSON representing status/message. The default adapter will turn those into correct status and message in the response.

---

## Practical tips & best practices

- Always return a response from your custom ErrorHandler (typically the result of ctx.Status(...).JSON(...)). This ensures the HTTP response is sent to the client.
- Use exception.AdapterHttpError(err) inside your custom handler to normalize between exception-created errors and plain errors.
- Log errors (with a trace id or request id) inside the ErrorHandler for observability. Avoid exposing stack traces or internal details in production responses.
- For special flows (e.g., returning different content types), your handler can inspect the request Accept header and emit HTML, plain text, or JSON accordingly.
- Keep the ErrorHandler lightweight — do most processing (e.g., notifying Sentry) in background goroutines if needed.
- For microservices / RPC components, note that there is a separate microservices.DefaultErrorHandler variant used in that context (signature differs).

---

## Tests & examples in the repository

- core/error_test.go contains tests that assert the default handler response shape and demonstrate registering a custom ErrorHandler in AppOptions.
- common/exception/http_test.go contains tests that exercise all built-in HTTP exception helpers (BadRequest, InternalServer, NotFound, etc.) and show usage patterns where handlers panic with exception.ThrowHttp(...) and the default handler returns the correct HTTP response.

Refer to those tests for concrete, working examples.

---

## Minimal example (complete)

```go
package main

import (
    "net/http"
    "time"
    "log"

    "github.com/tinh-tinh/tinhtinh/v2/core"
    "github.com/tinh-tinh/tinhtinh/v2/common/exception"
)

func main() {
    app := core.CreateFactory(AppModule, core.AppOptions{
        ErrorHandler: func(err error, ctx core.Ctx) error {
            // map/normalize
            h := exception.AdapterHttpError(err)

            // log for internal observability
            log.Printf("error: %v path=%s status=%d", err, ctx.Req().URL.Path, h.Status)

            // return JSON response
            return ctx.Status(h.Status).JSON(core.Map{
                "status":    h.Status,
                "message":   h.Msg,
                "timestamp": time.Now().Format(time.RFC3339),
                "path":      ctx.Req().URL.Path,
            })
        },
    })

    app.SetGlobalPrefix("/api")
    http.Handle("/", app.PrepareBeforeListen())
    http.ListenAndServe(":8080", nil)
}
```

---

If you want, I can:
- produce a small runnable example showing default vs custom ErrorHandler,
- scan your repo for handlers that return raw errors and suggest places to use common/exception helpers for consistent HTTP responses,
- or add a section showing how to adapt the response format for clients that expect {message: "..."} vs the default envelope.
