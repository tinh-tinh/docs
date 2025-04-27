---
sidebar_position: 7
---

# Performance

Use another lib to improve performance of app.

By default, TinhTinh use `encoding/json` lib of Go to decode and encoder data to transport. But you can use another lib to improve performance like:

- [goccy/go-json](https://github.com/goccy/go-json)
- [bytedance/sonic](https://github.com/bytedance/sonic)

And pass it when create app:

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