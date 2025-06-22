---
title: 🌐 HTTP Fetch
sidebar_position: 8
---

# Tinh Tinh Fetch

A flexible, type-safe HTTP client for the Tinh Tinh Framework, inspired by fetch/axios, supporting full-featured requests, easy configuration, and integration with Tinh Tinh modules.

---

## Features

- Simple and chainable syntax for all HTTP verbs (GET, POST, PUT, PATCH, DELETE, etc.)
- Base URL and per-request configuration
- Automatic JSON encoding/decoding (customizable)
- Query parameter and header support
- Timeout and cancellation (context-based)
- Cookie and credential management
- Response formatting with type inference
- Full integration as a Tinh Tinh module/provider

---

## Installation

```bash
go get -u github.com/tinh-tinh/fetch/v2
```

---

## Quick Usage

### Create a Fetch Instance

```go
import "github.com/tinh-tinh/fetch/v2"

client := fetch.Create(&fetch.Config{
    BaseUrl: "https://jsonplaceholder.typicode.com",
    Headers: map[string][]string{
        "x-api-key": {"abcd", "efgh"},
    },
    Timeout: 5 * time.Second,
    ResponseType: "json",
})
```

### Make HTTP Requests

```go
// GET with query params
type Query struct {
    PostID int `query:"postId"`
}
res := client.Get("comments", &Query{PostID: 1})

// POST with body and query
type Post struct {
    UserID int    `json:"userId"`
    Title  string `json:"title"`
    Body   string `json:"body"`
}
res = client.Post("posts", &Post{UserID: 1, Title: "foo", Body: "bar"}, &Query{PostID: 1})

// PUT, PATCH, DELETE
client.Put("posts/1", &Post{UserID: 1, Title: "updated", Body: "new"}, &Query{PostID: 1})
client.Patch("posts/1", &Post{UserID: 1, Title: "patch", Body: "body"}, &Query{PostID: 1})
client.Delete("posts/1", &Query{PostID: 1})
```

### Handling Responses

```go
type Posts []Post
var data Posts
res := client.Get("posts").Format(&data)

if res.Error != nil {
    // handle error
}
fmt.Println("Status:", res.Status)
fmt.Println("Data:", data)
```

---

## Advanced: Timeout & Cancellation

```go
token := fetch.NewCancelToken()
client := fetch.Create(&fetch.Config{
    BaseUrl:     "https://httpbin.org",
    CancelToken: token.Context(),
    Timeout:     2 * time.Second,
})

go func() {
    resp := client.Get("/delay/5")
    if resp.Error != nil {
        fmt.Println("HTTP error:", resp.Error)
    }
}()

time.Sleep(1 * time.Second)
token.Cancel() // cancel the request
```

---

## Module Integration Example

```go
import (
    "github.com/tinh-tinh/fetch/v2"
    "github.com/tinh-tinh/tinhtinh/v2/core"
)

func controller(module core.Module) core.Controller {
    ctrl := module.NewController("posts")
    httpFetch := fetch.Inject(module)

    ctrl.Get("/", func(ctx core.Ctx) error {
        var data []Post
        res := httpFetch.Get("posts").Format(&data)
        return ctx.Status(res.Status).JSON(core.Map{"data": data})
    })
    return ctrl
}

appModule := core.NewModule(core.NewModuleOptions{
    Imports: []core.Modules{
        fetch.Register(&fetch.Config{
            BaseUrl: "https://jsonplaceholder.typicode.com",
        }),
    },
    Controllers: []core.Controllers{controller},
})
```

---

## Configuration Reference

```go
type Config struct {
    Url            string
    BaseUrl        string
    Headers        http.Header
    Params         map[string]interface{}
    Data           map[string]interface{}
    Timeout        time.Duration
    WithCredentials bool
    ResponseType   string
    CancelToken    context.Context
    Encoder        core.Encode // default: json.Marshal
    Decoder        core.Decode // default: json.Unmarshal
}
```

---

## Tips

- Query struct tags (`query:"..."`) are used to serialize query parameters.
- Supports custom request/response encoding.
- Handles cookies if `WithCredentials` is true.
- Use `.Format(&yourStruct)` to decode JSON response directly.

---

For more patterns, see [`fetch_test.go`](https://github.com/tinh-tinh/fetch/blob/main/fetch_test.go) and [`module_test.go`](https://github.com/tinh-tinh/fetch/blob/main/module_test.go).