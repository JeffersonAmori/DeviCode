+++
author = "Jefferson Rocha"
title = "#001 | Async is not about Performance"
date = "2023-01-16"
description = ""
subtitle = "When we think of asynchronous code, performance gains immediately come to mind, but this is not exactly the case."
header_img = ""
short = false
toc = true
tags = ["csharp", "async", "dotnet"]
categories = ["csharp", "async", "dotnet"]
series = []
comment = true
+++

## Context
Before we delve deeper, we need to know that:

1. Task is the promise of a future value.
2. Async is not the same thing as parallelism (multi-thread).
3. Methods can be CPU-bound or IO-bound. CPU-bound relies on the CPU, while IO-bound refers to disks or network.
4. Whenever the compiler finds the keyword, `await` a state machine is created to store the state of the context.

## What happens behind the scenes
When we wait for a method with the keyword `await` we signal to the runtime that, if the thread that is waiting for the result of the asynchronous operation, it can return to the thread pool and execute other tasks.

However, if the method being executed is CPU-bound, the thread that waited for the method will continue to be busy with its execution. In this case, the result is the same as executing the method synchronously, but with the overhead of creating the state machine.

The benefit of async methods comes when we have IO-bound tasks. This means that processor-independent tasks like HTTP calls or reading a file from disk can be scheduled for some time in the future, and in the meantime the thread waiting for this method returns to the thread pool and can handle new requests. requests.

## The scalability aspect
Considering a web applicaion, leveraging `async`/`await` does not mean that the application code will run faster, but that it can handle more requests at the same time if we make available the threads that are waiting for other resources besides the CPU.

## What to keep in mind when using async/await

### Do not access the result directly:

```csharp
public int AddOneAsync()
{
    var result = GetValueAsync().Result;
    return result + 1;
}
```

Always use `await` to get the result:
```csharp
public int AddOneAsync()
{
    var result = await GetValueAsync();
    return result + 1;
}
```
This prevents the current thread from blocking and avoids possible deadlocks.

---
### Be deliberate in choosing whether to use `await` or return the `Task`.

Seeking to improve performance we might convert our methods that just wait for another method:

```csharp
public `async` Task<string> GetStringAsyncWrapperAsync(string uri)
{
  return `await` _htttClient.GetStringAsync(uri);
}
```
Into one that simply returns the `Task`:
```csharp
public Task<string> GetAsyncWrapper(string uri)
{
  return _htttClient.GetAsync(uri);
}
```
This provides the benefit of not having to allocate a new state machine (which is marginally faster and uses less memory), with the drawback that if an exception occurs within any `Task` being returned the stacktrace points to the `Task` first method you didn't use await.