---
sidebar_position: 7
---

# Performance

Use another library to improve the app's performance.

By default, Tinh Tinh uses Go's `encoding/json` library to decode and encode data for transport. However, you can use another library to improve performance, such as:

- [goccy/go-json](https://github.com/goccy/go-json)
- [bytedance/sonic](https://github.com/bytedance/sonic)

And pass it when creating the app:

```go
import (
  "github.com/tinh-tinh/tinhtinh/v2/core"
  "github.com/goccy/go-json"
)

app := core.CreateFactory(module, core.AppOptions{
  Encoder: json.Marshal,
  Decoder: json.Unmarshal,
})
```