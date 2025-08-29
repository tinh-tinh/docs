---
sidebar_position: 3
---

# Session

Sessions in tinhtinh provide a simple way to persist small pieces of request-scoped state across requests (for example, authenticated user id, flash messages, or CSRF nonce). By default the session middleware uses an in-memory cache (suitable for development and tests). For production you should use an external backing store (Redis, memcached, etc.) if you need persistence or multiple app instances.

This guide shows how to enable sessions, how to read/write values from handlers/middleware, recommended usage patterns, and security / production notes.

---

## Enable session middleware (AppOptions)

Create a session instance and pass it to the app factory through AppOptions.Session:

```go
import (
    "github.com/tinh-tinh/tinhtinh/v2/core"
    "github.com/tinh-tinh/tinhtinh/v2/middleware/session"
)

s := session.New(session.Options{
    Secret: "very-secret-value", // signing/encryption key used by the session middleware
})

app := core.CreateFactory(module, core.AppOptions{
    Session: s,
})
```

Notes:
- session.New(...) creates the session middleware instance. The snippet above shows the minimal configuration (a Secret). See the session package for additional Options (cookie settings, TTL, backing store) if you need to customize behavior.
- The session middleware registers itself in the app so session functions (ctx.Session(...)) become available inside handlers and other middleware.

---

## Basic usage in controllers

The Ctx API exposes a Session helper:

- Signature (ctx API): ctx.Session(key string, val ...interface{}) interface{}
  - If you call ctx.Session("key", value) it sets the session value for "key".
  - If you call ctx.Session("key") it returns the stored value (or nil if not set).
  - The returned value requires a type assertion in Go to your expected type.

Example — set and get session values:

```go
ctrl.Post("/login", func(ctx core.Ctx) error {
    // after authenticating the user
    userID := 42
    ctx.Session("user_id", userID) // set

    return ctx.JSON(core.Map{"ok": true})
})

ctrl.Get("/me", func(ctx core.Ctx) error {
    val := ctx.Session("user_id")
    if val == nil {
        return ctx.Status(401).JSON(core.Map{"error": "unauthenticated"})
    }
    uid := val.(int) // type assert to expected type
    return ctx.JSON(core.Map{"user_id": uid})
})
```

---

## Common patterns

1. Authentication middleware that attaches the user to context:

```go
authMW := func(ctx core.Ctx) error {
    id := ctx.Session("user_id")
    if id == nil {
        return ctx.Status(401).JSON(core.Map{"error": "unauthenticated"})
    }
    // set a handler-friendly value for the request
    ctx.Set("user_id", id.(int))
    return ctx.Next()
}
```

2. Logout (clearing session state)  
The session package may provide helpers to destroy the session or remove keys. A portable approach is to remove keys you set:

```go
ctrl.Post("/logout", func(ctx core.Ctx) error {
    // remove session keys set by your app
    ctx.Session("user_id", nil) // check your session package for a dedicated Destroy method
    return ctx.JSON(core.Map{"ok": true})
})
```

(Consult the session package in the repo for a Destroy or Clear method if you need to expire the whole session or remove the session cookie.)

---

## Backing store & scaling

- Default: in-memory store — good for development and tests only. Sessions stored in-memory do not survive process restarts and are not shared between app instances.
- Production: use a shared store (Redis, memcached) so sessions are available across multiple app instances and survive restarts. The session package typically exposes or documents pluggable stores; check middleware/session for available store adapters and example usage.

Example (conceptual) — using a Redis-backed store (API names vary by implementation):

```go
// pseudo-code: see session package for actual adapter API
redisStore := session.NewRedisStore(redisClient)
s := session.New(session.Options{
    Secret: "very-secret",
    Store:  redisStore,
    TTL:    time.Hour * 24,
})
```

---

## Security considerations

- Secret: use a strong, randomly-generated secret for signing/encrypting session cookies.
- Cookie flags: ensure the session cookie is created with Secure (HTTPS only), HttpOnly, and an appropriate SameSite value. The session.Options usually expose these settings — configure them for production.
- TTL: set an appropriate session expiration (TTL). Shorter TTLs reduce the lifetime of stolen session identifiers.
- Store: prefer Redis or another persistent, shared store for production deployments behind load balancers.

---

## Error handling & best practices

- Type assertions: ctx.Session returns interface{} — always check for nil and assert to the expected type safely (e.g., v, ok := ctx.Session("user_id").(int)).
- Keep session payloads small: store simple values (IDs, booleans, short strings). Avoid storing large objects in session.
- Avoid storing secrets or sensitive data directly in session values; instead store references (IDs) and fetch full data from a provider when needed.
- Register session middleware before other middleware/consumers that need it so session values are available early in the pipeline.

---

## Where to look in the repo

- middleware/session — session middleware implementation and Options (cookie settings, store adapters, TTL, destroy method).
- core/ctx.go / core/ctx_test.go — examples of ctx.Session usage and tests showing expected behavior.
- middleware/cookie — cookie helper middleware used by sessions for cookie parsing/signing.

---

If you want, I can:
- add a runnable example demonstrating Redis-backed sessions (I will scan the session package to use the correct store adapter API),
- scan the repo for uses of ctx.Session and produce a migration or audit report (e.g., to ensure Logout uses session.Destroy when available), or
- update other docs (authentication, middleware) to reference the session lifecycle and ordering.
