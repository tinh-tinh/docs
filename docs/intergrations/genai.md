---
title: 🤖 Generative AI
sidebar_position: 10
---

# Gen AI

This package supports interacting with AI models.

## Install

```bash
go get -u github.com/tinh-tinh/genai/v2
```

## Usage

```go
func Module() core.Module {
  appModule := core.NewModule(core.NewModuleOptions{
    Imports: []core.Modules{
      ai.ForRoot(option.WithAPIKey(os.Getenv("API_KEY"))),
    },
  })

  return appModule
}
```

When you use `ForRoot`, you initialize an AI client. If you want to perform actions, you need to specify an AI model.

```go
func UserModule(module core.Module) core.Module {
  return module.New(core.NewModuleOptions{
    Imports: []core.Modules{
      ai.ForFeature("gemini-1.5-flash"),
    },
    Controllers: []core.Controllers{
      userController,
    },
  })
}
```

In this case, we use the AI model "gemini-1.5-flash", and this is how to use it in a module:

```go
 func Controller(module core.Module) core.Controller {
  ctrl := module.NewController("users")

  ctrl.Get("", func(ctx core.Ctx) error {
    model := ai.InjectModel(module, "gemini-1.5-flash")
    resp, err := model.GenerateContent(context.Background(), genai.Text("Write a story about a magic backpack."))
    if err != nil {
      fmt.Println(err)
      return err
    }
    return ctx.JSON(core.Map{
      "data": resp,
    })
  })
  return ctrl
}
```