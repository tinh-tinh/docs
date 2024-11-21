---
sidebar_position: 3
---

# Controller

![controller](./img/controller.avif)

A controller's purpose is to receive specific requests for the application. The **routing** mechanism controls which controller receives which requests. Frequently, each controller has more than one route, and different routes can perform different actions.

Syntax for create controller in Tinh Tinh:

```go
package app

import "github.com/tinh-tinh/tinhtinh/core"

func Controller(module *core.DynamicModule) *core.DynamicController {
  ctrl := module.NewController("users")
  
  ctrl.Get("", func (ctx core.Ctx) {
    ctx.JSON(core.Map{
        "data": "ok"
    })
  })
  
  return ctrl
}
```

Method abailable in controller:

| **Method** | **Description** |
|-|-|
| Use(...[Middleware](./middleware.md)) | function for common middleware |
| Guard(...[Guard](./guard.md)) | function for check forbidden as middleware |
| Version(string) | function for mark version of controllers |
| Registry() | function registry a group middleware to use for all route path in this controller |
| Pipe(...[Pipe](./pipe.md)) | function for validate and transform dto |
| Get(string, Handler) | register handler use GET HTTP |
| Post(string, Handler) | register handler use POST HTTP |
| Patch(string, Handler) | register handler use PATCH HTTP |
| Put(string, Handler) | register handler use PUT HTTP |
| Delete(string, Handler) | register handler use DELETE HTTP |
| Handler(string, http.Handler) | register a pure http.Handler with path |
| Inject([Provide](./provider.md)) | Get provider in the same module with controller |
