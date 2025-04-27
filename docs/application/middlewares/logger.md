---
sidebar_position: 2
---

# Logging

Package `logger` middleware to trace log and save it into file.

## Usage

```go
import "github.com/tinh-tinh/tinhtinh/v2/middleware/logger"

// Something
app.Use(logger.Handler(logger.MiddlewareOptions{
  SeparateBaseStatus: true,
  Format:             logger.Dev,
  Rotate:             true,
}))
```

Some param in logger handler options:
- Path: folder save data log.
- Rotate: Create new file when current file log boundary.
- Max: Size each file log can be (MB).
- Format: The format you expected with file log.
- Level: Type level you expected
- SeperateBaseStatus: Will seperate file log level base on status code of http:
  + 200: info
  + 300: warning
  + 400: error
  + 500: fatal