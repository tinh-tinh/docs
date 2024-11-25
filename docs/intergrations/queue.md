---
title: 🛢 Queue
sidebar_position: 6
---

# Queue

Package support create queue for handle task with Redis.

## Install

```bash
go get -u github.com/tinh-tinh/queue
```

## Usage

If using `queue` package, you need install Redis and connect it to run background tasks. Import queue in module:

```go
package user

import (
	"github.com/redis/go-redis/v9"
	"github.com/tinh-tinh/mongoose"
	"github.com/tinh-tinh/queue"
	"github.com/tinh-tinh/tinhtinh/core"
)

func NewModule(module *core.DynamicModule) *core.DynamicModule {
  userModule := module.New(core.NewModuleOptions{
    Imports: []core.Module{
      mongoose.ForFeature(
        mongoose.NewModel[User]("users"),
      ),
      queue.Register("user_update", &queue.Options{
        Connect: &redis.Options{
          Addr:     "localhost:6379",
          DB:       0,
          Password: "",
        },
        Workers:       5,
        RetryFailures: 3,
      }),
    },
    Controllers: []core.Controller{NewController},
    Providers:   []core.Provider{NewService},
  })

  return userModule
}
```

Use in service:

```go
const USER_SERVICE core.Provide = "USER_SERVICE"

type userService struct {
	q *queue.Queue
}

func NewService(module *core.DynamicModule) *core.DynamicProvider {
	userQ := queue.Inject(module, "user_update")

	userQ.Process(func(job *queue.Job) {
		job.Process(func() error {
			a, _ := bcrypt.GenerateFromPassword([]byte(job.Id), 14)
			fmt.Println(a)
			return nil
		})
	})

	svc := module.NewProvider(core.ProviderOptions{
		Name: USER_SERVICE,
		Value: &userService{
			q: userQ,
		},
	})

	return svc
}

func (s *userService) Create(input interface{}) interface{} {
	s.q.AddJob(queue.AddJobOptions{
		Id:   "12345678",
		Data: make(map[string]interface{}),
	})
	return nil
}
```