---
sidebar_position: 5
---

# Middleware

Tinh Tinh middleware lets you run logic before (or around) route handlers. You can register middleware at multiple scopes (module, controller, route) and selectively apply them with Consumers. Middleware may also access module providers (DI) — note the DI pattern changed: MiddlewareRef is deprecated; use ctx.Ref(core.Provide(...)) from inside a normal middleware instead.

![middleware](./img/middleware.png)

This guide describes the improved middleware flow and shows practical examples drawn from the tinhtinh core tests and helpers.

---

## Highlights / Quick summary

- Middleware is any function that accepts a core.Ctx and returns an error.
- Middleware signatures:
  - Simple middleware: `func(ctx core.Ctx) error` — preferred
- DI-aware middleware: instead of the old `MiddlewareRef` signature, call `ctx.Ref(...)` inside a normal middleware to access providers/services.
- Middleware can be registered at:
  - Module-level (applies to all controllers in the module)
  - Controller-level (applies to all routes in the controller)
  - Route-level (applies only to a specific route)
  - Consumer (apply middleware selectively across routes/modules with include/exclude rules)
- Execution order is deterministic. Middleware forms a stack — outer → inner. See "Ordering & Execution" section.
- Middleware should call `ctx.Next()` to continue the chain. Returning an error (or panicking) halts execution and the framework will convert it into an HTTP error response.

---

## 1. Middleware signature (current)

Preferred, modern signature:

```go
func MyMiddleware(ctx core.Ctx) error {
    // do something with ctx
    return ctx.Next()
}
```

If you need a provider/service, resolve it inside the middleware:

```go
func MyAuthMiddleware(ctx core.Ctx) error {
    // Resolve a provider exported as core.Provide("AuthService")
    svc := ctx.Ref(core.Provide("AuthService"))
    if svc == nil {
        // handle missing provider
        return ctx.Status(http.StatusInternalServerError).JSON(core.Map{"error":"service unavailable"})
    }
    auth := svc.(*AuthService) // type assert as appropriate
    if !auth.Check(ctx.Req()) {
        return ctx.Status(http.StatusForbidden).JSON(core.Map{"error":"forbidden"})
    }
    // attach info to ctx for downstream handlers
    ctx.Set("user", auth.UserFromRequest(ctx.Req()))
    return ctx.Next()
}
```

Note: MiddlewareRef (functions of the form `func(ref core.RefProvider, ctx core.Ctx) error`) is deprecated. Replace those with the pattern above.

---

## 2. Register middleware

1) Route-level (single route)

```go
ctrl := module.NewController("user")
ctrl.Use(AuthMiddleware).Get("/profile", func(ctx core.Ctx) error {
    return ctx.JSON(core.Map{"msg": "with middleware"})
})
```

2) Controller-level (apply to all routes in a controller)

There are two common styles:

- Fluent style (call Registry to finalize the controller):

```go
ctrl := module.NewController("admin").
    Use(AdminOnlyMiddleware).
    Registry()

ctrl.Get("/stats", statsHandler)
ctrl.Get("/logs", logsHandler)
```

Important: when using the fluent `.Use(...).Registry()` pattern, call `.Registry()` as the final method of the controller builder chain. Registry finalizes and registers the controller with the module so subsequent route definitions become part of that controller and inherit the configured middleware. If `.Registry()` is omitted or called incorrectly, the controller might not be registered and middleware might not apply.

- Direct style (attach middleware then define routes directly):

```go
ctrl := module.NewController("admin")
ctrl.Use(AdminOnlyMiddleware)

ctrl.Get("/stats", statsHandler)
ctrl.Get("/logs", logsHandler)
```

Both styles achieve the same end result; the key is ensuring `.Registry()` is used when you rely on the builder/fluent API to register the controller.

3) Module-level (apply to every controller in a module)

```go
mod := core.NewModule(core.NewModuleOptions{
    Middlewares: []core.Middleware{LoggingMiddleware},
})
```

4) Consumer — selective application across a module tree (include/exclude rules):

