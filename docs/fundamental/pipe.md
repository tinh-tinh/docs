---
sidebar_position: 6
---

# Pipe

A **pipe** in Tinh Tinh is a middleware mechanism for **validating and transforming incoming request data** before it reaches your controller logic. Pipes ensure that all data passed to your handlers is type-safe, validated, and ready to use—reducing boilerplate and improving application reliability.

![pipe](./img/pipe.png)

## Using Generic Pipe Helpers

Tinh Tinh provides generic helpers like `core.BodyParser[P]`, `core.QueryParser[P]`, and `core.PathParser[P]`, allowing you to write type-safe, concise, and composable request parsing and validation logic.

### Example: Body Pipe with `core.BodyParser`

Define your DTO:

```go
type SignUpDto struct {
    Name     string `validate:"required"`
    Email    string `validate:"required,isEmail"`
    Password string `validate:"isStrongPassword"`
    Age      int    `validate:"isInt"`
}
```

Apply the body parser pipe in your controller using generics:

```go
ctrl.Pipe(core.BodyParser[SignUpDto]{}).Post("/signup", func(ctx core.Ctx) error {
    dto := ctx.Body().(*SignUpDto)
    // dto is already validated and parsed
    return ctx.JSON(core.Map{"data": dto})
})
```

### Example: Query Pipe with `core.QueryParser`

```go
type FilterDto struct {
    Name  string `validate:"required" query:"name"`
    Email string `validate:"required,isEmail" query:"email"`
    Age   int    `validate:"isInt" query:"age"`
}

ctrl.Pipe(core.QueryParser[FilterDto]{}).Get("/users", func(ctx core.Ctx) error {
    dto := ctx.Queries().(*FilterDto)
    return ctx.JSON(core.Map{"data": dto})
})
```

### Example: Path Pipe with `core.PathParser`

```go
type PathDto struct {
    ID int `validate:"required,isInt" path:"id"`
}

ctrl.Pipe(core.PathParser[PathDto]{}).Get("/users/{id}", func(ctx core.Ctx) error {
    dto := ctx.Paths().(*PathDto)
    return ctx.JSON(core.Map{"data": dto})
})
```

## Summary

- **Generic pipe helpers** (`core.BodyParser[P]{}`, `core.QueryParser[P]{}`, `core.PathParser[P]{}`) provide type safety and concise syntax for parsing and validating request data.
- Access parsed data inside your handler using `ctx.Body()`, `ctx.Queries()`, or `ctx.Paths()`—always as a pointer to your DTO.
- This approach is recommended for most use cases in Tinh Tinh.

---

For more on advanced serialization and validation, see [Serialization](../application/serialization.md) and [Validation](../fundamental/validation.md).
