---
sidebar_position: 4
---

# Metadata

Metadata is a static value have set in controller to use when request comming.

Example:

```go
func Roles(roles ...string) *core.Metadata {
	return core.SetMetadata(role_key, roles)
}
```

Use it in controller like this:

```go
ctrl.Metadata(Roles("admin")).Get("", func(ctx core.Ctx) error {
  return ctx.JSON(core.Map{
    "data": "ok",
  })
})
```

In this case we need get metadata of route path when have request comming to handler. So we use `ctx.GetMetadata()` to get it.

```go
func RoleGuard(ctrl *core.DynamicController, ctx *core.Ctx) bool {
  roles, ok := ctx.GetMetadata(role_key).([]string)
  if !ok || len(roles) == 0 {
    return true
  }
  isRole := slices.IndexFunc(roles, func(role string) bool {
    return ctx.Query("role") == role
  })
  return isRole != -1
}
```

and use it for all route in controller:

```go
ctrl := module.NewController("test").Guard(RoleGuard).Registry()
```