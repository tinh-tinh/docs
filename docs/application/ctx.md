---
sidebar_position: 1
---

# Ctx Handler

Wrapped Handler in Tinh Tinh

Ctx Handler is wrapped function based on http.Handler of `net/http`. This support for multi function than standard library. Example ctx handler:

```go
func (ctx core.Ctx) error {
  // handler in it.
}
```

Some common function in ctx:

## Req

`Req()` is use to get `*http.Request` of `net/http` from ctx.

```go
ctrl.Get("", func (ctx core.Ctx) error {
  host := ctx.Req().Host
  // handler
})
```

## Res

`Res()` is use to get `http.ResponseWriter` of `net/http` from ctx.

```go
ctrl.Get("", func (ctx core.Ctx) error {
  ctx.Res().Header().Set("key", "value")
  // handler
})
```

## Headers

`Headers(key string)` is use to get key of header in ctx.

```go
ctrl.Get("", func (ctx core.Ctx) error {
  token := ctx.Headers("x-token")
  // handler
})
```

In this case need save value in each request, use it

## Set & Get

```go
const key core.CtxKey = "key"

ctrl.Use(func (ctx core.Ctx) error {
  ctx.Set(key, "value")
  return ctx.Next()
}).Post("", func (ctx core.Ctx) error {
  return ctx.JSON(core.Map{
    "data": ctx.Get(key),
  })
})
```

Method `Set(key string, val interface{})` will save value with key into request context of `net/http` and method `Get(key string)` will get value from key in context request.