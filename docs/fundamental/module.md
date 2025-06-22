---
sidebar_position: 2
---

# Module

A unit for handling business logic with handler and instance data.

![image](./img/module.png)

Each application has at least one module: the **root module**. The root module is the starting point for building the application. There is a difference between the root module and submodules.

Syntax to create a root module in Tinh Tinh:

```go
package app

import "github.com/tinh-tinh/tinhtinh/v2/core"

func NewModule() core.Module {
	appModule := core.NewModule(core.NewModuleOptions{})

	return appModule
}
```

Create a child module:

```go
package sub

import "github.com/tinh-tinh/tinhtinh/v2/core"

func NewModule(module core.Module) core.Module {
	subModule := module.New(core.NewModuleOptions{})

	return subModule
}
```

Although there is a different syntax for creating the root module and submodules, they receive the same options.

Options include:
- **Scope**: There are two types of scopes, Global and Request.
- **Imports**: Array of functions that return values as `Modules`
- **Controllers**: Array of functions that return values as `Controllers`
- **Providers**: Array of functions that return values as `Providers`
- **Exports**: Array of functions that return values as `Providers`