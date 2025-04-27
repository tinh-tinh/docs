---
title: 🍃 MongoDB
sidebar_position: 4
---

# Mongo

Package support interactive with MongoDB like a ODM.

## Install

```bash
go get -u github.com/tinh-tinh/mongoose/v2
```

## Usage

Import mongoose module in app module:

```go
package app

import (
  "social-network/app/user"

  "github.com/tinh-tinh/mongoose/v2"
  "github.com/tinh-tinh/tinhtinh/v2/core"
)

func NewModule() core.Module {
  appModule := core.NewModule(core.NewModuleOptions{
    Imports: []core.Modules{
      mongoose.ForRoot("mongodb://localhost:27017", "db"),
      user.NewModule,
    },
    Controllers: []core.Controllers{NewController},
    Providers:   []core.Providers{NewService},
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
  "github.com/tinh-tinh/mongoose/v2"
  "github.com/tinh-tinh/tinhtinh/v2/core"
)

func NewModule(module core.Module) core.Module {
  userModule := module.New(core.NewModuleOptions{
    Imports: []core.Modules{
      mongoose.ForFeature(
        mongoose.NewModel[User]("users"),
      ),
    },
    Controllers: []core.Controllers{NewController},
    Providers:   []core.Providers{NewService},
  })

  return userModule
}
```


