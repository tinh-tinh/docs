---
title: 🔔 Pubsub
sidebar_position: 7
---

# PubSub for Tinh Tinh

PubSub is a modular, in-memory publish/subscribe (pubsub) system for the Tinh Tinh framework. It enables decoupled communication between different parts of your application using topics and message subscribers.

---

## Features

- **Central Broker:** Manages topics and subscribers.
- **Dynamic Subscribers:** Subscribe to one or many topics dynamically.
- **Asynchronous Messaging:** Message delivery is non-blocking.
- **Topic Patterns:** Supports wildcards and topic delimiters for pattern-based subscriptions.
- **Broadcast Support:** Broadcast a message to all subscribers.
- **Integration with Tinh Tinh Modules:** Use dependency injection for broker and subscribers.
- **Subscriber Limits:** Optionally limit the max number of subscribers.
- **Handler Utility:** Simplifies listening to topics with concise functions and is the recommended way to consume messages.

---

## Installation

```bash
go get -u github.com/tinh-tinh/pubsub/v2
```

---

## Basic Usage

### 1. Register the Broker

```go
import "github.com/tinh-tinh/pubsub/v2"

pubsubModule := pubsub.ForRoot(pubsub.BrokerOptions{
    // Optional: MaxSubscribers, Wildcard, Delimiter, etc.
})
```

### 2. Register Subscribers (Feature Modules)

Subscribe to specific topics:

```go
priceSubModule := pubsub.ForFeature("BTC", "ETH") // subscribe to multiple topics
```

### 3. Use Handler for Consumption (Recommended)

**Instead of using `subscriber.GetMessages()` or `Listener`, use the Handler utility for subscribing to topics and consuming messages.**

```go
import (
    "github.com/tinh-tinh/pubsub/v2"
    "github.com/tinh-tinh/tinhtinh/v2/core"
)

type PriceService struct {
    Message interface{}
}

priceService := func(module core.Module) core.Provider {
    return module.NewProvider(core.ProviderOptions{
        Name:  "PriceService",
        Value: &PriceService{},
    })
}

priceHandler := func(module core.Module) core.Provider {
    handler := pubsub.NewHandler(module)
    priceService := module.Ref("PriceService").(*PriceService)

    handler.Listen(func(msg *pubsub.Message) {
        priceService.Message = msg.GetContent()
    }, "BTC", "ETH", "SOL")

    return handler
}
```

### 4. Use in Controllers

```go
controller := func(module core.Module) core.Controller {
    ctrl := module.NewController("prices")

    ctrl.Post("", func(ctx core.Ctx) error {
        broker := pubsub.InjectBroker(module)
        go broker.Publish("BTC", "hihi")
        return ctx.JSON(core.Map{"data": "ok"})
    })

    ctrl.Get("", func(ctx core.Ctx) error {
        service := module.Ref("PriceService").(*PriceService)
        return ctx.JSON(core.Map{"data": service.Message})
    })

    return ctrl
}
```

---

## Advanced Features

### Broker Options

- `MaxSubscribers`: Limit the maximum number of subscribers.
- `Wildcard` and `Delimiter`: Enable pattern matching for topics (e.g., `"orders.*"` with `"."` as delimiter).

### Handler

- The Handler manages subscription and message handling in a background goroutine.
- You can listen to multiple topics in one handler.

---

## Examples

**Handler Usage:**

```go
handler := pubsub.NewHandler(module)
handler.Listen(func(msg *pubsub.Message) {
    fmt.Println("Received:", msg.GetContent())
}, "BTC", "ETH")
```

---

For more advanced patterns and full API, see the [pubsub source code](https://github.com/tinh-tinh/pubsub).