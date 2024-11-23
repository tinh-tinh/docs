---
title: ⛓ SQL Gorm
sidebar_position: 3
---

# SQL

Package support interactive with SQL Database through Gorm

## Install 

```bash
go get -u github.com/tinh-tinh/sqlorm
```

## Usage

Connect database with gorm, example with postgres:

```go
package app

import (
  "github.com/tinh-tinh/sqlorm"
	"github.com/tinh-tinh/tinhtinh/core"
	"gorm.io/driver/postgres"
)

func Module() *core.DynamicModule {
  dsn := "host=localhost user=postgres password=postgres dbname=test port=5432 sslmode=disable TimeZone=Asia/Shanghai"

  appModule := core.NewModule(core.NewModuleOptions{
    Imports: []core.Module{
      sqlorm.ForRoot(sqlorm.Options{
        Dialect: postgres.Open(dsn),
      }),
    },
  })

  return appModule
}
```

Define model base on `gorm` package:

```go
type User struct {
  sqlorm.Model `gorm:"embedded"`
  Name         string `gorm:"type:varchar(255);not null"`
  Email        string `gorm:"type:varchar(255);not null"`
}
```

After init add this model to `ForRoot` function to migrate data.

```go
appModule := core.NewModule(core.NewModuleOptions{
  Imports: []core.Module{
    sqlorm.ForRoot(sqlorm.Options{
      Dialect: postgres.Open(dsn),
      Models:  []interface{}{&User{}},
    }),
  },
})
```

Package `sqlorm` support `Repository` to use model with some utils function like `Create`, `BatchCreate`, `FindOne`, ....

## Use with repository

Use `ForFeature` function to init repository in module.

```go
package app

import (
  "github.com/tinh-tinh/sqlorm"
	"github.com/tinh-tinh/tinhtinh/core"
)


func userModule(module *core.DynamicModule) *core.DynamicModule {
  mod := module.New(core.NewModuleOptions{
    Imports:     []core.Module{
      sqlorm.ForFeature(
        sqlorm.NewRepo(User{}),
      ),
    },
  })

  return mod
}
```

Use repository in service like this:

```go
package app

import (
  "github.com/tinh-tinh/sqlorm"
	"github.com/tinh-tinh/tinhtinh/core"
)

type UserService struct {
  Model *sqlorm.Repository[User]
}

func UserService(module *core.DynamicModule) *core.DynamicProvider {
  repo := sqlorm.InjectRepository[User](module)

  svc := module.NewProvider(core.ProviderOptions{
    Name: "User",
    Value: &UserService{
      Model: repo,
    },
  })

  return svc
}
```