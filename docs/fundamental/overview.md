---
sidebar_position: 1
---

# Overview

Architecture of Tinh Tinh

![image](./img/overview.png)

TinhTinh following modular architecture, with each module incluse components:

- Controllers (as Router): for registry router to handler http
- Providers: Instance with name only available in module have registered.
- Middlewares: common middleware will apply for all controller in module.

# Folder structure 

After you init with tinhtinh-cli, the structure folder will be in:

```
code
  ├── app 
  │   ├── app_controller.go
  │   ├── app_module.go
  │   └── app_service.go
  ├── go.mod
  ├── go.sum
  └── main.go
```

Here's a brief overview of those core files:

- `app_controller.go`: A basic controller with a simple route.

- `app_service.go`: A basic service with crud method.

- `app_module.go`: The root module of the application.

- `main.go`: the entry file of the application which uses the core function `CreateFactory` to create a application instance.

File main is like this:

```go
package main

import (
	"project_name/app"

	"github.com/tinh-tinh/tinhtinh/v2/core"
)

func main() {
	server := core.CreateFactory(app.NewModule)

	server.Listen(3000)
}
```