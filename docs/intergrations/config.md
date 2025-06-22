---
title: ⚙️ Config 
sidebar_position: 1
---

# Config Module for Tinh Tinh

The Config module for the Tinh Tinh framework provides flexible configuration management for Go applications. It supports loading environment variables from `.env` files and struct-based configs from YAML, with first-class integration for dependency injection and modular apps.

---

## Features

- Load configuration from `.env`, `.yaml`, or `.yml` files
- Strongly-typed config structs with tags for mapping, default values, and validation
- Namespace-based config injection for multiple configs (e.g., database, cache)
- Conditional module registration based on environment or custom logic
- Supports default values and validation via struct tags
- Seamless integration with Tinh Tinh modules and controllers

---

## Installation

```bash
go get -u github.com/tinh-tinh/config/v2
```

---

## Quick Start

### 1. Basic Usage with `.env`

```go
import "github.com/tinh-tinh/config/v2"

type AppConfig struct {
    NodeEnv   string        `mapstructure:"NODE_ENV"`
    Port      int           `mapstructure:"PORT"`
    ExpiresIn time.Duration `mapstructure:"EXPIRES_IN"`
    Log       bool          `mapstructure:"LOG"`
    Secret    string        `mapstructure:"SECRET"`
}

cfg, err := config.NewEnv[AppConfig](".env")
if err != nil {
    panic(err)
}
fmt.Println(cfg.Port)
```

### 2. Using with YAML

```go
type YamlConfig struct {
    Host string `yaml:"host"`
    Port int    `yaml:"port"`
}

cfg, err := config.NewYaml[YamlConfig]("config.yaml")
if err != nil {
    panic(err)
}
fmt.Println(cfg.Host)
```

---

## Tinh Tinh Module Integration

### Register as a Global Config

```go
import "github.com/tinh-tinh/tinhtinh/v2/core"

appModule := core.NewModule(core.NewModuleOptions{
    Imports: []core.Modules{
        config.ForRoot[AppConfig](".env"),
    },
})
```

### Inject Config Anywhere

```go
cfg := config.Inject[AppConfig](module)
fmt.Println(cfg.Port)
```

---

## Namespaced Configs (Multiple Sources)

```go
type MysqlConfig struct {
    DBHost string `mapstructure:"MYSQL_DBHOST"`
    DBPort string `mapstructure:"MYSQL_DBPORT"`
}

type MongoConfig struct {
    DBHost string `mapstructure:"MONGO_DBHOST"`
    DBPort string `mapstructure:"MONGO_DBPORT"`
}

// Register namespaced configs
appModule := core.NewModule(core.NewModuleOptions{
    Imports: []core.Modules{
        config.ForFeature[MysqlConfig]("mysql"),
        config.ForFeature("mongo", func() *MongoConfig {
            return &MongoConfig{
                DBHost: os.Getenv("MONGO_DBHOST"),
                DBPort: os.Getenv("MONGO_DBPORT"),
            }
        }),
    },
})

// Usage in controller/service
mysqlCfg := config.InjectNamespace[MysqlConfig](module, "mysql")
mongoCfg := config.InjectNamespace[MongoConfig](module, "mongo")
```

---

## Conditional Module Registration

```go
import "os"

// Register module only if an env var is set
mod := config.RegisterWhen(config.ForRoot[AppConfig](".env"), "NODE_ENV")

// Or by custom function
mod = config.RegisterWhen(config.ForRoot[AppConfig](".env"), func() bool {
    return os.Getenv("ENABLE_FEATURE") == "true"
})
```

---

## Struct Tag Reference

- `mapstructure:"ENV_VAR"` — maps struct field to environment variable
- `default:"value"` — sets default value if env is missing
- `validate:"..."` — supports validation (e.g. `required`, `isEmail`)
- `yaml:"key"` — for YAML files

Example:

```go
type Config struct {
    NodeEnv string `mapstructure:"NODE_ENV" default:"production"`
}
```

---

## Supported File Types

- `.env` (default, uses [godotenv](https://github.com/joho/godotenv))
- `.yaml` / `.yml` (uses [gopkg.in/yaml.v3](https://pkg.go.dev/gopkg.in/yaml.v3))

---

## Advanced: Raw Loading

```go
config.ForRootRaw(".env.example") // Load .env file globally (no struct)
```

---

## Error Handling & Validation

- Returns errors when variables are missing, types are invalid, or validation fails.
- Supports default values for missing environment variables.

---

## Testing & Patterns

See [`env_test.go`](https://github.com/tinh-tinh/config/blob/main/env_test.go) and [`namespace_test.go`](https://github.com/tinh-tinh/config/blob/main/namespace_test.go) for examples of real-world usage.

---

For more, see the [source code](https://github.com/tinh-tinh/config).