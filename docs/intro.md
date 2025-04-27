---
sidebar_position: 1
---

# Introduce

Tinh Tinh is a framework for building efficient scalable Golang server side applications. It is a [Nest](https://docs.nestjs.com) expired web framework built on top of net/http for Go. Designed to ease things up for fast development with zero memory allocation and performance in mind.

These docs for Tinh Tinh, which was released on September 15th, 2024.

## Philosophy

In recent years, so have many Go framework release, but plenty of super libraries, helpers, and tools exist for Go, none of them effectively solve the main problem of - Architecture.

Tinh Tinh providers an out-of-the-box application architecture which allows developers and teams to create highly testable, scalable, loosely coupled and easily maintainable applications. The architecture is heavily inspired by Nest.

## Installation

First of all, download and install Go 1.22 or higher is required.

Init your go module:

```bash
go mod init your-package
```

Installation is done use the go get command:

```bash
go get -u github.com/tinh-tinh/tinhtinh/v2
```

Install cli to easy to init project. 

```bash
go install github.com/tinh-tinh/tinhtinh-cli@latest
```

After then 

```bash
tinhtinh-cli init
```

After init, you can run go by common command:
```bash
go run main.go
```

The sever will be run on **`http://localhost:3000`**, you can open browser and see it.