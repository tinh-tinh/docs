---
sidebar_position: 7
---

# Guard

A middleware responsible for managing access to the controller.

![guard](./img/guard.png)

A guard is a middleware used only to manage access in routes. It is a function that returns a boolean value: if true, the route can be handled; otherwise, the app will throw a 403 error. 

## Define a guard

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
- Module
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