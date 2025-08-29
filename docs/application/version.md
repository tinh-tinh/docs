---
sidebar_position: 5
---

# Versioning

Tinh Tinh supports multi-version APIs on the same controller path. You can enable one of four versioning strategies and register controllers tied to specific version values. This guide explains how to enable versioning, how the framework extracts the version for each incoming request, and practical examples for each supported type.

Supported versioning types (core.VersionOptions.Type):
- core.URIVersion
- core.HeaderVersion
- core.MediaTypeVersion
- core.CustomVersion

---

## Enable versioning (app-wide)

Enable and configure versioning when you create your app factory. Typical flow:

```go
app := core.CreateFactory(AppModule)
app.SetGlobalPrefix("/api") // optional global API prefix
app.EnableVersioning(core.VersionOptions{
    Type: core.URIVersion, // choose one of the supported types
    // other fields depend on the chosen Type (see below)
})
```

Important:
- Versioning is an app-level feature. Choose and configure a single versioning strategy for your app with EnableVersioning.
- Call SetGlobalPrefix prior to relying on URI-based patterns so the final route paths are what you expect.

---

## Registering versioned controllers

Attach a version string to a controller with Controller.Version(version string). After configuration, call `.Registry()` so the controller (and its version metadata) are applied to subsequent route registrations.

Example: register two versions of the same logical controller path:

```go
// v1 controller
ctrlV1 := module.NewController("test").
    Version("1").
    Registry()

ctrlV1.Get("/", func(ctx core.Ctx) error {
    return ctx.JSON(core.Map{"data": "v1"})
})

// v2 controller
ctrlV2 := module.NewController("test").
    Version("2").
    Registry()

ctrlV2.Get("/", func(ctx core.Ctx) error {
    return ctx.JSON(core.Map{"data": "v2"})
})
```

Now the same logical resource ("test") has two separate handlers, one per version.

Notes:
- Always call `.Registry()` after configuring the controller (Use, Guard, Pipe, Version, Metadata, etc.). Registry finalizes the controller and ensures the version setting is attached to subsequent route definitions.
- You may also register an unversioned controller (no Version call) to act as a default/fallback handler.

---

## How version matching works (conceptual)

- For every incoming request the framework extracts a "version" value using the configured VersionOptions.
- Route matching then considers controllers registered with a matching version string. Only controllers with the extracted version are eligible matches for the request path.
- If you have registered controllers for multiple versions, ensure they are configured with the same base path (module.NewController("...")) and different .Version(...) values.
- If no controller matches the extracted version, the request will not match a versioned route. To handle requests with no version or unknown versions, register an unversioned controller or a controller with the version value you expect to treat as default.

(Implementation note: version extraction happens prior to the final route handler selection — ensure your registered controller versions align with the extractor output.)

---

## URIVersion (recommended when version is part of the URL)

When using URI versioning the version is expected as a path segment (commonly directly after the global API prefix). Enable like:

```go
app.EnableVersioning(core.VersionOptions{
    Type: core.URIVersion,
})
```

Example routes (with app.SetGlobalPrefix("/api")):
- GET /api/v1/test -> matches controller Version("1")
- GET /api/v2/test -> matches controller Version("2")

Typical URL style:
- /api/v1/resource
- /api/v2/resource

Notes:
- The exact placement of the version segment is governed by the framework's routing conventions: when you register a controller with Version("1") it will be resolved under the corresponding path segment (for example `/v1`).
- Use URIVersion when you want explicit, visible version segments for clients.

Example curl:
curl http://localhost:8080/api/v1/test

---

## HeaderVersion (version extracted from a custom header)

When you prefer not to change the URL, extract the version from an HTTP header:

```go
app.EnableVersioning(core.VersionOptions{
    Type:   core.HeaderVersion,
    Header: "X-Version", // header used to convey version
})
```

Example request:
GET /api/test
Header: X-Version: 2

Behavior:
- The framework will use the header value (e.g. "2") as the version key and match controllers registered with .Version("2").

Example curl:
curl -H "X-Version: 2" http://localhost:8080/api/test

---

## MediaTypeVersion (version encoded in the Accept/Content-Type header)

Media type versioning extracts a version value from the request media type (typically Accept). Configure with a key string the extractor looks for:

```go
app.EnableVersioning(core.VersionOptions{
    Type: core.MediaTypeVersion,
    Key:  "v=", // the key or token used when scanning media type parameters
})
```

Common Accept header examples:
- Accept: application/json;v=2
- Accept: application/vnd.myapp+json;v=2

Notes:
- The configured Key ("v=" above) is used to locate the version parameter in the media type header and extract the value following the key.
- Media type versioning is useful for content negotiation-based versioning or when multiple representations are served from the same URL.

Example curl:
curl -H "Accept: application/json;v=2" http://localhost:8080/api/test

---

## CustomVersion (full control via extractor function)

When none of the built-in strategies suit your needs you can provide a custom extractor function. The extractor receives the raw *http.Request and must return the version string (or empty string if none):

```go
app.EnableVersioning(core.VersionOptions{
    Type: core.CustomVersion,
    Extractor: func(r *http.Request) string {
        // Custom logic: query param, cookie, composite header, etc.
        // Example: use query parameter ?version=
        return r.URL.Query().Get("version")
    },
})
```

Example request:
GET /api/test?version=2

Notes:
- The extractor should return exactly the string used when registering controllers via .Version(...) (e.g., "2" in the example).
- Custom extractor gives complete flexibility (cookies, combined headers, derived logic, etc.).

---

## Examples: complete app (URI & Header variants)

URI versioning example:

```go
func main() {
    app := core.CreateFactory(AppModule)
    app.SetGlobalPrefix("/api")
    app.EnableVersioning(core.VersionOptions{ Type: core.URIVersion })

    // register controllers as shown earlier
    // start server...
}
```

Header versioning example:

```go
app := core.CreateFactory(AppModule)
app.SetGlobalPrefix("/api")
app.EnableVersioning(core.VersionOptions{
    Type:   core.HeaderVersion,
    Header: "X-Version",
})
```

---

## Practical considerations & best practices

- Choose one versioning strategy per app. Mixing strategies can cause client confusion unless you intentionally implement a combined extractor in CustomVersion.
- Register an unversioned or "default" controller if you want to serve requests that lack a version header/segment (or to support clients that don't send version values).
- Use semantic version strings (e.g., "1", "v1", "2024-01") consistently between your controller.Version(...) calls and your extractor output. Match exactly.
- Remember to call `.Registry()` after controller configuration so the controller's version metadata is applied to subsequent routes.
- Tests: when testing versioned controllers, assert both the path matching (for URIVersion) and the header/media-type extraction where applicable.
- Document the chosen strategy for API consumers (which header, which URL format, or which Accept format).

---

## Troubleshooting

- Routes not picking up controller-level middleware/guards/pipes? Ensure `.Registry()` was called before route definitions.
- Requests returning 404 for versioned routes? Verify:
  - The configured VersionOptions.Type matches your client behavior.
  - The extractor (header, media type key, or custom function) returns the exact string passed to .Version().
  - You registered the controller with the expected version value.
- Want to support both legacy and new clients? Consider providing a default unversioned controller or using CustomVersion to map legacy indicators to new version strings.

---

If you'd like, I can:
- produce runnable sample apps for each versioning type (URI / Header / MediaType / Custom) using the tinhtinh repo patterns,
- scan your codebase for controllers that could be converted to versioned controllers and propose a migration plan,
- or add a short curl-based test matrix you can run to validate version extraction behavior.
