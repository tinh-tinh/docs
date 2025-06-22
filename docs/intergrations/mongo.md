---
title: 🍃 MongoDB
sidebar_position: 4
---

# MongoDB ODM Module for Tinh Tinh

This module provides a type-safe, dependency-injected, and idiomatic way to use MongoDB with the Tinh Tinh framework. It wraps the official Mongo Go driver and adds model hooks, modular connection, multi-tenancy, and schema-like features.

---

## Features

- **Strong Typing:** Type-safe models and queries with Go generics.
- **Hooks:** Pre/post hooks for queries and mutations.
- **Modular Design:** Register MongoDB connections and models as Tinh Tinh modules/providers.
- **Multi-Tenancy:** Built-in support for tenant-specific connections.
- **Base Schemas:** Includes `BaseSchema` (with ObjectID, CreatedAt, UpdatedAt) for quick model setup.
- **Connection Factory:** Register Mongo connections with config or factory.
- **Model Utilities:** Create, update, find, and delete documents via model methods.

---

## Installation

```bash
go get -u github.com/tinh-tinh/mongoose/v2
```

---

## Basic Usage

### 1. Define Your Model

```go
import "github.com/tinh-tinh/mongoose/v2"

type Book struct {
    mongoose.BaseSchema `bson:",inline"`
    Title  string `bson:"title"`
    Author string `bson:"author"`
}
```

### 2. Register the Mongo Module

```go
import "github.com/tinh-tinh/tinhtinh/v2/core"

mongoModule := mongoose.ForRoot(mongoose.Config{
    Uri: "mongodb://localhost:27017",
    // other options...
})
```
Or use a factory:

```go
mongoose.ForRootFactory(func(module core.Module) *mongoose.Connect {
    return mongoose.New(mongoose.Config{Uri: "mongodb://localhost:27017"})
})
```

### 3. Register Your Model

```go
bookModel := mongoose.NewModel[Book]("Book")
modelsModule := mongoose.ForFeature(bookModel)
```

### 4. Inject and Use Models

```go
service := mongoose.InjectModel[Book](module)
book, err := service.Create(&Book{Title: "1984", Author: "George Orwell"})
found, err := service.FindOne(bson.M{"title": "1984"})
```

---

## CRUD Example

```go
// Create
book, err := service.Create(&Book{Title: "Dune", Author: "Frank Herbert"})

// Read one
found, err := service.FindOne(bson.M{"title": "Dune"})

// Read many
books, err := service.Find(bson.M{})

// Update
err := service.UpdateByID(book.ID, bson.M{"author": "F. Herbert"})

// Delete
err := service.DeleteByID(book.ID)
```

---

## Multi-Tenancy Example

For multi-tenant (per-request DB) setups:

```go
import "github.com/tinh-tinh/mongoose/v2/tenancy"

tenancy.ForRoot(tenancy.Options{
    Uri: "mongodb://localhost:27017",
    GetTenantID: func(r *http.Request) string {
        return r.Header.Get("X-Tenant-ID")
    },
})
```

Register your models using `tenancy.ForFeature(...)`. Inject models per request context.

---

## Model Hooks

You can register hooks for validation, save, update, delete, etc.:

```go
bookModel.Pre(mongoose.Save, func(params ...any) error {
    // validation or mutation logic
    return nil
})
```

---

## Helper Utilities

- `mongoose.ToDoc(v interface{}) (*bson.D, error)` - Convert Go struct to BSON doc.
- `mongoose.IsValidateObjectID(str string) bool` - Validate if a string is a valid ObjectID.
- `mongoose.ToObjectID(str string) primitive.ObjectID` - Parse string to ObjectID.

---

## Controller Example

```go
bookController := func(module core.Module) core.Controller {
    ctrl := module.NewController("books")

    ctrl.Post("", func(ctx core.Ctx) error {
        service := mongoose.InjectModel[Book](module)
        data, err := service.Create(&Book{
            Title:  "The Catcher in the Rye",
            Author: "J. D. Salinger",
        })
        if err != nil {
            return core.InternalServerException(ctx.Res(), err.Error())
        }
        return ctx.JSON(core.Map{"data": data})
    })

    ctrl.Get("", func(ctx core.Ctx) error {
        service := mongoose.InjectModel[Book](module)
        books, err := service.Find(bson.M{})
        if err != nil {
            return core.InternalServerException(ctx.Res(), err.Error())
        }
        return ctx.JSON(core.Map{"data": books})
    })

    // Add update/delete endpoints similarly
    return ctrl
}
```

---

For more advanced usage, including hooks and multi-tenancy, see the [mongoose source code](https://github.com/tinh-tinh/mongoose).
