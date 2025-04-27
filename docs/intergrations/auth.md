---
title: 🔒 Security
sidebar_position: 9
---

# Auth

Package support authentication, authorization and rate limiter.

## Install

```bash
go get -u github.com/tinh-tinh/auth/v2
```

## Authenticate with JWT

Package `auth` support authenticate with jwt module.

```go
appModule := core.NewModule(core.NewModuleOptions{
  Imports: []core.Modules{
    auth.Register(auth.JwtOptions{
      Alg:        jwt.SigningMethodRS256,
      PrivateKey: "LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlCUEFJQkFBSkJBTzVIKytVM0xrWC91SlRvRHhWN01CUURXSTdGU0l0VXNjbGFFKzlaUUg5Q2VpOGIxcUVmCnJxR0hSVDVWUis4c3UxVWtCUVpZTER3MnN3RTVWbjg5c0ZVQ0F3RUFBUUpCQUw4ZjRBMUlDSWEvQ2ZmdWR3TGMKNzRCdCtwOXg0TEZaZXMwdHdtV3Vha3hub3NaV0w4eVpSTUJpRmI4a25VL0hwb3piTnNxMmN1ZU9wKzVWdGRXNApiTlVDSVFENm9JdWxqcHdrZTFGY1VPaldnaXRQSjNnbFBma3NHVFBhdFYwYnJJVVI5d0loQVBOanJ1enB4ckhsCkUxRmJxeGtUNFZ5bWhCOU1HazU0Wk1jWnVjSmZOcjBUQWlFQWhML3UxOVZPdlVBWVd6Wjc3Y3JxMTdWSFBTcXoKUlhsZjd2TnJpdEg1ZGdjQ0lRRHR5QmFPdUxuNDlIOFIvZ2ZEZ1V1cjg3YWl5UHZ1YStxeEpXMzQrb0tFNXdJZwpQbG1KYXZsbW9jUG4rTkVRdGhLcTZuZFVYRGpXTTlTbktQQTVlUDZSUEs0PQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQ==",
      PublicKey:  "LS0tLS1CRUdJTiBQVUJMSUMgS0VZLS0tLS0KTUZ3d0RRWUpLb1pJaHZjTkFRRUJCUUFEU3dBd1NBSkJBTzVIKytVM0xrWC91SlRvRHhWN01CUURXSTdGU0l0VQpzY2xhRSs5WlFIOUNlaThiMXFFZnJxR0hSVDVWUis4c3UxVWtCUVpZTER3MnN3RTVWbjg5c0ZVQ0F3RUFBUT09Ci0tLS0tRU5EIFBVQkxJQyBLRVktLS0tLQ==",
      Exp:        time.Hour,
    }),
  },
})
```

Use in service:

```go
func Controller(module core.Module) core.Controller {
  ctrl := module.NewController("auth")
  jwtService := auth.InjectJwt(module)

  ctrl.Post("", func(ctx core.Ctx) error {
    data, err := jwtService.Generate(jwt.MapClaims{
      "id": 1,
    })

    if err != nil {
      return err
    }
    return ctx.JSON(core.Map{
      "data": data,
    })
  })

  ctrl.Guard(auth.Guard).Get("", func(ctx core.Ctx) error {
    return ctx.JSON(core.Map{
      "data": "ok",
    })
  })

  return ctrl
}
```

## Authorization with metadata

Package `auth` support metadata `Roles` to authorization with jwt token.

```go
func Controller(module core.Module) core.Controller {
  ctrl := module.NewController("auth")
  jwtService := auth.InjectJwt(module)

  ctrl.Post("", func(ctx core.Ctx) error {
    data, err := jwtService.Generate(jwt.MapClaims{
      "id": 1,
      "roles": []string{"admin", "user"},
    })

    if err != nil {
      return err
    }
    return ctx.JSON(core.Map{
      "data": data,
    })
  })

  ctrl.Metadata(auth.Roles("admin")).Guard(auth.Guard, auth.RoleGuard).Get("", func(ctx core.Ctx) error {
    return ctx.JSON(core.Map{
      "data": "ok",
    })
  })

  return ctrl
}
```

## Rate Limiter

Use sub-package `throttler` to add rate limiter for app.

```go
appModule := core.NewModule(core.NewModuleOptions{
  Imports: []core.Modules{
    throttler.ForRoot(&throttler.Config{Limit:100, Ttl: 5 * time.Second}),
    authModule,
  },
})
```

After register, you need use `Guard` to enable rate limiter with your route you expected.

```go
ctrl.Guard(throttler.Guard).Get("", func(ctx core.Ctx) error {
  return ctx.JSON(core.Map{
    "data": "ok",
  })
})
```
