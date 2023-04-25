+++
author = "Jefferson Rocha"
title = "#003 | Value types are not always allocated onto the stack"
date = 2023-01-30T05:22:47Z
description = ""
subtitle = "When learning C# one of the first things we are taught is that ValueType variables are always allocated on the stack, but this is not always true."
header_img = ""
short = false
toc = true
tags = ["csharp", "dotnet", "memory management"]
categories = []
series = []
comment = true
+++

## Background

.Net has two ways to allocate variables in memory: **stack** and **heap**.

A **stack** stores all variables of type value within the execution scope of the current frame.
The **heap** stores all reference-type objects.

The big difference between these two structures is that the memory allocated on the stack is released after the execution of the frame, while what is on the heap is deallocated by the garbage collector.

⭐**Important**: the application stops all processing while the *garbage collector* is running, so many allocations into the **heap** can result in performance degradation.

## When `ValueType`s are allocated on the heap

When they are declared as properties of a class.
In this case the value type variable is allocated with its parent object.

```csharp
class Calculator
{
	// Allocated into the heap along with the Calculator instance.
	private int _currentvalue = default;

	int Sum(int x, int y)
	{
		// x and y are allocated on the stack and memory is
		// returned to the system after executing the method.
		return x + y;
	}
}
```
In the example above, the variables `x` and `y` are allocated on the stack when the `Sum` method is executed and deallocated at the end of the execution.

The `_currentvalue` variable, despite being of value type, is allocated on the heap along with the `Calculator` instance.

## How to guarantee that the object will always be allocated on the stack?

C# 7.2 introduced the `ref struct` type which guarantees that it will always be allocated on the stack and never escape to the heap.
For that there are a few restrictions, most notably its use as a property of a type that can be allocated in the heap is not allowed.

## Como garantir que o objeto sempre será alocado na stack?

### In practice

The following code does not compile:

```csharp
    public ref struct MyRefStruct { }

    public class AnyClass
    {
        MyRefStruct _myRefStruct;
    }
```
And fails with the message:
> Field or auto-implemented property cannot be of type 'MyRefStruct' unless it is an instance member of a ref struct.


## TL;DR

Value type variables are allocated onto the stack only when they are declared within a method.
When an object has a value-type property, the property is allocated on the heap alongside its parent object.