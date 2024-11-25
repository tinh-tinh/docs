---
title: 🤖 Generative AI
sidebar_position: 10
---

# Gen AI

Package support interactive with AI Model

## Install

```bash
go get -u github.com/tinh-tinh/genai
```

## Usage

```go
func Module() *core.DynamicModule {
  appModule := core.NewModule(core.NewModuleOptions{
    Imports: []core.Module{
      ai.ForRoot(option.WithAPIKey(os.Getenv("API_KEY"))),
    },
  })

  return appModule
}
```

When use `ForRoot` you init a AI Client, if you want to action you need specific AI Model.

```go
func UserModule(module *core.DynamicModule) *core.DynamicModule {
  return module.New(core.NewModuleOptions{
    Imports: []core.Module{
      ai.ForFeature("gemini-1.5-flash"),
    },
    Controllers: []core.Controller{
      userController,
    },
  })
}
```

In this case we use AI model "gemini-1.5-flash", and this is way to use it in module:

```go
 func Controller(module *core.DynamicModule) *core.DynamicController {
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