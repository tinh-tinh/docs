---
sidebar_position: 2
---

# Module

![image](./img/module.avif)

Each application has a least one module, a **root module**. The root module is the starting point to build the application. Have a different in root module with sub module.

Syntax for create root module in Tinh Tinh:

```go
package app

import "github.com/tinh-tinh/tinhtinh/core"

func Module() *core.DynamicModule {
  module := core.NewModule(core.NewModuleOptions{})
  
  return module
}
```

Create child module:

```go
package sub

import "github.com/tinh-tinh/tinhtinh/core"

func Module(module *core.DynamicModule) *core.DynamicModule {
  subModule := module.New(core.NewModuleOptions{})
  
  return subModule 
}
```

Although have different syntax for create root module and sub module, but it will receive same options.

Options among:
- **Scope**: Have two type scopes, Global and Request.
- **Imports**: Array of function will return value as `DynamicModule`
- **Controllers**: Array of function will return value as `DynamicController`
- **Providers**: Array of function will return value as `DynamicProvider`
- **Exports**: Array of function will return value as `DynamicProvider`