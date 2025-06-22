---
title: 🛢 Queue
sidebar_position: 6
---

# Queue Module for Tinh Tinh

The Queue module provides a robust, Redis-based job queue for the Tinh Tinh framework, supporting job scheduling, rate limiting, retries, concurrency, delayed jobs, priorities, and more.

---

## Features

- **Redis-Based:** Robust persistence and distributed processing.
- **Delayed Jobs:** Schedule jobs to run after a delay.
- **Cron Scheduling:** Schedule and repeat jobs using cron patterns.
- **Rate Limiting:** Control job processing rate.
- **Retries:** Automatic retry on failure.
- **Priority:** Job prioritization.
- **Concurrency:** Multiple workers per queue.
- **Pause/Resume:** Temporarily stop and resume job processing.
- **Crash Recovery:** Recovers jobs after process crashes.
- **Remove on Complete/Fail:** Clean up jobs after handling.

---

## Installation

```bash
go get -u github.com/tinh-tinh/queue/v2
```

---

## Quick Start

### 1. Register the Module

```go
import "github.com/tinh-tinh/queue/v2"

queueModule := queue.ForRoot(&queue.Options{
    Connect: &redis.Options{
        Addr: "localhost:6379",
        DB:   0,
    },
    Workers: 3,
    RetryFailures: 3,
})
```

Or via factory:

```go
queueModule := queue.ForRootFactory(func(ref core.RefProvider) *queue.Options {
    return &queue.Options{ /* ... */ }
})
```

### 2. Register and Inject Queues

```go
userQueueModule := queue.Register("user") // uses default/global options

// In your service or controller:
userQueue := queue.Inject(module, "user")
```

---

## Example: Adding and Processing Jobs

```go
// Create a queue instance
userQueue := queue.New("user", &queue.Options{
    Connect: &redis.Options{
        Addr: "localhost:6379",
        DB:   0,
    },
    Workers: 3,
    RetryFailures: 3,
    Limiter: &queue.RateLimiter{
        Max: 3,
        Duration: time.Second,
    },
})

// Define job processing logic
userQueue.Process(func(job *queue.Job) {
    job.Process(func() error {
        // Your job logic here
        fmt.Println("Processing job:", job.Id, job.Data)
        return nil
    })
})

// Add a job
userQueue.AddJob(queue.AddJobOptions{
    Id:   "1",
    Data: "some data",
})
```

---

## Advanced Usage

### Bulk Add Jobs

```go
userQueue.BulkAddJob([]queue.AddJobOptions{
    {Id: "2", Data: "value 2"},
    {Id: "3", Data: "value 3"},
    // ...
})
```

### Pause and Resume

```go
userQueue.Pause()
userQueue.AddJob(queue.AddJobOptions{Id: "4", Data: "delayed"})
userQueue.Resume()
```

### Scheduling (Cron)

```go
scheduledQueue := queue.New("scheduled", &queue.Options{
    Connect: &redis.Options{Addr: "localhost:6379", DB: 0},
    Pattern: "@every 1m", // every 1 minute
    Workers: 1,
})
scheduledQueue.Process(func(job *queue.Job) {
    job.Process(func() error {
        // Job logic
        return nil
    })
})
scheduledQueue.AddJob(queue.AddJobOptions{Id: "1", Data: "scheduled data"})
```

### Delayed Jobs

```go
delayQueue := queue.New("delayed", &queue.Options{
    Connect: &redis.Options{Addr: "localhost:6379", DB: 0},
    Delay:   5 * time.Second,
})
delayQueue.Process(func(job *queue.Job) {
    job.Process(func() error {
        // Job logic
        return nil
    })
})
delayQueue.AddJob(queue.AddJobOptions{Id: "1", Data: "delayed data"})
```

### Remove on Complete or Fail

```go
queueWithCleanup := queue.New("clean", &queue.Options{
    Connect: &redis.Options{Addr: "localhost:6379", DB: 0},
    RemoveOnComplete: true,
    RemoveOnFail:     true,
})
```

---

## Job Status & Counts

```go
completed := userQueue.CountJobs(queue.CompletedStatus)
failed := userQueue.CountJobs(queue.FailedStatus)
waiting := userQueue.CountJobs(queue.WaitStatus)
```

---

## Crash Recovery

Jobs are automatically recovered and re-processed after process crashes.

---

For advanced patterns and full API, see the [queue source code](https://github.com/tinh-tinh/queue).