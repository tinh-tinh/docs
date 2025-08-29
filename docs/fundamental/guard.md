---
sidebar_position: 7
---

# Guard

Guards in Tinh Tinh control whether a request is allowed to proceed to the handler. They are evaluated before the route handler runs and are intended to authorize/validate requests (e.g., role checks, feature flags, simple request rules).

![guard](./img/guard.png)

This document reflects the current implementation in the tinhtinh repo: guards are lightweight, request-scoped predicates that receive the request context and return a boolean. If any guard returns false the framework responds with 403 Forbidden.

---

## Summary of current behavior

- Guard signature (current): func(ctx core.Ctx) bool
- Guards run before the handler; multiple guards attached at different scopes are combined (logical AND): every guard must return true for the request to proceed.
- Guards may call ctx.Set(...) to attach request-scoped values for downstream middleware / handlers.
- To access module providers / services inside a guard, use ctx.Ref(core.Provide("...")).
- You can attach guards at route, controller and module scopes.
- When using the fluent controller builder pattern, call .Registry() last to finalize and register the controller so guards and middleware apply to routes defined afterwards.

---

## 1. Define a Guard

The current, preferred guard shape:

```go
func MyGuard(ctx core.Ctx) bool {
    // simple check (return true to allow)
    return ctx.Query("allow") == "1"
}
```

A guard can also set context values for handlers:

```go
func WithUserGuard(ctx core.Ctx) bool {
    // set derived data for handlers
    ctx.Set("from_guard", ctx.Query("token"))
    return true
}
```

To access providers/services use ctx.Ref:

```go
func AuthGuard(ctx core.Ctx) bool {
    svc := ctx.Ref(core.Provide("AuthService"))
    if svc == nil {
        // if service is missing treat as forbidden or log — return false
        return false
    }
    authSvc := svc.(*AuthService)
    return authSvc.ValidateRequest(ctx.Req())
}
```

Note: older patterns that passed a RefProvider into the guard function are deprecated. Migrate to the pattern above and call ctx.Ref(...) inside the guard.

---

## 2. Attach Guard to a single route

Attach a guard when declaring a route:

```go
ctrl := module.NewController("user")
ctrl.Guard(MyGuard).Get("/private", func(ctx core.Ctx) error {
    return ctx.JSON(core.Map{"msg": "allowed"})
})
```

If MyGuard returns false the framework responds with 403 and the handler will not run.

---

## 3. Attach Guard at controller level (apply to many routes)

You can attach a guard to a controller so it applies to all routes under that controller.

Two styles:

- Explicit builder (call Registry() last — important when using the fluent style):

```go
ctrl := module.NewController("admin").
    Guard(AdminGuard).
    Registry() // finalize/attach the controller

ctrl.Get("/dashboard", dashboardHandler) // inherits AdminGuard
ctrl.Get("/config", configHandler)       // inherits AdminGuard
```

Important: when you use the fluent builder chain (.Guard(...).Registry()) call `.Registry()` last. Registry finalizes the controller and ensures subsequent route definitions belong to it and inherit configured guards/middleware. Failure to call Registry() correctly may cause routes to not receive the controller-level guards.

- Direct style:

```go
ctrl := module.NewController("admin")
ctrl.Guard(AdminGuard)

ctrl.Get("/dashboard", dashboardHandler)
ctrl.Get("/config", configHandler)
```

Both approaches attach the guard to all controller routes; choose the builder style if you prefer fluent chaining, but remember to call `.Registry()` as the terminal call.

---

## 4. Module-level guards

Apply guards across all controllers in a module:

```go
mod := core.NewModule(core.NewModuleOptions{
    Guards: []core.Guard{ModuleWideGuard},
})
```

Module-level guards and controller-level guards are both evaluated (all must return true).

---

## 5. Guard evaluation semantics

- Multiple guards attached to the same route (from module, controller and route) are combined with logical AND: every guard must return true.
- If any guard returns false, the framework returns HTTP 403 Forbidden.
- Guards do not return errors; side effects (logging, ctx.Set) are done inside the guard; to produce a custom response you can instead place an early middleware that uses ctx.Status(...).JSON(...) and returns an error to stop the chain with a controlled response.

---

## 6. Migration notes (old → new)

If you have older guards that used a DI-aware signature (for example: func(ref core.RefProvider, ctx core.Ctx) (bool, error)), migrate them to the new form:

Before (deprecated):
```go
func OldGuard(ref core.RefProvider, ctx core.Ctx) (bool, error) {
    svc := ref.Ref("AuthService")
    // ...
    return true, nil
}
ctrl.UseRef(OldGuard) // older attachment pattern
```

After (current):
```go
func NewGuard(ctx core.Ctx) bool {
    svc := ctx.Ref(core.Provide("AuthService"))
    if svc == nil { return false }
    auth := svc.(*AuthService)
    return auth.ValidateRequest(ctx.Req())
}
ctrl.Guard(NewGuard).Get("/secure", handler)
```

Checklist:
- Change signature to func(core.Ctx) bool.
- Replace ref.Ref(...) with ctx.Ref(core.Provide("...")).
- Attach with ctrl.Guard(...) or provide via module options (Guards).

---

## 7. Example: combining module & controller guards

```go
moduleGuard := func(ctx core.Ctx) bool { return ctx.Query("module") == "value" }
controllerGuard := func(ctx core.Ctx) bool { return ctx.Query("ctrl") == "value" }

authCtrl := func(m core.Module) core.Controller {
    ctrl := m.NewController("test")
    ctrl.Guard(controllerGuard).Get("", func(ctx core.Ctx) error {
        return ctx.JSON(core.Map{"data": "1"})
    })
    ctrl.Get("module", func(ctx core.Ctx) error {
        return ctx.JSON(core.Map{"data": "2"})
    })
    return ctrl
}

module := func() core.Module {
    appModule := core.NewModule(core.NewModuleOptions{
        Controllers: []core.Controllers{authCtrl},
        Guards:      []core.Guard{moduleGuard},
    })
    return appModule
}
```

Requests must satisfy both moduleGuard and controllerGuard (if both apply) to reach the handler.

---

## 8. Best practices

- Keep guards focused on boolean checks (access allowed / denied). Use middleware for richer responses or side-effects that should stop the chain with a specific HTTP response body.
- Guards may set request-scoped values via ctx.Set(...) for handlers to read.
- Use ctx.Ref(core.Provide("...")) to obtain services/providers inside guards. Avoid globals.
- When defining controller-wide guards with the fluent API, call `.Registry()` last to ensure routes defined afterwards are registered correctly and inherit the guard.

---

## 9. Related features

- Middleware: for richer control flows and custom responses use middleware (see middleware_guide.md).
- Pipes / Parsers: run parsing pipes (BodyParser / QueryParser / PathParser) where you need typed input available to guards or handlers.
- Consumers: apply cross-cutting rules (middleware) to selected routes; guards remain the simple allow/deny checks.

---

If you want, I can:
- scan the repo for legacy MiddlewareRef/Guard signatures and prepare a migration patch (list of files and suggested replacements),
- add a short runnable example app showing module/controller guards and how ctx.Ref(...) resolves a provider inside a guard.
