---
sidebar_position: 1
---

# Ctx (Request Context)

The Ctx object is the per-request context used throughout tinhtinh. It wraps the native `*http.Request` and the response writer and exposes a set of convenience helpers for common web tasks: reading and parsing incoming data (path/query/body/headers/cookies), working with sessions and signed cookies, file uploads, rendering, streaming files, interacting with module providers, and writing structured responses.

This document is based on the implementation and tests under `core/ctx.go` and `core/ctx_test.go` and includes practical examples driven from those tests.

---

## High-level overview

- Ctx is passed to every controller handler: `func(ctx core.Ctx) error`.
- Handlers return `error`. The framework's exception middleware formats and writes the final error response.
- Use parser helpers (`BodyParser`, `QueryParser`, `PathParser`) to populate typed structs — safer and clearer than manual parsing.
- Middlewares can store/retrieve values on the ctx via `Set/Get` and can call `Next()` to continue the chain.

---

## Important methods (selected)

Note: method names below match the core API. See "Reference" section for full signature excerpt.

- Request / Response
  - Req() *http.Request
  - Res() *SafeResponseWriter

- Headers & Cookies
  - Headers(key string) string
  - Cookies(key string) *http.Cookie
  - SetCookie(key, value string, maxAge int)
  - SignedCookie(key string, val ...string) (string, error) — requires cookie middleware with signing key

- Param & Body parsing
  - Path(key string) string
  - PathInt/PathFloat/PathBool
  - Query(key string) string
  - QueryInt/QueryFloat/QueryBool
  - BodyParser(payload interface{}) error
  - QueryParser(payload interface{}) error
  - PathParser(payload interface{}) error
  - Body(), Paths(), Queries() — return parsed structs (used when using Pipe/Parser middlewares)

- Files (uploads)
  - UploadedFile() *storage.File
  - UploadedFiles() []*storage.File
  - UploadedFieldFile() map[string][]*storage.File
  - Useful file middleware constants: FILE, FILES, FIELD_FILES (set by interceptors)

- Responses & helpers
  - Status(code int) Ctx
  - JSON(data any) error
  - XML(data any) error
  - Render(name string, bind Map, layouts ...string) error
  - Redirect(uri string) error
  - ExportCSV(name string, body [][]string) error
  - StreamableFile(filePath string, opts ...StreamableFileOptions) error

- Flow, DI & misc
  - Next() error
  - SetCtx(w http.ResponseWriter, r *http.Request)
  - SetHandler(h http.Handler)
  - Ref(name Provide) interface{} — resolve module/provider exports
  - Session(key string, val ...interface{}) interface{}
  - Get/Set, GetMetadata(key string)
  - Scan(val any) error — decode current response body into val (handy for tests)

---

## Examples

Examples below are taken from/resemble tests in `core/ctx_test.go`. Use them as patterns for typical handlers.

1) Read request and set response header
```go
func(ctx core.Ctx) error {
    host := ctx.Req().Host
    ctx.Res().Header().Set("X-Test", "true")
    return ctx.JSON(core.Map{"host": host})
}
```

2) Parsing path, query and body values
```go
// Path params: /api/test/{id}
func(ctx core.Ctx) error {
    idStr := ctx.Path("id")
    idInt := ctx.PathInt("id", 0)
    return ctx.JSON(core.Map{"id": idStr, "idInt": idInt})
}

// Query params: /api/test?name=foo&age=20
func(ctx core.Ctx) error {
    name := ctx.Query("name")
    age := ctx.QueryInt("age", 0)
    return ctx.JSON(core.Map{"name": name, "age": age})
}

// Body parsing (JSON)
type Payload struct {
    Name string `json:"name"`
}
func(ctx core.Ctx) error {
    var p Payload
    if err := ctx.BodyParser(&p); err != nil {
        return ctx.Status(400).JSON(core.Map{"error": err.Error()})
    }
    return ctx.JSON(core.Map{"name": p.Name})
}
```

3) Struct-based parsers (recommended)
```go
type Q struct {
    Name string `query:"name"`
    Age  uint   `query:"age"`
}
func(ctx core.Ctx) error {
    var q Q
    if err := ctx.QueryParser(&q); err != nil {
        return err
    }
    return ctx.JSON(core.Map{"name": q.Name, "age": q.Age})
}
```

4) Sessions and cookies
```go
// session setter/getter
ctx.Session("user_id", 42)
id := ctx.Session("user_id")

// normal cookie
ctx.SetCookie("token", "abc123", 3600)
c := ctx.Cookies("token")

// signed cookie (requires cookie middleware use(cookie.Handler(...)))
_, err := ctx.SignedCookie("sid", "42")
val, err := ctx.SignedCookie("sid")
```

