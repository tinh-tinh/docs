---
title: 🤖 Generative AI
sidebar_position: 10
---

# Generative AI Module for Tinh Tinh

The Tinh Tinh Generative AI module provides seamless integration with Google Generative AI (Gemini, etc.) for Go applications, using dependency injection patterns from the Tinh Tinh framework. It supports easy model registration, multi-model usage, and controller-based text generation.

---

## Features

- **Google Generative AI integration** (via [google/generative-ai-go](https://github.com/google/generative-ai-go))
- **Dependency Injection** for the AI client and models (`ForRoot`, `ForFeature`)
- **Modular**: Register and use multiple AI models in your app
- Easily call model methods (e.g. `GenerateContent`) from your Tinh Tinh controllers and services

---

## Installation

```bash
go get -u github.com/tinh-tinh/genai/v2
```

---

## Quick Start

### 1. Register the AI Client

```go
import ai "github.com/tinh-tinh/genai/v2"
import "google.golang.org/api/option"

appModule := core.NewModule(core.NewModuleOptions{
    Imports: []core.Modules{
        ai.ForRoot(option.WithAPIKey(os.Getenv("API_KEY"))),
    },
})
```

### 2. Register and Use a Model

```go
userModule := func(module core.Module) core.Module {
    return module.New(core.NewModuleOptions{
        Imports: []core.Modules{
            ai.ForFeature("gemini-1.5-flash"), // Register a model by name
        },
        Controllers: []core.Controllers{
            userController,
        },
    })
}

func userController(module core.Module) core.Controller {
    ctrl := module.NewController("users")

    ctrl.Get("", func(ctx core.Ctx) error {
        model := ai.InjectModel(module, "gemini-1.5-flash")
        resp, err := model.GenerateContent(context.Background(), genai.Text("Write a story about a magic backpack."))
        if err != nil {
            return err
        }
        return ctx.JSON(core.Map{
            "data": resp,
        })
    })
    return ctrl
}
```

### 3. Full App Example

```go
appModule := func() core.Module {
    return core.NewModule(core.NewModuleOptions{
        Imports: []core.Modules{
            ai.ForRoot(option.WithAPIKey(os.Getenv("API_KEY"))),
            userModule,
        },
    })
}
```

---

## API

### Register the GenAI Client

- `ai.ForRoot(...option.ClientOption)`: Register the GenAI client globally (with API key or other options).

### Register a Model

- `ai.ForFeature(modelName string)`: Register a specific model (e.g. `"gemini-1.5-flash"`) as a module provider.

### Inject and Use

- `ai.InjectClient(module core.Module)`: Get the `*genai.Client`.
- `ai.InjectModel(module core.Module, name string)`: Get a `*genai.GenerativeModel` for the given model name.

---

## Example: Generate Content

```go
model := ai.InjectModel(module, "gemini-1.5-flash")
result, err := model.GenerateContent(context.Background(), genai.Text("Your prompt"))
if err != nil {
    // handle error
}
fmt.Println(result)
```
---

For advanced usage and latest updates, see the [genai source code](https://github.com/tinh-tinh/genai).