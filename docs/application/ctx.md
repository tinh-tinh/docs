---
sidebar_position: 1
---

# Ctx Handler

The `Ctx` object is the core context passed to every request handler in tinhtinh. It encapsulates the HTTP request and response objects, as well as helper methods for common web operations.

---

## Key Features & Examples

Below, each major feature is introduced with a real usage example derived from `core/ctx_test.go`.

---

### 1. Request/Response Access

**Access the underlying HTTP request and response writer.**

```go
func(ctx core.Ctx) error {
    // Get the request's host
    host := ctx.Req().Host

    // Set a header in the response
    ctx.Res().Header().Set("X-Test", "true")

    return ctx.JSON(core.Map{"host": host})
}
```

---

### 2. Parameter Parsing

#### Path Parameters

```go
func(ctx core.Ctx) error {
    // /api/test/{id}
    id := ctx.Path("id")
    idInt := ctx.PathInt("id")
    idFloat := ctx.PathFloat("id")
    idBool := ctx.PathBool("id")
    return ctx.JSON(core.Map{
      "id": id, 
      "idInt": idInt, 
      "idFloat": idFloat, 
      "idBool": idBool,
    })
}
```

#### Query Parameters

```go
func(ctx core.Ctx) error {
    // /api/test?name=test&age=20
    name := ctx.Query("name")
    age := ctx.QueryInt("age")
    score := ctx.QueryFloat("score", 0.0)
    active := ctx.QueryBool("active", false)
    return ctx.JSON(core.Map{"name": name, "age": age, "score": score, "active": active})
}
```

#### Body Parsing

```go
func(ctx core.Ctx) error {
    var payload struct {
        Name string `json:"name"`
    }
    err := ctx.BodyParser(&payload)
    if err != nil {
        return ctx.Status(400).JSON(core.Map{"error": err.Error()})
    }
    return ctx.JSON(core.Map{"name": payload.Name})
}
```

#### Struct-Based Parsing

```go
// PathParser
type ParamData struct {
    ID int `path:"id"`
}
func(ctx core.Ctx) error {
    var data ParamData
    _ = ctx.PathParser(&data)
    return ctx.JSON(core.Map{"id": data.ID})
}

// QueryParser
type QueryData struct {
    Name string `query:"name"`
    Age  uint   `query:"age"`
}
func(ctx core.Ctx) error {
    var data QueryData
    _ = ctx.QueryParser(&data)
    return ctx.JSON(core.Map{"name": data.Name, "age": data.Age})
}
```

---

### 3. Cookie & Session Management

#### Cookies

```go
func(ctx core.Ctx) error {
    // Set a cookie
    ctx.SetCookie("token", "abc123", 3600)
    // Get a cookie value
    cookie, _ := ctx.Cookies("token")
    return ctx.JSON(core.Map{"cookie": cookie.Value})
}
```

#### Signed Cookies

```go
func(ctx core.Ctx) error {
    _, err := ctx.SignedCookie("session", "signed-value")
    if err != nil {
        return ctx.Status(500).JSON(core.Map{"error": err.Error()})
    }
    val, _ := ctx.SignedCookie("session")
    return ctx.JSON(core.Map{"signed": val})
}
```

#### Session

```go
func(ctx core.Ctx) error {
    // Set session value
    ctx.Session("user_id", 123)
    // Get session value
    id := ctx.Session("user_id")
    return ctx.JSON(core.Map{"user_id": id})
}
```

---

### 4. Header Management

```go
func(ctx core.Ctx) error {
    // Read a request header
    agent := ctx.Headers("User-Agent")
    // Set a response header
    ctx.Res().Header().Set("X-Custom", "yes")
    return ctx.JSON(core.Map{"agent": agent})
}
```

---

### 5. File Uploads

```go
func(ctx core.Ctx) error {
    // Get the first uploaded file
    file := ctx.UploadedFile()
    // Get all files
    files := ctx.UploadedFiles()
    return ctx.JSON(core.Map{
        "filePresent": file != nil,
        "fileCount": len(files),
    })
}
```

---

### 6. Response Helpers

#### Status & JSON

```go
func(ctx core.Ctx) error {
    return ctx.Status(201).JSON(core.Map{"created": true})
}
```

#### Redirect

```go
func(ctx core.Ctx) error {
    return ctx.Redirect("/api/another-endpoint")
}
```

#### Render HTML Template

```go
func(ctx core.Ctx) error {
    return ctx.Render("layout.html", core.Map{"Name": "Alice"}, "layout.html", "home.html")
}
```

#### Stream Files

```go
func(ctx core.Ctx) error {
    return ctx.StreamableFile("output.csv", core.StreamableFileOptions{
        Download: true,
        FilePath: "output.csv",
    })
}
```
