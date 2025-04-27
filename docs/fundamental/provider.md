---
sidebar_position: 4
---

# Provider 

A unit to store value use across module.

![provider](./img/provider.png)

Provider is a instance have declare in module. Two method to define a provider:

- Define with value 
- Define with function

```go
// Define with value
package user

import "github.com/tinh-tinh/tinhtinh/v2/core"

const USER_SERVICE core.Provide = "user_service"

type UserService struct {
  Name string
}

func NewService(module core.Module) core.Provider {
	svc := module.NewProvider(core.ProviderOptions{
		Name: USER_SERVICE,
		Value: &UserService{},
	})

	return svc
}
```

Define provider in the module:

```go
package user

import "github.com/tinh-tinh/tinhtinh/v2/core"

func NewModule(module core.Module) core.Module {
	userModule := module.New(core.NewModuleOptions{
		Providers:   []core.Providers{NewService},
	})

	return userModule
}
	
```

If you need create provider base on the other provider, you can create provider with a factory.

```go
// Define with functions
package user

import "github.com/tinh-tinh/tinhtinh/v2/core"

const AUTH_SERVICE core.Provide = "auth_service"

type AuthService struct {
  Name string
}
func AuthService(module core.Module) core.Provider {
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