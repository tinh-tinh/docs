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

## Installation

First of all, download and install Go 1.22 or higher is required.

Init your go module:

```bash
go mod init your-package
```

Installation is done use the go get command:

```bash
go get -u github.com/tinh-tinh/tinhtinh
```

Install cli to easy to init project. 

```bash
go install github.com/tinh-tinh/tinhtinh-cli@latest
```

After then 

```bash
tinhtinh-cli init
```

The structure folder will be in:

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