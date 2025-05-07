---
sidebar_position: 2
---

# Module

A unit for handle business with handler and instance data.

![image](./img/module.png)

Each application has a least one module, a **root module**. The root module is the starting point to build the application. Have a different in root module with sub module.

Syntax for create root module in Tinh Tinh:

```go
package app

import "github.com/tinh-tinh/tinhtinh/v2/core"

func NewModule() core.Module {
	appModule := core.NewModule(core.NewModuleOptions{})

	return appModule
}
```

Create child module:

```go
package sub

import "github.com/tinh-tinh/tinhtinh/v2/core"

func NewModule(module core.Module) core.Module {
	subModule := module.New(core.NewModuleOptions{})

	return subModule
}
```

Although have different syntax for create root module and sub module, but it will receive same options.

Options among:
- **Scope**: Have two type scopes, Global and Request.
- **Imports**: Array of function will return value as `Modules`
- **Controllers**: Array of function will return value as `Controllers`
- **Providers**: Array of function will return value as `Providers`
- **Exports**: Array of function will return value as `Providers`