---
sidebar_position: 6
---

# Pipe

![pipe](./img/pipe.avif)

Pipe is function validate and transform data request before handler function. Have 3 types pipe
- **Body**: validate and transform data from request body
- **Query**: validate and transform data from request query
- **Param**: validate and transform data from request params.

## Body

```go
// Define dto

type SignUpDto struct {
  Name     string `validate:"required"`
  Email    string `validate:"required,isEmail"`
  Password string `validate:"isStrongPassword"`
  Age      int    `validate:"isInt"`
}
```

Use in controller

```go
package app

import "github.com/tinh-tinh/tinhtinh/core"

func Controller(module *core.DynamicModule) *core.DynamicController {
  ctrl := module.NewController("test")
  
  ctrl.Pipe(core.Body(&SignUpDto{})).Post("", func (ctx core.Ctx) error {
    return ctx.JSON(core.Map{
      "data": ctx.Body(),
    })
  })
  
  return ctrl
}
```

## Query

Define dto for query

```go
package app

import "github.com/tinh-tinh/tinhtinh/core"

type FilterDto struct {
  Name  string `validate:"required" query:"name"`
  Email string `validate:"required,isEmail" query:"email"`
  Age   int    `validate:"isInt" query:"age"`
}

func Controller(module *core.DynamicModule) *core.DynamicController {
  ctrl := module.NewController("test")
  
  ctrl.Pipe(core.Query(&FilterDto{})).Post("", func (ctx core.Ctx) error {
    return ctx.JSON(core.Map{
      "data": ctx.Queries(),
    })
  })
  
  return ctrl
}
```

## Param

Define dto for param

```go
package app

import "github.com/tinh-tinh/tinhtinh/core"

type ParamDto struct {
  ID int `validate:"required,isInt" param:"id"`
}

func Controller(module *core.DynamicModule) *core.DynamicController {
  ctrl := module.NewController("test")
  
  ctrl.Pipe(core.Param(&ParamDto{})).Get("{id}", func (ctx core.Ctx) error {
    return ctx.JSON(core.Map{
      "data": ctx.Params(),
    })
  })
  
  return ctrl
}
```