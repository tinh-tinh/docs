---
sidebar_position: 3
---

# Session

The session based on in-memory cache to store information across multi requests.

Enable session in create App

```go
import (
  "github.com/tinh-tinh/tinhtinh/v2/core"
  "github.com/tinh-tinh/tinhtinh/v2/middleware/session"
)

session := session.New(session.Options{
  Secret: "secret",
})

app := core.CreateFactory(module, core.AppOptions{
  Session: session,
})
```

Use in controller:

```go
ctrl.Post("", func(ctx core.Ctx) error {
  ctx.Session("key", "val")

  return ctx.JSON(core.Map{
    "data": "ok",
  })
})

ctrl.Get("", func(ctx core.Ctx) error {
  data := ctx.Session("key")
  return ctx.JSON(core.Map{
    "data": data,
  })
})
```