---
sidebar_position: 8
---

# Interceptor

An **interceptor** in Tinh Tinh is a middleware mechanism for **transforming or shaping response data** after your controller logic runs—but before the response is sent to the client. Interceptors allow you to enforce global response formats, clean up or mask fields, or benchmark response handling.

![interceptor](./img/interceptor.png)

## Using Generic Interceptor Helpers

Tinh Tinh provides generic helpers like `core.MapInterceptor[T]`, allowing you to write type-safe, reusable, and composable response interceptors.

### Example: Response Interceptor with `core.MapInterceptor`

Define a transformation function for your response DTO:

```go
type UserResponse struct {
    Name  string `json:"name"`
    Email string `json:"email"`
    Age   int    `json:"age,omitempty"`
}

// Remove empty fields and print timing
func CleanAndLog(ctx core.Ctx) core.CallHandler {
    start := time.Now()
    return func(data *UserResponse) *UserResponse {
        if data.Age == 0 {
            data.Age = -1 // Or omit from JSON with omitempty
        }
        fmt.Printf("Interceptor elapsed: %dns\n", time.Since(start).Nanoseconds())
        return data
    }
}
```

Apply the interceptor in your controller using generics:

```go
ctrl.Interceptor(core.MapInterceptor[UserResponse]{Handler: CleanAndLog}).Get("/user", func(ctx core.Ctx) error {
    // Some business logic
    return ctx.JSON(&UserResponse{
        Name:  "Alice",
        Email: "alice@example.com",
        // Age: 0 will be cleaned/modified by the interceptor
    })
})
```

- `core.MapInterceptor[UserResponse]{Handler: CleanAndLog}` is a generic struct instance that acts as an interceptor.
- The transformed response is sent to the client.

## Other Interceptor Patterns

You may combine interceptors for various needs:
- Mask sensitive data
- Benchmark or log responses
- Standardize or reshape API output

## Summary

- **Generic interceptor helpers** (`core.MapInterceptor[T]{}`) provide type safety and concise syntax for handling and transforming response data.
- Use interceptors to enforce API contracts, mask or transform output, and collect response metrics.
- This approach is recommended for consistent and maintainable response shaping in Tinh Tinh.

---

For more on advanced response shaping and serialization, see [Serialization](../application/serialization.md).