5) Uploads (via built-in interceptors)
```go
// Use provided file interceptors (FileInterceptor / FilesInterceptor) in route or controller pipe.
// In handler:
file := ctx.UploadedFile()
files := ctx.UploadedFiles()
fieldMap := ctx.UploadedFieldFile()
// The middleware sets ctx key FILE / FILES / FIELD_FILES.
```

6) Template rendering
```go
return ctx.Render("layout.html", core.Map{"Name": "John"}, "layout.html", "home.html")
```

7) Stream file / download
```go
// Sends `output.csv`. opts.Download=true sets Content-Disposition: attachment
return ctx.StreamableFile("output.csv", core.StreamableFileOptions{
    Download: true,
    FilePath: "output.csv", // used as suggested filename
})
```

8) CSV export helper
```go
rows := [][]string{
    {"Name","Age"},
    {"Alice","30"},
}
return ctx.ExportCSV("users.csv", rows)
```

9) Redirect
```go
return ctx.Redirect("/login")
```

10) Using module providers (DI) from middleware/handler
```go
svc := ctx.Ref(core.Provide("my_service"))
if svc == nil { return fmt.Errorf("service not found") }
ctx.Set("svc", svc)
```

---

## Middleware patterns

- Middleware can mutate Ctx state and call `ctx.Next()` to proceed.
- Use `ctx.Set(key, val)` / `ctx.Get(key)` to pass values through the chain.
- Example: middleware that sets a value for handlers:
```go
mw := func(ctx core.Ctx) error {
    ctx.Set("license", "ABC")
    return ctx.Next()
}
```

- The cookie signing helpers (`SignedCookie`) require the cookie middleware to be registered with an appropriate signing key (see tests that call `app.Use(cookie.Handler(cookie.Options{ Key: "..." }))`).

- File upload interceptors (provided in `core/file.go`) parse incoming multipart content and store uploaded metadata into ctx (FILE, FILES, FIELD_FILES) using configured storage middleware.

---

## Error handling

- Handler functions return `error`. When a handler returns an error (or a middleware panics), the framework exception middleware converts it to a standardized HTTP response (see `common/exception` helpers used in tests).
- Many of the parsing helpers will return an error that should be propagated (e.g., invalid conversion in QueryParser/PathParser).

---

## Testing tips

- Tests use httptest servers and exercise Ctx features (parsers, cookies, sessions, file streaming, render).
- `Scan(val any)` can be used in test helpers to decode the last response into a struct for assertions.
- `SetCtx(w, r)` is useful in unit-style tests for creating a Ctx around custom request/response objects.

---

## Reference (full interface excerpt)

```go
type Ctx interface {
    Req() *http.Request
    Res() *SafeResponseWriter
    Headers(key string) string
    Cookies(key string) *http.Cookie
    SetCookie(key string, value string, maxAge int)
    SignedCookie(key string, val ...string) (string, error)
    BodyParser(payload interface{}) error
    QueryParser(payload interface{}) error
    PathParser(payload interface{}) error
    Body() interface{}
    Paths() interface{}
    Queries() interface{}
    Path(key string) string
    PathInt(key string, defaultVal ...int) int
    PathFloat(key string, defaultVal ...float64) float64
    PathBool(key string, defaultVal ...bool) bool
    Query(key string) string
    QueryInt(key string, defaultVal ...int) int
    QueryFloat(key string, defaultVal ...float64) float64
    QueryBool(key string, defaultVal ...bool) bool
    SetCallHandler(call CallHandler)
    JSON(data any) error
    Get(key interface{}) interface{}
    Set(key interface{}, val interface{})
    Next() error
    Session(key string, val ...interface{}) interface{}
    SetCtx(w http.ResponseWriter, r *http.Request)
    SetHandler(h http.Handler)
    UploadedFile() *storage.File
    UploadedFiles() []*storage.File
    UploadedFieldFile() map[string][]*storage.File
    Redirect(uri string) error
    Ref(name Provide) interface{}
    GetMetadata(key string) interface{}
    ExportCSV(name string, body [][]string) error
    Status(statusCode int) Ctx
    XML(data any) error
    Render(name string, bind Map, layouts ...string) error
    StreamableFile(filePath string, opts ...StreamableFileOptions) error
    Scan(val any) error
}
```

---

If you'd like, I can:
- add a "Quickstart" minimal controller example,
- produce a short reference table enumerating each method (params, returns, behavior),
- or add concrete examples showing integration with cookie/session/file middlewares (with code snippets of app setup).
