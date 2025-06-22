---
sidebar_position: 5
---

# Middleware

A function for handle between request comming to controller.

![middleware](./img/middleware.png)

A middleware is a function called before handler. A syntax to create middleware:

```go
package app

func Middleware(ctx core.Ctx) error {
  // Something
  return ctx.Next()
}
```

Use middleware in controller:

```go
package app

import "github.com/tinh-tinh/tinhtinh/v2/core"

func Controller(module core.Module) core.Controller {
  ctrl := module.NewController("test")
  
  ctrl.Use(Middleware).Get("", func(ctx core.Ctx) error {
    return ctx.JSON(core.Map{
      "data": "ok",
    })
  })
    
  return ctrl
}
```

Use middleware for all route in controller:

```go
package app

import "github.com/tinh-tinh/tinhtinh/v2/core"

func Controller(module core.Module) core.Controller {
  ctrl := module.NewController("test").Use(Middleware).Registry()
    
  ctrl.Get("", func(ctx core.Ctx) error {
    return ctx.JSON(core.Map{
	    "data": "ok",
	  })
  })
    
  return ctrl
}
```

Use middleware for module:

```go
package app

import "github.com/tinh-tinh/tinhtinh/v2/core"

func Module(module core.Module) core.Module {
  mod := module.New(core.NewModuleOptions{
    Middlewares: []core.Middleware{Middleware}
  })
    
  return mod
}
```

## Use middleware with provider

If you need get some provider of module in middleware, you can use `MiddlewareRef`, by create function like this:

```go
package app

func MiddlewareRef(ref core.RefProvider, ctx core.Ctx) error {
  prd := ref.Ref(Name)   // Name of provider in module
  // Something
  return ctx.Next()
}
```

And use it:

```go
ctrl := module.NewController("test").UseRef(MiddlewareRef)
```

The middleware Ref cannot import in module. But you still use it with module like that:

```go
package app

import "github.com/tinh-tinh/tinhtinh/v2/core"

func Module(module core.Module) core.Module {
  mod := module.New(core.NewModuleOptions{
    // something
  }).UseRef(MiddlewareRef)

  return mod
}
```