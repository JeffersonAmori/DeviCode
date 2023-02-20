+++
author = "Jefferson Rocha"
title = "#006 | The problem with ILogger"
date = 2023-02-20T06:12:03Z
description = ""
subtitle = "The current interface ILogger could be way more efficient by leveraging generics"
header_img = ""
short = false
toc = true
tags = ["Fluent validation", "csharp", "dotnet"]
categories = []
series = []
comment = true
+++

# The problem with `ILogger`

The current implementation of [`ILogger`](https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.logging.ilogger?view=dotnet-plat-ext-7.0) uses `object[]` in its signatures, which means any value passed to it either 1) is already on the heap or 2) is boxed.

```csharp
Log(ILogger, LogLevel, Exception, String, Object[])
```

Now consider the app logs the salary of any given `Employee`:

```csharp
class Employee
{
    public int Id { get; set; }
    public string Name { get; set; }
    public DateTime DateOfBirth { get; set; }
    public decimal Salary { get; set; }
}

// Creating the Employee.
var employee = new Employee() { /* ... */};

// ...

// Loggin the salary for the employee.
logger.Log(LogLevel.Information, "The Employee {name} makes {salary}.", employee.Name, employee.Salary);
```

The problem with this approach is that, even if the information log level is not enabled, we're still boxing (allocating into the heap) the salary value.
This has no real value if no logs are generated.

# Generics to the rescue

By  using generics, we can avoid allocating `value-types` into the heap by avoiding the boxing from `decimal` into `object`.

```csharp
void Log<T1>(LogLevel logLevel, string message, T1 param1);
void Log<T1, T2>(LogLevel logLevel, string message, T1 param1, T2 param2);
```

By using generics in the signature the compiler can then figure out that the values we're passing to the logger are `reference-` or `value-types`, avoiding the unnecessary boxing.

⭐ **Important**: The more often the GC has to run to clear the heap, the bigger the performance penalty.

# Benchmark

Here's the benchmark for the `LoggerAdapter`, which can be ~20% faster when the log level is disabled.
You can check out the implementation of `LoggerAdapter`, see [this class](https://github.com/JeffersonAmori/HighLowGame/blob/master/LoggerAdapter/ILoggerAdapter.cs).

``` text
BenchmarkDotNet=v0.13.2, OS=Windows 10 (10.0.19045.2364)
Intel Xeon CPU E5-1620 v4 3.50GHz, 1 CPU, 8 logical and 4 physical cores
.NET SDK=7.0.101
  [Host]     : .NET 6.0.12 (6.0.1222.56807), X64 RyuJIT AVX2  [AttachedDebugger]
  DefaultJob : .NET 6.0.12 (6.0.1222.56807), X64 RyuJIT AVX2


|                          Method | Count |         Mean |      Error |     StdDev |       Median | Ratio | RatioSD |      Gen0 |     Gen1 |     Gen2 |  Allocated | Alloc Ratio |
|-------------------------------- |------ |-------------:|-----------:|-----------:|-------------:|------:|--------:|----------:|---------:|---------:|-----------:|------------:|
|     LoggerAdapter_WithValueType |   100 |     82.14 us |   1.634 us |   3.654 us |     81.28 us |  0.93 |    0.04 |   15.9912 |   0.8545 |        - |   81.91 KB |        0.85 |
| LoggerAdapter_WithReferenceType |   100 |     84.14 us |   1.678 us |   3.233 us |     83.19 us |  0.94 |    0.04 |   15.5029 |   0.7324 |        - |   79.56 KB |        0.82 |
|           ILogger_WithValueType |   100 |     90.16 us |   1.517 us |   1.345 us |     90.29 us |  1.00 |    0.00 |   18.7988 |   1.7090 |        - |   96.58 KB |        1.00 |
|       ILogger_WithReferenceType |   100 |     92.73 us |   1.339 us |   1.118 us |     93.16 us |  1.03 |    0.02 |   18.4326 |   1.5869 |        - |   94.24 KB |        0.98 |
|                                 |       |              |            |            |              |       |         |           |          |          |            |             |
|     LoggerAdapter_WithValueType |  1000 |    795.12 us |  13.559 us |  12.020 us |    798.17 us |  0.86 |    0.02 |  154.2969 |  42.9688 |        - |   792.3 KB |        0.84 |
| LoggerAdapter_WithReferenceType |  1000 |    809.43 us |  15.959 us |  24.846 us |    799.37 us |  0.88 |    0.03 |  150.3906 |  37.1094 |        - |  768.86 KB |        0.82 |
|           ILogger_WithValueType |  1000 |    927.77 us |  16.804 us |  15.718 us |    927.99 us |  1.00 |    0.00 |  176.7578 |  60.5469 |        - |  940.69 KB |        1.00 |
|       ILogger_WithReferenceType |  1000 |    923.30 us |  11.470 us |  10.168 us |    920.73 us |  0.99 |    0.02 |  172.8516 |  58.5938 |        - |  917.25 KB |        0.98 |
|                                 |       |              |            |            |              |       |         |           |          |          |            |             |
|     LoggerAdapter_WithValueType | 10000 | 12,360.32 us | 235.461 us | 231.254 us | 12,381.22 us |  0.81 |    0.02 | 1296.8750 | 546.8750 | 265.6250 | 7994.95 KB |        0.84 |
| LoggerAdapter_WithReferenceType | 10000 | 12,021.00 us | 240.273 us | 661.783 us | 11,753.21 us |  0.79 |    0.05 | 1234.3750 | 468.7500 | 234.3750 | 7760.56 KB |        0.82 |
|           ILogger_WithValueType | 10000 | 15,300.89 us | 279.522 us | 247.789 us | 15,250.01 us |  1.00 |    0.00 | 1546.8750 | 640.6250 | 312.5000 | 9479.68 KB |        1.00 |
|       ILogger_WithReferenceType | 10000 | 17,030.86 us | 336.816 us | 580.992 us | 17,074.59 us |  1.11 |    0.06 | 1625.0000 | 625.0000 | 312.5000 |  9245.3 KB |        0.98 |
```

TL;DR

The current signature for `ILogger` uses `object[]`, which is not best performance-wise. For hot paths using `Generics` might be worthwhile.