---
sidebar_position: 5
---

# Middleware

A function for handle between request comming to controller.

![middleware](./img/middleware.avif)

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

func Controller(module *core.DynamicModule) *core.DynamicController {
  ctrl := module.NewController("test")
  
  ctrl.Use(Middleware).Get("", func(ctx core.Ctx) error {
    return ctx.JSON(Map{
      "data": "ok",
    })
  })
    
  return ctrl
}
```

Use middleware for all route in controller:

```go
package app

func Controller(module *core.DynamicModule) *core.DynamicController {
  ctrl := module.NewController("test").Use(Middleware).Registry()
    
  ctrl.Get("", func(ctx core.Ctx) error {
    return ctx.JSON(Map{
	    "data": "ok",
	  })
  })
    
  return ctrl
}
```

Use middleware for module:

```go
package app

func Module(module *core.DynamicModule) *core.DynamicModule {
  mod := module.New(core.NewModuleOptions{
    Middlewares: []core.Middleware{Middleware}
  })
    
  return mod
}
```