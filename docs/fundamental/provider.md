---
sidebar_position: 4
---

# Provider 

![provider](./img/provider.avif)

Provider is a instance have declare in module. Two method to define a provider:

- Define with value 
- Define with function

```go
// Define with value
package user

import "github.com/tinh-tinh/tinhtinh/core"

const USER_SERVICE core.Provide = "user_service"

type UserService struct {
  Name string
}

func Service(module *core.DynamicModule) *core.DynamicProvider {
  provider := module.NewProvider(core.ProviderOptions{
    Name: USER_SERVICE,
    Value: &UserService{Name: "haha"},
  })
  
  return provider
}
```

Define provider in the module:

```go
package user

import "github.com/tinh-tinh/tinhtinh/core"

func Module(module *core.DynamicModule) *core.DynamicModule {
  userModule := module.New(core.NewModuleOptions{
    Providers: []core.Provider{Service},
  })
  
  return userModule
}
```

If you need create provider base on the other provider, you can create provider with a factory.

```go
// Define with functions
package user

import "github.com/tinh-tinh/tinhtinh/core"

const AUTH_SERVICE core.Provide = "auth_service"

type AuthService struct {
  Name string
}
func AuthService(module *core.DynamicModule) *core.DynamicProvider {
  provider := module.NewProvider(core.ProviderOptions{
    Name: USER_SERVICE,
    Factory: func(param ...interface{}) interface{} {
      userService := param[0].(*UserService)
      return &AuthService{
        Name: userService.Name,
      }
    },
    Inject: []core.Provide{USER_SERVICE},
  })
  
  return provider
}
```