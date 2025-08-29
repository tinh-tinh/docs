---
sidebar_position: 2
---

# Serialization

Serialization is the process of transforming incoming request data (path, query, body) into Go types (structs) that handlers can use. In tinhtinh this is typically done either via Pipes (recommended) or direct Ctx helper functions.

This guide explains the available helpers, common patterns and the new AppOptions-based Scan Validator feature that lets you plug a custom validator to run automatically after parsing.

---

## Basics: Pipes vs direct functions

- Use Pipes when you want typed data available to the handler via ctx.Body(), ctx.Paths(), ctx.Queries(). Pipes are executed as middleware in the request pipeline and are the recommended approach for typed input + optional validation.
- Use direct Ctx functions (BodyParser / QueryParser / PathParser) inside handlers when you prefer to parse inline.

---

## Using Pipes (recommended)

When you register a parser Pipe on a controller or a route, it will parse and (optionally) validate input before the handler runs. After a Pipe runs, your handler can access parsed values via:

| Function | Description |
|---|---|
| ctx.Body() | Returns the parsed request body (parsed by BodyParser pipe) |
| ctx.Paths() | Returns parsed path parameters (parsed by PathParser pipe) |
| ctx.Queries() | Returns parsed query parameters (parsed by QueryParser pipe) |

Example — BodyParser Pipe (generic style used in repo tests):
```go
type CreateUserDto struct {
    Username string `json:"username" validate:"required"`
    Email    string `json:"email" validate:"required,email"`
}

ctrl.Pipe(core.BodyParser[CreateUserDto]{}).
    Post("/users", func(ctx core.Ctx) error {
        dto := ctx.Body().(*CreateUserDto)
        // dto is already parsed (and validated if a Scan Validator is configured)
        return ctx.JSON(core.Map{"username": dto.Username})
    })
```

Notes:
- Pipe types in the repository use generics e.g., core.BodyParser[T]{}.
- If a Scan Validator is configured (see below), it will be invoked after parsing and before the handler runs.

---

## Direct Ctx functions

You can parse inline inside a handler using the Ctx helpers. These return an error if parsing (or validation via Scan Validator) fails.

### BodyParser

Parse JSON (or supported body content) into a Go struct:

```go
type BodyData struct {
    Name string `json:"name" validate:"required"`
}

ctrl.Post("", func(ctx core.Ctx) error {
    var bodyData BodyData
    if err := ctx.BodyParser(&bodyData); err != nil {
        // err may be a parsing/validation error
        return ctx.Status(400).JSON(core.Map{"error": err.Error()})
    }
    return ctx.JSON(core.Map{"data": bodyData.Name})
})
```

### QueryParser

Parse query string values into a struct using `query` tags:

```go
type QueryData struct {
    Age    int  `query:"age" validate:"gte=0"`
    Format bool `query:"format"`
}

ctrl.Get("", func(ctx core.Ctx) error {
    var q QueryData
    if err := ctx.QueryParser(&q); err != nil {
        return ctx.Status(400).JSON(core.Map{"error": err.Error()})
    }
    return ctx.JSON(core.Map{"data": q})
})
```

### PathParser (ParamParser)

Parse path parameters into a struct using `path` tags:

```go
type ParamData struct {
    ID     int  `path:"id" validate:"gt=0"`
    Export bool `path:"export"`
}

ctrl.Get("{id}/{export}", func(ctx core.Ctx) error {
    var p ParamData
    if err := ctx.PathParser(&p); err != nil {
        return ctx.Status(400).JSON(core.Map{"error": err.Error()})
    }
    return ctx.JSON(core.Map{"id": p.ID})
})
```

---

## New feature: Custom Scan Validator via AppOptions

tinhtinh now supports registering a custom "Scan Validator" when you create your app. When provided, the validator is automatically called after successful parsing (BodyParser / QueryParser / PathParser and the Parser Pipes). If the validator returns an error, parsing is treated as failed and the error is returned to the handler chain (and handled by the framework's exception middleware).

This enables centralized, pluggable validation logic (for example, using go-playground/validator or any custom validation rules).

### How to configure

Pass a validator function when creating the app factory via AppOptions. The minimal expected shape is a function that accepts the parsed value and returns an error:

- Suggested signature: func(v any) error

Example using go-playground/validator/v10:

```go
import (
    "github.com/go-playground/validator/v10"
    "github.com/tinh-tinh/tinhtinh/v2/core"
)

// create a validator function
v := validator.New()
validateFn := func(val any) error {
    return v.Struct(val)
}

// create app with custom Scan Validator
app := core.CreateFactory(module, core.AppOptions{
    ScanValidator: validateFn,
})
```

Behavior:
- After BodyParser / QueryParser / PathParser parse a struct, tinhtinh will call ScanValidator(parsedValue).
- If ScanValidator returns nil, parsing succeeds and the handler is executed.
- If ScanValidator returns an error, the parsing function returns that error (so you can return 400 or let the exception middleware format it).

### Example with a DTO

```go
type CreateUserDto struct {
    Username string `json:"username" validate:"required"`
    Email    string `json:"email" validate:"required,email"`
}

ctrl.Post("", func(ctx core.Ctx) error {
    var dto CreateUserDto
    if err := ctx.BodyParser(&dto); err != nil {
        return ctx.Status(400).JSON(core.Map{"error": err.Error()})
    }
    // if we reach here dto passed validation via ScanValidator
    return ctx.JSON(core.Map{"ok": true})
})
```

If you prefer, ScanValidator can be any function that implements validation semantics you need (cross-field checks, custom tag parsing, conditional rules, etc.).

### Default behavior

- If no ScanValidator is configured in AppOptions, tinhtinh will not run extra validation automatically after parsing. Parsers will only perform basic parsing/conversion; any validation must be done manually inside middleware or handlers.

---

## Error handling & best practices

- Validation errors returned by the Scan Validator should be descriptive. The framework's exception middleware will format errors consistently if you return them.
- For request-level validation that needs to short-circuit with a specific HTTP response body, prefer adding a middleware that runs validation and returns a formatted error (ctx.Status(...).JSON(...)) instead of only relying on guard booleans.
- Use struct tags (e.g., `validate:"required"`) together with a go-playground/validator-backed ScanValidator for concise, declarative validation.

---

## Summary

- Prefer parser Pipes for typed input available to the handler via ctx.Body/ctx.Paths/ctx.Queries.
- Use the Ctx helpers (BodyParser / QueryParser / PathParser) for inline parsing.
- Configure a global Scan Validator in AppOptions to centralize validation logic and have it invoked automatically after parsing.
- If no validator is provided, perform explicit validation in middleware or handlers.

---

If you want, I can:
- add a small runnable example app demonstrating AppOptions.ScanValidator with go-playground/validator,
- or scan your repo for places where ScanValidator should be added and propose code changes.
