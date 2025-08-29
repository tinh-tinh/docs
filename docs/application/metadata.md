---
sidebar_position: 4
---

# Metadata

Metadata lets you attach static, route-scoped values at controller- or route-registration time and retrieve them when a request is handled. Typical uses include role lists, feature flags, descriptive route info, or any small piece of static configuration that handlers, guards or middleware need to inspect.

Metadata is set when you register controllers/routes and is available from the request context using ctx.GetMetadata(...).

---

## Key ideas

- Metadata is static and attached at registration time (when you build controllers/routes).
- The framework stores metadata per-route, so when a request reaches a handler you can read the metadata for that route from the context.
- Use metadata for small configuration values (roles, feature toggles, human-readable descriptions).
- Always call `.Registry()` after configuring controller-level Metadata/Use/Guard/Pipe so the controller and its metadata are applied to routes (see Controller docs).

---

## API (common functions)

- core.SetMetadata(key string, value any) *core.Metadata  
  Helper to create a metadata object you can attach to a controller or route.

- Controller.Metadata(meta *core.Metadata) Controller  
  Attach metadata to a controller (applies to all routes declared after .Registry()).

- Route-level metadata (controller.Metadata can also be chained before route methods to apply to that route).

- ctx.GetMetadata(key string) any  
  Retrieve metadata for the current route inside a handler/middleware/guard.

Note: the actual helper/struct names used above match the tinhtinh repository patterns — metadata values are retrieved via ctx.GetMetadata(key).

---

## Example: role metadata helper

Define a small helper that produces metadata for roles:

```go
package middleware

import "github.com/tinh-tinh/tinhtinh/v2/core"

const roleKey = "route_roles"

func Roles(roles ...string) *core.Metadata {
    return core.SetMetadata(roleKey, roles)
}
```

Attach it to a controller so all routes in that controller carry the role metadata:

```go
func NewAdminController(module core.Module) core.Controller {
    ctrl := module.NewController("admin").
        Metadata(Roles("admin")). // attach metadata at controller level
        Registry()               // <-- REQUIRED: finalize controller registration

    ctrl.Get("", ListHandler)
    ctrl.Get("{id}", GetHandler)
    return ctrl
}
```

Or attach metadata to a single route:

```go
ctrl.Metadata(Roles("admin", "manager")).Get("/special", SpecialHandler)
```

---

## Reading metadata in Guards / Middleware / Handlers

Metadata is most commonly read in guards and middleware to perform access checks.

Important: Guards now use the context-based API — do not rely on the old RefProvider parameter. Resolve providers with ctx.Ref(core.Provide(...)) if you need DI.

Example guard using metadata (current preferred signature):

```go
func RoleGuard(ctx core.Ctx) bool {
    v := ctx.GetMetadata(roleKey)
    if v == nil {
        // no roles configured → allow by default (change behavior if you prefer deny-by-default)
        return true
    }
    roles, ok := v.([]string)
    if !ok || len(roles) == 0 {
        return true
    }

    // Example check: simple query param based role check
    requested := ctx.Query("role")
    for _, r := range roles {
        if r == requested {
            return true
        }
    }
    return false
}
```

Attach the guard to the controller so it runs for all its routes:

```go
ctrl := module.NewController("test").
    Guard(RoleGuard).
    Registry()
```

You may also read metadata inside middleware or handlers:

```go
mw := func(ctx core.Ctx) error {
    if meta := ctx.GetMetadata("description"); meta != nil {
        // use metadata (type assert to expected type)
        desc := meta.(string)
        ctx.Set("route_description", desc)
    }
    return ctx.Next()
}
```

---

## Practical notes & patterns

- Keys: use package-level constants for metadata keys to avoid typos and collisions (e.g., roleKey above).
- Types: metadata values are stored as interface{} — always check for nil and type-assert safely.
- Defaults: choose a convention for routes with no metadata (allow vs deny). The guard example above defaults to allow; adapt to your security requirements.
- Combining metadata: controller-level metadata applies to all routes defined after you call `.Registry()`. If you attach route-level metadata later it will override/augment as intended depending on how you read/merge values.
- DI in guards/middleware: resolve providers inside the context using ctx.Ref(core.Provide("...")) rather than older ref-based signatures.

---

## Testing tips

- Unit-test guards/middleware by creating a Ctx and setting up the expected metadata (tests often use SetCtx or helper factories — see core/ctx_test.go).
- When writing integration tests with httptest servers, register controllers exactly as in app code (including .Registry()) so metadata is attached correctly to routes.

---

## Best practices

- Keep metadata compact and static (small slices, booleans, strings).
- Avoid storing sensitive info in metadata (it is configuration, not a secure storage).
- Prefer explicit metadata merging rules if you need to combine controller-level and route-level metadata (e.g., controller roles + route extra roles).
- Use metadata for lightweight, route-scoped configuration — use session/state or providers for per-request dynamic data.

---

If you want, I can:
- scan the tinhtinh repo for all uses of Metadata/SetMetadata and produce a list of controllers/routes using metadata,
- add a short migration snippet for converting old Guard/RefProvider usages to context-based guards that use ctx.GetMetadata and ctx.Ref(core.Provide(...)),
- or produce a runnable example project showing controller-level and route-level metadata with guards and tests.
