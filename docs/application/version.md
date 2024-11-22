---
sidebar_position: 6
---

# Version

Feature allows multi version on one controller.

There are 4 types of versioning that are supported:
- URI
- Header
- Media Type
- Custom

## Common use

Enable versioning in app:

```go
app := core.CreateFactory(AppModule)
app.SetGlobalPrefix("/api")
app.EnableVersioning(core.VersionOptions{
  Type: // one of four types support
})
```

Use in controller:

```go
ctrl := module.NewController("test").Version("1")

ctrl.Get("/", func(ctx core.Ctx) error {
  return ctx.JSON(core.Map{
    "data": "1",
  })
})
return ctrl
```

```go
ctrl := module.NewController("test").Version("2")

ctrl.Get("/", func(ctx core.Ctx) error {
  return ctx.JSON(core.Map{
    "data": "2",
  })
})
```

## URI Version

If you use URI versioning, you only need enable this config:

```go
app.EnableVersioning(core.VersionOptions{
	Type: core.URIVersion,
})
```

## Header Version

When you choose header versioning, you need specific header you used.

```go
app.EnableVersioning(core.VersionOptions{
  Type:   core.HeaderVersion,
  Header: "X-Version", 
})
```

## Media Type Version

If you use media-type, you need set key you are using.

```go
app.EnableVersioning(core.VersionOptions{
  Type: core.MediaTypeVersion,
  Key:  "v=",
})
```

## Custom Version

When you choose custom, you can pass a function to return a value you expected as a version.

```go
app.EnableVersioning(core.VersionOptions{
  Type: core.CustomVersion,
  Extractor: func(r *http.Request) string {
    return r.URL.Query().Get("version")
  },
})
```