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

## Cookies

`Cookies(key string)` is use to get cookie by key from ctx.

```go
ctrl.Get("", func (ctx core.Ctx) error {
  cookie := ctx.Cookie("key)
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
  "github.com/tinh-tinh/tinhtinh/core"
  "github.com/tinh-tinh/tinhtinh/middleware/cookie"
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

## BodyParser

`BodyParser(payload interface{})` is use to parse request body to struct value.

```go
type BodyData struct {
	Name string `json:"name"`
}

ctrl.Post("", func(ctx core.Ctx) error {
  var bodyData BodyData
  err := ctx.BodyParser(&bodyData)
  if err != nil {
    return err
  }
  return ctx.JSON(core.Map{
    "data": bodyData.Name,
  })
})
```

## QueryParser

`QueryParser(payload interface{})` is use to parse request query string to struct value.

```go
type QueryData struct {
  Age    int  `query:"age"`
  Format bool `query:"format"`
}

ctrl.Get("", func(ctx core.Ctx) error {
  var queryData QueryData
  err := ctx.QueryParse(&queryData)
  if err != nil {
    return err
  }
  return ctx.JSON(core.Map{
    "data": queryData,
  })
})
```

## ParamParser

`ParamParser(payload interface{})` is use to parse param query in request to struct value.

```go
type ParamData struct {
  ID     int  `param:"id"`
  Export bool `param:"export"`
}

ctrl.Get("{id}/{export}", func(ctx core.Ctx) error {
  var queryData ParamData
  err := ctx.ParamParse(&queryData)
  if err != nil {
    return err
  }
  return ctx.JSON(core.Map{
    "data": queryData.ID,
  })
})
```