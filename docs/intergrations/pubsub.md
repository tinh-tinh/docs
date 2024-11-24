---
title: 🔔 Pubsub
sidebar_position: 7
---

# Pubsub

Package support create and listen event in scope app.

## Install

```bash
go get -u github.com/tinh-tinh/pubsub
```

## Usage 

Import `ForRoot` as a global module in app:

```go
package app

import (
  "social-network/app/user"

  "github.com/tinh-tinh/mongoose"
  "github.com/tinh-tinh/pubsub"
  "github.com/tinh-tinh/tinhtinh/core"
)

func NewModule() *core.DynamicModule {
  appModule := core.NewModule(core.NewModuleOptions{
    Imports: []core.Module{
      mongoose.ForRoot("mongodb://localhost:27017", "db"),
      pubsub.ForRoot(),
      user.NewModule,
    },
    Controllers: []core.Controller{NewController},
    Providers:   []core.Provider{NewService},
  })

  return appModule
}
```

Register channel want to subscribe:

```go
package user

import (
  "github.com/tinh-tinh/mongoose"
  "github.com/tinh-tinh/pubsub"
  "github.com/tinh-tinh/tinhtinh/core"
)

func NewModule(module *core.DynamicModule) *core.DynamicModule {
  userModule := module.New(core.NewModuleOptions{
    Imports: []core.Module{
      mongoose.ForFeature(
        mongoose.NewModel[User]("users"),
      ),
      pubsub.ForFeature("USER"),
    },
    Controllers: []core.Controller{NewController},
    Providers:   []core.Provider{NewService},
  })

  return userModule
}
```

In above example, we have register subscribe with channel `"USER"`, and they way to get data from this channel:

```go
package user

import (
  "github.com/tinh-tinh/pubsub"
  "github.com/tinh-tinh/tinhtinh/core"
)

const USER_SERVICE core.Provide = "USER_SERVICE"

type userService struct {
	Data interface{}
}

func NewService(module *core.DynamicModule) *core.DynamicProvider {
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

  "github.com/tinh-tinh/pubsub"
  "github.com/tinh-tinh/swagger"
  "github.com/tinh-tinh/tinhtinh/core"
)

func NewController(module *core.DynamicModule) *core.DynamicController {
  ctrl := module.NewController("user").Metadata(swagger.ApiTag("User")).Registry()

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