---
title: ⛓ SQL Gorm
sidebar_position: 3
---

# SQL ORM Module

The SQL ORM module integrates the power of GORM with the Tinh Tinh framework, providing a modular, type-safe, and context-aware way to manage your application's SQL database access.

## Features

- **GORM Integration:** Full support for GORM's dialects, migrations, and advanced features.
- **Modular Design:** Easily register DB connections and repositories as Tinh Tinh modules/providers.
- **Type-Safe Repositories:** Generics-based repository pattern for strong typing.
- **Auto-Migration:** Optional, automatic schema migration.
- **Retry Connection:** Built-in retry logic for resilient DB connections.
- **Multi-Tenancy:** Built-in support for multi-tenant architectures.
- **Repository Query Builders:** Powerful, chainable query API.
- **Helper Utilities:** Handle Go/SQL nullable types with ease.

## Installation

```bash
go get -u github.com/tinh-tinh/sqlorm/v2
```

## Basic Usage

### Registering the SQL Module

```go
import (
    "github.com/tinh-tinh/sqlorm/v2"
    "gorm.io/driver/postgres"
)

module := sqlorm.ForRoot(sqlorm.Config{
    Dialect: postgres.Open("postgres://user:pass@localhost:5432/dbname"),
    Models: []interface{}{User{}},
    Sync: true, // auto migration
})
```

Or using a factory:

```go
sqlorm.ForRootFactory(func(module core.RefProvider) sqlorm.Config {
    return sqlorm.Config{
        Dialect: postgres.Open(...),
        Models:  []interface{}{User{}},
    }
})
```

### Injecting the Database or Repository

```go
db := sqlorm.Inject(module)
repo := sqlorm.InjectRepository[User](module)
```

## Working With Repositories

```go
userRepo := sqlorm.InjectRepository[User](module)
user, err := userRepo.Create(&User{Name: "Alice", Email: "alice@example.com"})
found, err := userRepo.FindOne(sqlorm.Query{/*...*/}, &sqlorm.FindOneOptions{})
```

### SQLORM CRUD Example