```go
app := core.NewModule(core.NewModuleOptions{
    Imports: []core.Modules{userModule, postModule},
})

// Apply tenantMiddleware to all routes
app.Consumer(core.NewConsumer().Apply(tenantMiddleware).Include(core.RoutesPath{
    Path: "*", Method: core.MethodAll,
}))

// Apply locationMiddleware to all routes except /post/exclude
app.Consumer(core.NewConsumer().Apply(locationMiddleware).Exclude(core.RoutesPath{
    Path: "/post/exclude", Method: core.MethodAll,
}))
```

---

## 3. Migration: MiddlewareRef → ctx.Ref(core.Provide)

If you are using the old DI-aware middleware signature (MiddlewareRef), migrate to simple middleware and call `ctx.Ref(...)`:

Before (deprecated):
```go
// old MiddlewareRef signature
func AuthMiddleware(ref core.RefProvider, ctx core.Ctx) error {
    authSvc := ref.Ref("AuthService").(*AuthService)
    if !authSvc.Check(ctx.Req()) { ... }
    return ctx.Next()
}

// attached with UseRef / MiddlewaresRef (older API)
ctrl.UseRef(AuthMiddleware).Get("/secure", handler)
```

After (recommended):
```go
// new preferred pattern
func AuthMiddleware(ctx core.Ctx) error {
    svc := ctx.Ref(core.Provide("AuthService"))
    if svc == nil {
        return ctx.Status(http.StatusInternalServerError).JSON(core.Map{"error":"service unavailable"})
    }
    authSvc := svc.(*AuthService)
    if !authSvc.Check(ctx.Req()) {
        return ctx.Status(http.StatusForbidden).JSON(core.Map{"error":"forbidden"})
    }
    return ctx.Next()
}

// attach with the normal Use method
ctrl.Use(AuthMiddleware).Get("/secure", handler)
```

Migration checklist:
- Replace `MiddlewareRef` functions with `func(ctx core.Ctx) error`.
- Inside the new function call `ctx.Ref(core.Provide("..."))` to get the provider.
- Replace `UseRef` / `MiddlewaresRef` usage with `Use` and `Middlewares` respectively (module options `Middlewares` accepts the normal middleware functions). If your code used `UseRef` helper specifically, update call sites to `Use`.

---

## 4. Ordering & Execution

Order matters — middleware is a stack. Execution proceeds from outermost to innermost registration:

1. App/global-level middleware (if present)
2. Module-level middleware (order as provided)
3. Consumer-applied middleware (follows Consumer registration order)
4. Controller-level middleware (controller.Use order)
5. Route-level middleware
6. Route handler

Within the same scope, middleware runs in the order it was added. Each middleware must call `ctx.Next()` to hand control to the next element. If a middleware returns an error, the chain stops and the framework's exception layer formats a response (see tests like core/middleware_test.go).

Example ordering:

```go
m1 := func(ctx core.Ctx) error { fmt.Println("module-1"); return ctx.Next() }
c1 := func(ctx core.Ctx) error { fmt.Println("ctrl-1"); return ctx.Next() }
r1 := func(ctx core.Ctx) error { fmt.Println("route-1"); return ctx.Next() }

mod := core.NewModule(core.NewModuleOptions{Middlewares: []core.Middleware{m1}})
ctrl := mod.NewController("test").Use(c1)
ctrl.Get("/x", r1, handler)
```

Call order when hitting /x:
module-1 -> ctrl-1 -> route-1 -> handler

---

## 5. Error handling & panics

- Returning an `error` from middleware stops the chain. The framework converts errors into HTTP responses (use `common/exception` helpers or `ctx.Status(...).JSON(...)` for structured responses).
- Panics are recovered by the framework and become `500` responses. Tests demonstrate panic handling in core/middleware_test.go.
- Best practice: return controlled errors (or use exception helpers) instead of panicking.

---

## 6. Passing data through middleware & handlers

Use `ctx.Set` / `ctx.Get` to share request-scoped values:

