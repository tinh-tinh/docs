---
sidebar_position: 7
---

# Guard

Guards in Tinh Tinh help you control access to routes by defining logic that determines whether a request should be allowed to proceed.

![guard](./img/guard.png)

---

## Summary of Features

- Allow custom guard: You can create your own guard functions to check conditions before handler execution.
- Allow registering guard for a single route: Attach guards directly to individual controller routes.
- Allow registering guard for multiple routes in a controller: Apply a guard to all routes of a controller at once.
- Allow registering guard for all controllers in a module: Apply a guard globally within a module via configuration.
- Support guards with dependency injection: Guards have access to providers/services via the RefProvider.

---

## 1. Define Guard

A guard is a function that receives a service provider and the request context, and returns `(bool, error)`:

```go
func MyGuard(ref core.RefProvider, ctx core.Ctx) (bool, error) {
    // your logic, e.g. check user from ref
    user := ref.Ref("UserService")
    // evaluate auth or role
    return true, nil // allow
}
```

---

## 2. Add Guard to a Controller Route

Attach to one route only:

```go
ctrl := module.NewController("user")
ctrl.Guard(MyGuard).Get("/private", func(ctx core.Ctx) error {
    return ctx.JSON(core.Map{"msg": "allowed"})
})
```

---

## 3. Add Guard for Multiple Routes in Controller

Apply to all routes in the controller:

```go
ctrl := module.NewController("admin").
    Guard(MyGuard).
    Registry()

ctrl.Get("/dashboard", handlerA)
ctrl.Get("/config", handlerB)
```

---

## 4. Guard for Module

Apply to all controllers in a module:

```go
mod := module.New(core.NewModuleOptions{
    Guards: []core.Guard{MyGuard},
})
```

---

**For more, see the [Tinh Tinh guard tests and examples](https://github.com/tinh-tinh/tinhtinh/search?q=guard).**

```