This guide demonstrates detailed CRUD (Create, Read, Update, Delete) operations using the `sqlorm` module in a Tinh Tinh application, based on the patterns in the unit tests of the [`tinh-tinh/sqlorm`](https://github.com/tinh-tinh/sqlorm) repository.

### Define Your Model

```go
import "github.com/tinh-tinh/sqlorm/v2"

type User struct {
    sqlorm.Model
    Name  string `gorm:"type:varchar(255);not null"`
    Email string `gorm:"type:varchar(255);not null"`
}
```

### Create

```go
user := &User{Name: "John", Email: "john@gmail.com"}
result, err := repo.Create(user)
// result is *User, err is error
```

### Read (Find One)

```go
// Find by ID
user, err := repo.FindByID("uuid-string")

// Find by condition (e.g., name)
user, err := repo.FindOne(map[string]interface{}{"name": "John"})
```

### Read (Find All)

```go
users, err := repo.FindAll(nil, sqlorm.FindOptions{})
// Find with query options (e.g., limit, order)
opts := sqlorm.FindOptions{Limit: 10, Order: []string{"created_at desc"}}
users, err := repo.FindAll(nil, opts)
```

### Update

```go
// Update by ID
updated, err := repo.UpdateByID(user.ID, map[string]interface{}{"email": "newemail@example.com"})

// Or update by condition
updated, err := repo.UpdateOne(map[string]interface{}{"name": "John"}, map[string]interface{}{"email": "newemail@example.com"})
```

### Delete

```go
// Delete by ID
err := repo.DeleteByID(user.ID)

// Delete by condition
err := repo.DeleteOne(map[string]interface{}{"email": "john@gmail.com"})
```

### Example Controller

```go
userController := func(module core.Module) core.Controller {
    ctrl := module.NewController("users")
    repo := sqlorm.InjectRepository[User](module)

    // Create User
    ctrl.Post("", func(ctx core.Ctx) error {
        result, err := repo.Create(&User{Name: "John", Email: "john@gmail.com"})
        if err != nil {
            return core.InternalServerException(ctx.Res(), err.Error())
        }
        return ctx.JSON(core.Map{"data": result})
    })

    // Get All Users
    ctrl.Get("", func(ctx core.Ctx) error {
        result, err := repo.FindAll(nil, sqlorm.FindOptions{})
        if err != nil {
            return core.InternalServerException(ctx.Res(), err.Error())
        }
        return ctx.JSON(core.Map{"data": result})
    })

    // Update and Delete endpoints can be added similarly...
    return ctrl
}
```

### Query Builder Example

The `sqlorm` module provides a convenient, chainable query builder for advanced SQL queries using GORM under the hood. This allows you to construct complex WHERE clauses, conditions, and filtering logic with a simple and type-safe API.

Use a function with the `*sqlorm.QueryBuilder` argument to define your query:

#### Example 1: Simple Filtering

```go
users, err := repo.FindAll(func(qb *sqlorm.QueryBuilder) {
    qb.Equal("name", "John")
})
```

This generates: `WHERE name = 'John'`

#### Example 2: Multiple Conditions

```go
users, err := repo.FindAll(func(qb *sqlorm.QueryBuilder) {
    qb.Equal("age", 30).MoreThan("id", 10)
})
```
This generates: `WHERE age = 30 AND id > 10`

#### Example 3: IN, OR, and NOT

```go
users, err := repo.FindAll(func(qb *sqlorm.QueryBuilder) {
    qb.In("name", "Alice", "Bob", "Charlie").Or("email", "admin@example.com")
})
```
This generates: `WHERE name IN ('Alice','Bob','Charlie') OR email = 'admin@example.com'`

#### Example 4: Range Queries

```go
users, err := repo.FindAll(func(qb *sqlorm.QueryBuilder) {
    qb.MoreThanOrEqual("age", 18).LessThan("age", 30)
})
```
This generates: `WHERE age >= 18 AND age < 30`

#### Example 5: Combined with FindOptions

```go
opts := sqlorm.FindOptions{
    Order: []string{"created_at desc"},
    Limit: 10,
}
users, err := repo.FindAll(func(qb *sqlorm.QueryBuilder) {
    qb.Equal("status", "active")
}, opts)
```

#### API Reference

- `Equal(column, value)`
- `Not(column, value)`
- `Or(column, value)`
- `In(column, ...values)`
- `MoreThan(column, value)`
- `MoreThanOrEqual(column, value)`
- `LessThan(column, value)`
- `LessThanOrEqual(column, value)`

Each method can be chained for expressive queries.

## Multi-Tenancy

The module provides a `tenancy` submodule for dynamic connection per tenant:

```go
import "github.com/tinh-tinh/sqlorm/v2/tenancy"

tenancy.ForRoot(tenancy.Options{
    Connect: tenancy.ConnectOptions{
        Host: "...", User: "...", Password: "...", Port: 5432,
    },
    GetTenantID: func(r *http.Request) string { ... },
    Models: []interface{}{User{}},
    Sync: true,
})
```

You can inject tenant-specific repositories in your controllers by context.

## Models

You can use the provided base models or define your own:

```go
type User struct {
    sqlorm.Model
    Name  string
    Email string
}
```
The base models provide UUID ID, CreatedAt, UpdatedAt, and soft delete support.

## Utilities

Convert Go types to SQL nullable types with helpers like:

```go
sqlorm.ToSqlNullString(&str)
sqlorm.ToSqlNullTime(&t)
```

## Error Handling & Retry

Configure automatic connection retry with:

```go
Retry: &sqlorm.RetryOptions{
    MaxRetries: 3,
    Delay:      time.Second * 2,
}
```

For more advanced usage, such as custom query building, batch operations, or multi-tenant repository injection, refer to the [sqlorm source code](https://github.com/tinh-tinh/sqlorm).