```go
userMiddleware := func(ctx core.Ctx) error {
    ctx.Set("user_id", 42)
    return ctx.Next()
}

handler := func(ctx core.Ctx) error {
    uid := ctx.Get("user_id")
    return ctx.JSON(core.Map{"user": uid})
}
```

Common use: auth middleware attaches authenticated user info for handlers to consume.

---

## 7. Interop with Pipes & Parsers

- Pipes (BodyParser, QueryParser, PathParser) are also middleware-like and can be applied per-route or controller using `Pipe`.
- Use pipes to populate `ctx.Body()`, `ctx.Paths()`, `ctx.Queries()` before your handler is executed.

Example:
```go
ctrl.Pipe(core.BodyParser[MyPayload]{}).Post("/create", func(ctx core.Ctx) error {
    payload := ctx.Body().(*MyPayload)
    // ...
})
```

---

## 8. Useful built-in helpers & middleware

- File interceptors (FileInterceptor, FilesInterceptor) parse multipart uploads and set ctx keys: FILE / FILES / FIELD_FILES.
- Cookie middleware (middleware/cookie) enables cookie signing and `ctx.SignedCookie`.
- Session middleware (middleware/session) provides `ctx.Session`.

File interceptor usage:

```go
ctrl.Post("/upload", FileInterceptor(storage.UploadFileOption{ /*...*/ }), func(ctx core.Ctx) error {
    f := ctx.UploadedFile()
    // ...
})
```

---

## 9. Best practices

- Keep middleware single responsibility: auth, validation, logging, metrics, etc.
- Use `ctx.Ref(core.Provide("..."))` to access providers; avoid global variables.
- Return structured errors and rely on the global exception handler for consistent responses.
- Prefer registering heavy middleware only where needed (avoid global registration if only a few routes need it).
- Use Consumer when you need cross-cutting middleware applied selectively across modules/routes without editing each controller.

---

## 10. Practical examples

1) Simple logging middleware:

```go
logMW := func(ctx core.Ctx) error {
    fmt.Printf("[%s] %s %s\n", time.Now().Format(time.RFC3339), ctx.Req().Method, ctx.Req().URL.Path)
    return ctx.Next()
}

appModule := core.NewModule(core.NewModuleOptions{
    Middlewares: []core.Middleware{logMW},
})
```

2) Auth middleware (using ctx.Ref)

```go
authMW := func(ctx core.Ctx) error {
    svc := ctx.Ref(core.Provide("AuthService"))
    if svc == nil {
        return ctx.Status(http.StatusInternalServerError).JSON(core.Map{"error": "service unavailable"})
    }
    authSvc := svc.(*AuthService)
    if !authSvc.Check(ctx.Req()) {
        return ctx.Status(http.StatusForbidden).JSON(core.Map{"error": "forbidden"})
    }
    ctx.Set("user", authSvc.UserFromRequest(ctx.Req()))
    return ctx.Next()
}

ctrl.Use(authMW).Get("/secure", secureHandler)
```

3) Conditional middleware via Consumer:

```go
app.Consumer(
    core.NewConsumer().
        Apply(rateLimitMW).
        Include(core.RoutesPath{Path: "/api/*", Method: core.MethodAll}).
        Exclude(core.RoutesPath{Path: "/api/public/*", Method: core.MethodAll}),
)
```

---

## Reference

- Middleware signature (current): `func(core.Ctx) error`
- Deprecated: MiddlewareRef style (do not use); instead use `ctx.Ref(core.Provide(...))` inside `func(core.Ctx) error`.
- Important ctx APIs used inside middleware: `ctx.Next()`, `ctx.Set()`, `ctx.Get()`, `ctx.Ref()`, `ctx.Status()`, `ctx.JSON()`.

---

If you'd like, I can:
- produce a short migration patch listing files where `MiddlewareRef`/`UseRef` is used and provide replacements,
- add a runnable mini-example showing middleware ordering with logs and Consumer rules,
- or include a small diagram showing middleware execution order relative to Consumers, Guards, and Pipes.
