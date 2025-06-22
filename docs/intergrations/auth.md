---
title: 🔒 Security
sidebar_position: 9
---

# Auth Module for Tinh Tinh

The Tinh Tinh Auth module provides flexible, extensible authentication and authorization utilities for the Tinh Tinh framework. It supports JWT (HMAC and RSA), password hashing, OAuth2, encryption, Casbin-based authorization, Two-Factor Auth (2FA), CSRF, and more. All features are designed for easy dependency injection and modular use.

---

## Features

- **JWT authentication** (HMAC & RSA): create, verify, decode tokens with custom expiration and signing algorithms.
- **OAuth2** social login support (Google, GitHub, etc.) via [goth](https://github.com/markbates/goth).
- **Password hashing & verification** using HMAC-SHA256 with random salt.
- **Symmetric encryption** for sensitive data (AES-GCM).
- **Role-based authorization** via [Casbin](https://github.com/casbin/casbin).
- **Two-Factor Authentication (2FA)** via TOTP.
- **CSRF Protection** middleware and token generator.
- Easy integration with Tinh Tinh modules and controllers.

---

## Installation

```bash
go get -u github.com/tinh-tinh/auth/v2
```

---

## JWT Usage

### Register JWT Module

```go
import "github.com/tinh-tinh/auth/v2"

appModule := core.NewModule(core.NewModuleOptions{
    Imports: []core.Modules{
        auth.Register(auth.JwtOptions{
            Alg:    jwt.SigningMethodHS256,
            Secret: "supersecret", // or use SigningMethodRS256 and provide keys
            Exp:    time.Hour * 2,
        }),
    },
})
```

### Using JWT in Controllers

```go
jwtService := auth.InjectJwt(module)
token, err := jwtService.Generate(jwt.MapClaims{"user_id": 42})
claims, err := jwtService.Verify(token)
```

#### Unit Test Patterns

```go
jwtService := auth.NewJwtHS(auth.JwtOptions{
    Alg:    jwt.SigningMethodHS256,
    Secret: "secret",
    Exp:    time.Hour,
})
token, err := jwtService.Generate(jwt.MapClaims{"foo": "bar"})
payload, err := jwtService.Verify(token)
require.Equal(t, "bar", payload["foo"])
```

---

## JWT Expiry and Error Handling

- Supports custom expiration per-token via `GenOptions`.
- Handles wrong tokens, wrong secrets, and expiration correctly (see tests).
- Example:

```go
token, err := jwtService.Generate(jwt.MapClaims{"foo": "bar"}, auth.GenOptions{Exp: 1 * time.Millisecond})
time.Sleep(10 * time.Millisecond)
_, err = jwtService.Verify(token)
require.NotNil(t, err) // Expired
```

---

## Password Hashing

```go
hash := auth.Hash("mypassword")
ok := auth.VerifyHash(hash, "mypassword")
require.True(t, ok)
```

- Supports custom salt length: `Hash("password", 4)`

---

## Symmetric Encryption

```go
crypto := auth.NewCrypto("your-32-byte-key-1234567890123456")
cipher := crypto.Encrypt("secret")
plain := crypto.Decrypt(cipher)
require.Equal(t, "secret", plain)
```

- Handles invalid keys with panics for both encrypt and decrypt.

---

## Guard Middleware Example

Use JWT guards in controllers to secure endpoints:

```go
authController := func(module core.Module) core.Controller {
    ctrl := module.NewController("test")
    jwtService := auth.InjectJwt(module)

    ctrl.Get("", func(ctx core.Ctx) error {
        token, _ := jwtService.Generate(jwt.MapClaims{"roles": []string{"admin"}})
        return ctx.JSON(core.Map{"data": token})
    })

    ctrl.Guard(auth.Guard).Post("", func(ctx core.Ctx) error {
        return ctx.JSON(core.Map{"data": "ok"})
    })
    return ctrl
}
```

- The guard checks for valid JWT in the Authorization header.

---

## Casbin Authorization

```go
import "github.com/tinh-tinh/auth/v2/authz"
enforcer := authz.Inject(module)
ok, err := enforcer.Enforce("alice", "/resource", "read")
```

- You can add custom guards that check Casbin policies in controllers or modules.

---

## Two-Factor Authentication (2FA)

```go
import "github.com/tinh-tinh/auth/v2/twofa"

totpCode := twofa.Inject(module)
data, err := totpCode.Generate(totp.GenerateOpts{
    Issuer:      "YourApp",
    AccountName: "user@example.com",
})
valid := totpCode.Validate(code, data.Secret())
```

---

## CSRF Protection

```go
import "github.com/tinh-tinh/auth/v2/csrf"

csrfToken := csrf.Inject(module)
token := csrfToken.Generate(ctx.Req())

ctrl.Guard(csrf.Guard).Post("", func(ctx core.Ctx) error {
    // Only passes if CSRF token is valid
})
```

- Configure secret and token extraction via the `Register` config.

---

## Test-Driven Examples

All patterns above are directly validated with comprehensive unit and integration tests:
- [hash_test.go](https://github.com/tinh-tinh/auth/blob/main/hash_test.go)
- [crypto_test.go](https://github.com/tinh-tinh/auth/blob/main/crypto_test.go)
- [jwt_hmac_test.go](https://github.com/tinh-tinh/auth/blob/main/jwt_hmac_test.go)
- [jwt_rsa_test.go](https://github.com/tinh-tinh/auth/blob/main/jwt_rsa_test.go)
- [middleware_test.go](https://github.com/tinh-tinh/auth/blob/main/middleware_test.go)
- [authz/authz_test.go](https://github.com/tinh-tinh/auth/blob/main/authz/authz_test.go)
- [twofa/twofa_test.go](https://github.com/tinh-tinh/auth/blob/main/twofa/twofa_test.go)
- [csrf/module_test.go](https://github.com/tinh-tinh/auth/blob/main/csrf/module_test.go)

---

## See Also

- [jwt.go](https://github.com/tinh-tinh/auth/blob/main/jwt.go), [oauth2/oauth2.go](https://github.com/tinh-tinh/auth/blob/main/oauth2/oauth2.go), [authz/authz.go](https://github.com/tinh-tinh/auth/blob/main/authz/authz.go)
- [More source code and updates](https://github.com/tinh-tinh/auth)