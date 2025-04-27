---
sidebar_position: 7
---

# Guard

A middleware responsible to manage access to controller.

![guard](./img/guard.png)

Guard is a middleware only use for manage access in route. It is a function return bool value if true the route can be handler, else app will throw 403 error. Have two guard for each level. `Guard` for Controller and `AppGuard` for Module.

## Define guard and use in controller

```go
package app

import "github.com/tinh-tinh/tinhtinh/v2/core"

func QueryGuard(ref core.RefProvider, ctx core.Ctx) bool {
  return ctx.Query("key") == "value"
}

func Controller(module core.Module) core.Controller {
  ctrl := module.NewController("test")
  
  ctrl.Guard(QueryGuard).Get("", func (ctx core.Ctx) error {
    return ctx.JSON(core.Map{
      "data": "ok",
    })
  })
  
  return ctrl
}
```

## Define guard and use in module

```go
package app

import "github.com/tinh-tinh/tinhtinh/v2/core"

func QueryGuard(ref RefProvider, ctx Ctx) bool {
  return ctx.Query("key") == "value"
}

func Controller(module core.Module) core.Module {
  mod := module.New(core.NewModuleOptions{
    Guards: []core.Guard{QueryGuard},
  })
  
  return mod
}
```