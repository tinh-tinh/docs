---
sidebar_position: 3
---

# Cookie

A cookie is a small piece of data a server sends to a user's web browser. 

Function support cookie in Ctx Handler.


## Cookies

`Cookies(key string)` is use to get cookie by key from ctx.

```go
ctrl.Get("", func (ctx core.Ctx) error {
  cookie := ctx.Cookie("key")
})
```

## Set Cookies

`SetCookie(key string, value string, maxAge int)` use to set a cookie with value and max age in to ctx.

```go
ctrl.Get("", func (ctx core.Ctx) error {
  ctx.SetCookie("key", "val", 3600)
})
```

## SignedCookie

`SignedCookie(key string, val ...string)` use to set cookie with encrypt data. 

Defined secret key in app

```go
import (
  "github.com/tinh-tinh/tinhtinh/v2/core"
  "github.com/tinh-tinh/tinhtinh/v2/middleware/cookie"
)

func main() {
  // Something
  app.Use(cookie.Handler(cookie.Options{
    Key: "abc&1*~#^2^#s0^=)^^7%b34",
  }))
}
```

Use in ctx:

```go
ctrl.Post("", func(ctx core.Ctx) error {
  _, err := ctx.SignedCookie("key", "val")
  if err != nil {
    return err
  }

  return ctx.JSON(core.Map{
    "data": "ok",
  })
})

ctrl.Get("", func(ctx core.Ctx) error {
  data, err := ctx.SignedCookie("key")
  if err != nil {
    return err
  }
  return ctx.JSON(core.Map{
    "data": data,
  })
})
```

