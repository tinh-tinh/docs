---
sidebar_position: 7
---

# Custom Handler

To support custom function handler with data you expected.

`CreateWrapper` is function for you custom your data you expected. Example:

```go
tenant := core.CreateWrapper(func(data bool, ctx core.Ctx) string {
  if data {
    return "master"
  }
  return ctx.Req().Header.Get("x-tenant-id")
})

// something
// use in controller
ctrl.Get("", tenant.Handler(true, func(wCtx core.WrappedCtx[string]) error {
  return wCtx.JSON(core.Map{
    "data": wCtx.Data,
  })
}))

ctrl.Get("tenant", tenant.Handler(false, func(wCtx core.WrappedCtx[string]) error {
  return wCtx.JSON(core.Map{
    "data": wCtx.Data,
  })
}))
```

A `CreateWrapper[I, V]` will receive any data types, so you can change with your data types expected.
