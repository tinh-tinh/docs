---
sidebar_position: 1
---

# Introduction

Tinh Tinh is a framework for building efficient, scalable Golang server-side applications. It is a [Nest](https://docs.nestjs.com)-inspired web framework built on top of net/http for Go. Designed to facilitate fast development with zero memory allocation and performance in mind.

These are the docs for Tinh Tinh, which was released on May 7, 2025.

## Philosophy

In recent years, many Go frameworks have been released. While there are plenty of excellent libraries, helpers, and tools for Go, none of them effectively solve the main problem: architecture.

Tinh Tinh provides an out-of-the-box application architecture that allows developers and teams to create highly testable, scalable, loosely coupled, and easily maintainable applications. The architecture is heavily inspired by Nest.

## Installation

First, download and install Go 1.22 or higher.

Install the CLI to easily initialize a project:

```bash
go install github.com/tinh-tinh/tinhtinh-cli/v2@latest
```

After installing, initialize your project:

```bash
tinhtinh-cli init project_name
```

After initialization, you can run your project with the following command:
```bash
go run main.go
```

The server will run on **`http://localhost:3000`**. Open your browser to see it.