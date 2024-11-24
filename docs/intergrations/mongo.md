---
title: 🍃 MongoDB
sidebar_position: 4
---

# Mongo

Package support interactive with MongoDB like a ODM.

## Install

```bash
go get -u github.com/tinh-tinh/mongoose
```

## Usage

Import mongoose module in app module:

```go
package app

import (
  "social-network/app/user"

  "github.com/tinh-tinh/mongoose"
  "github.com/tinh-tinh/tinhtinh/core"
)

func NewModule() *core.DynamicModule {
  appModule := core.NewModule(core.NewModuleOptions{
    Imports: []core.Module{
      mongoose.ForRoot("mongodb://localhost:27017", "db"),
      user.NewModule,
    },
    Controllers: []core.Controller{NewController},
    Providers:   []core.Provider{NewService},
  })

  return appModule
}
```

## Declare model in Mongoose

Define model with `BaseSchema` data store with `_id`, `createdAt`, `updatedAt`.

```go
type User struct {
	mongoose.BaseSchema `bson:"inline"`
	Name                string `bson:"name"`
	Age                 int    `bson:"age"`
}
```

And add it in `ForFeature` to use common function of `mongoose` package.

```go
package user

import (
  "github.com/tinh-tinh/mongoose"
  "github.com/tinh-tinh/tinhtinh/core"
)

func NewModule(module *core.DynamicModule) *core.DynamicModule {
  userModule := module.New(core.NewModuleOptions{
    Imports: []core.Module{
      mongoose.ForFeature(
        mongoose.NewModel[User]("users"),
      ),
    },
    Controllers: []core.Controller{NewController},
    Providers:   []core.Provider{NewService},
  })

  return userModule
}
```


