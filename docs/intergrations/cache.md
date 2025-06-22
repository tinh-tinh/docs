---
title: ⚡Cache Manager
sidebar_position: 5
---

# Cache Manager

The Cache Manager provides a unified API to manage caching in Tinh Tinh applications. It supports memory, Memcache, and Redis backends, and can be flexibly configured and injected into your modules and controllers.

## Installation

```bash
go get -u github.com/tinh-tinh/cacher/v2
```

## Features

- **Pluggable Backends:** Supports in-memory, Memcache, and Redis stores.
- **Type Safety:** Generic interface for strong typing.
- **Namespace Support:** Isolate cache by logical namespace.
- **Compression:** Optional data compression.
- **Context-aware:** Supports context propagation for advanced use cases.
- **Hooks:** Register hooks for cache lifecycle events.

## Basic Usage

### Setting Up an In-Memory Cache

```go
import "github.com/tinh-tinh/cacher/v2"

cache := cacher.NewSchema[string](cacher.Config{
    Store: cacher.NewInMemory(cacher.StoreOptions{
        Ttl: 15 * time.Minute,
    }),
})

err := cache.Set("users", "John")
data, err := cache.Get("users")
// ...
```

### Using Namespaces

You can create isolated caches using the `Namespace` field:

```go
cache1 := cacher.NewSchema[string](cacher.Config{
    Store:     store,
    Namespace: "cache1",
})
cache2 := cacher.NewSchema[string](cacher.Config{
    Store:     store,
    Namespace: "cache2",
})
```

### Memcache Example

```go
import "github.com/tinh-tinh/cacher/storage/memcache"

cache := memcache.New(memcache.Options{
    Addr: []string{"localhost:11211"},
    Ttl:  15 * time.Minute,
})
```

### Redis Example

```go
import (
    "github.com/tinh-tinh/cacher/storage/redis"
    redis_store "github.com/redis/go-redis/v9"
)

cache := redis.New(redis.Options{
    Connect: &redis_store.Options{
        Addr: "localhost:6379",
    },
    Ttl: 15 * time.Minute,
})
```

## API Overview

The main cache interface provides:

- `Set(key, value, opts...)`: Store a value.
- `Get(key)`: Retrieve a value.
- `Delete(key)`: Remove a value.
- `Clear()`: Remove all values.
- `MSet(...params)`: Batch set.
- `MGet(...keys)`: Batch get.

## Module Integration

You can register the cache as a provider in a Tinh Tinh module and inject it into controllers:

```go
import (
    "github.com/tinh-tinh/cacher/v2"
    "github.com/tinh-tinh/tinhtinh/v2/core"
)

func userController(module core.Module) core.Controller {
    cache := cacher.Inject[[]byte](module)
    ctrl := module.NewController("users")

    ctrl.Get("", func(ctx core.Ctx) error {
        data, err := cache.Get("users")
        // handle data
    })
    // ...
    return ctrl
}
```

To register the cache provider:

```go
module := core.NewModule(core.NewModuleOptions{
    Imports: []core.Modules{
        cacher.Register(cacher.Config{ Store: cache }),
        userModule,
    },
})
```

## Advanced

- **Compression:** Set `CompressAlg` in `Config`.
- **Hooks:** Use the `Hooks` field to register cache hooks.
- **Context:** Use `SetCtx` and `GetCtx` for context operations.

## Testing

The repository includes comprehensive tests for all stores and features. See `cacher_test.go` and `storage/memcache/memcache_test.go` for examples.

---

For further details, see the [cacher source code](https://github.com/tinh-tinh/cacher).