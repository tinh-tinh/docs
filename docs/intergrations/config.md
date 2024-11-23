---
title: ⚙️ Config 
sidebar_position: 1
---

# Config

Package support config environment with file yaml or env.

## Install

```bash
go get -u github.com/tinh-tinh/config
```

## Usage

Declare type struct with key `mapstructure`:

```go
type Config struct {
	NodeEnv   string        `mapstructure:"NODE_ENV"`
	Port      int           `mapstructure:"PORT"`
	ExpiresIn time.Duration `mapstructure:"EXPIRES_IN"`
	Secret    string        `mapstructure:"SECRET"`
}
```

And add it when import `ConfigModule` (support file **.env**, **.yaml**):

```go
package app

import (
  "github.com/tinh-tinh/config"
  "github.com/tinh-tinh/tinhtinh/core"
)

func Module() *core.DynamicModule {
  appModule := core.NewModule(core.NewModuleOptions{
    Imports: []core.Module{
      config.ForRoot[Config](".env"),
    },
  })

  return appModule
}
```

And use it in service:

```go
package app

import (
  "github.com/tinh-tinh/config"
  "github.com/tinh-tinh/tinhtinh/core"
)

type AppName struct {
  Name string
}

func Service(module *core.DynamicModule) *core.DynamicProvider {
 cfg := config.Inject[Config](module)
 svc := module.NewProvider(core.ProviderOptions{
  Name: "svc",
  Value: &AppName{Name: cfg.NodeEnv}
 })

  return svc
}
```

Function `Inject` in package `config` will get data from env was config in `ForRoot` and auto assert type with struct pass in `Inject`.

## Custom value env with function

You can use `Load` option to config value.

```go
package app

type Database struct {
  Host string
  Port string
}

type Config struct {
  Node string 
  Database Database
}

func Load() *Config {
  return &Config{
    NodeEnv: os.Getenv("NODE_ENV"),
    Database: Database{
      Host: os.Getenv("DB_HOST"),
      Port: os.Getenv("DB_PORT"),
    },
  }
}

func Module() *core.DynamicModule {
  appModule := core.NewModule(core.NewModuleOptions{
    Imports: []core.Module{
			config.ForRoot[Config](config.Options[Config]{
				EnvPath: ".env",
				Load:    Load,
			}),
    },
  })

  return appModule
}
```

## Specific Namespace For Config

You can specific config value with namespace for each module, look like you have same config:

```go
type Config struct {
  DBHost string
  DBPort string
  DBUser string
  DBPass string
  DBName string
}
```

And you can use it in two module. Example in MysqlModule:


```go
func MySQLModule(module *core.DynamicModule) *core.DynamicModule {
  mysql := module.New(core.NewModuleOptions{
    Imports: []core.Module{
      config.ForFeature[Config]("mysql", func() *Config {
        return &Config{
          DBHost: os.Getenv("MYSQL_DBHOST"),
          DBPort: os.Getenv("MYSQL_DBPORT"),
          DBUser: os.Getenv("MYSQL_DBUSER"),
          DBPass: os.Getenv("MYSQL_DBPASS"),
          DBName: os.Getenv("MYSQL_DBNAME"),
        }
      }),
    },
  })

  return mysql
}
```

And in `MongoModule`:

```go
func MongoModule(module *core.DynamicModule) *core.DynamicModule {
  mongo := module.New(core.NewModuleOptions{
    Imports: []core.Module{
      config.ForFeature("mongo", func() *Config {
        return &Config{
          DBHost: os.Getenv("MONGO_DBHOST"),
          DBPort: os.Getenv("MONGO_DBPORT"),
          DBUser: os.Getenv("MONGO_DBUSER"),
          DBPass: os.Getenv("MONGO_DBPASS"),
          DBName: os.Getenv("MONGO_DBNAME"),
        }
      }),
    },
  })

  return mongo
}
```

So when use `Config`, you can specific namespace to get value like:

```go
func(module *core.DynamicModule) *core.DynamicProvider {
	cfg := config.InjectNamespace[Config](module, "mongo")
  // Something
}
```

## Condition Module

When some case, we need init module base on some key in env.

```go
package app

import (
  "github.com/joho/godotenv"
  "github.com/tinh-tinh/config"
  "github.com/tinh-tinh/tinhtinh/core"
)

func Module() *core.DynamicModule {
  godotenv.Load(".env.example")
  appModule := core.NewModule(core.NewModuleOptions{
    Imports: []core.Module{
      config.RegisterWhen(userModule, "NODE_ENV"),
    },
  })

  return appModule
}
```

With this config, module `userModule` only init when `NODE_ENV` have value in file `.env`.

In other case, you need use function to custom condition you can pass function like this:

```go
package app

import (
  "os"

  "github.com/joho/godotenv"
  "github.com/tinh-tinh/config"
  "github.com/tinh-tinh/tinhtinh/core"
)

func Module() *core.DynamicModule {
  godotenv.Load(".env.example")
  appModule := core.NewModule(core.NewModuleOptions{
    Imports: []core.Module{
      config.RegisterWhen(userModule, func() bool {
				return os.Getenv("NODE_ENV") == "development"
			}),
    },
  })

  return appModule
}
```