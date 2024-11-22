---
sidebar_position: 2
---

# Serialization

Serilization is a process transform data from request to Go struct data types.

The process serialize among use [Pipe](../fundamental/pipe.md) or use raw function in Ctx.

## Use Pipe

If using Pipe in controller we can use some built-in function in ctx to get data.

| Functions | Description |
|-|-|
| ctx.Body() | Get data from requet body and parsing with `Body()` in Pipe |
| ctx.Params() | Get data from requet param and parsing with `Param()` in Pipe |
| ctx.Queries() | Get data from requet query and parsing with `Query()` in Pipe |

## Use functions 

### BodyParser

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

### QueryParser

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

### ParamParser

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

