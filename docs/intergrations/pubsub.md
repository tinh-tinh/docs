---
title: 🔔 Pubsub
sidebar_position: 7
---

# Pubsub

Package support create and listen event in scope app.

## Install

```bash
go get -u github.com/tinh-tinh/pubsub/v2
```

## Usage 

Import `ForRoot` as a global module in app:

```go
package app

import (
  "social-network/app/user"

  "github.com/tinh-tinh/mongoose/v2"
  "github.com/tinh-tinh/pubsub/v2"
  "github.com/tinh-tinh/tinhtinh/v2/core"
)

func NewModule() core.Module {
  appModule := core.NewModule(core.NewModuleOptions{
    Imports: []core.Modules{
      mongoose.ForRoot("mongodb://localhost:27017", "db"),
      pubsub.ForRoot(),
      user.NewModule,
    },
    Controllers: []core.Controllers{NewController},
    Providers:   []core.Providers{NewService},
  })

  return appModule
}
```

Register channel want to subscribe:

```go
package user

import (
  "github.com/tinh-tinh/mongoose/v2"
  "github.com/tinh-tinh/pubsub/v2"
  "github.com/tinh-tinh/tinhtinh/v2/core"
)

func NewModule(module core.Module) core.Module {
  userModule := module.New(core.NewModuleOptions{
    Imports: []core.Modules{
      mongoose.ForFeature(
        mongoose.NewModel[User]("users"),
      ),
      pubsub.ForFeature("USER"),
    },
    Controllers: []core.Controllers{NewController},
    Providers:   []core.Providers{NewService},
  })

  return userModule
}
```

In above example, we have register subscribe with channel `"USER"`, and they way to get data from this channel:

```go
package user

import (
  "github.com/tinh-tinh/pubsub/v2"
  "github.com/tinh-tinh/tinhtinh/v2/core"
)

const USER_SERVICE core.Provide = "USER_SERVICE"

type userService struct {
	Data interface{}
}

func NewService(module core.Module) core.Provider {
  svc := module.NewProvider(core.ProviderOptions{
    Name: USER_SERVICE,
    Factory: pubsub.Listener(module, func(s *pubsub.Subscriber) interface{} {
      userSvc := &userService{}
      go (func() {
        msg, ok := <-s.GetMessages()
        if ok {
          userSvc.Data = msg.GetContent()
        }
      })()

      return userSvc
    }),
  })

  return svc
}
```

And that is how to send data to it channel

```go
package user

import (
  "social-network/app/user/dto"

  "github.com/tinh-tinh/pubsub/v2"
  "github.com/tinh-tinh/tinhtinh/v2/core"
)

func NewController(module core.Module) core.Controller {
  ctrl := module.NewController("user").Registry()

  svc := module.Ref(USER_SERVICE).(*userService)
  ctrl.Pipe(core.Body(&dto.CreateUser{})).Post("/", func(ctx core.Ctx) error {
    broker := pubsub.InjectBroker(module)
    go broker.Publish("USER", "data")
    svc.Create(ctx.Body())
    return ctx.JSON(core.Map{"data": "ok"})
  })

  return ctrl
}
```