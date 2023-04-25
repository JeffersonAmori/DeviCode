+++
author = "Jefferson Rocha"
title = "#004 | Different types of delegates in C#"
date = 2023-02-06T02:06:06Z
description = ""
subtitle = "C# has variable types to represent all elements of the language, and variables to represent methods are called delegates."
header_img = ""
short = false
toc = true
tags = ["csharp", "dotnet"]
categories = []
series = []
comment = true
+++

# Delegates em C#

`Delegates` are used to refer to methods.
In the same way that `int` is used for integers and `string` for text, we use `delegate` to refer to methods.

### In practice

Consider the following code:

```csharp

    delegate string? Serialize(object value, JsonSerializerOptions? options = null);
    delegate object? Deserialize(string value, Type returnType, JsonSerializerOptions? options = null);

    class BusinessSerializer
    {
        Serialize PointerToExternalMethodSerialize { get; }
        Deserialize PointerToExternalMethodDeserialize { get; }

        AnyClass(Serialize referenceToSomeSerializeMethod, Deserialize referenceToSomeDeserializeMethod)
        {
            PointerToExternalMethodSerialize = referenceToSomeSerializeMethod;
            PointerToExternalMethodDeserialize = referenceToSomeDeserializeMethod;
        }

        string? Serialize(object value) => PointerToExternalMethodSerialize(value);
        object? Deserialize(string value, Type returnType) => PointerToExternalMethodDeserialize(value, returnType);
    }
```

The `BusinessSerializer` class declares the `Serialize` and `Deserialize` methods, but their implementation depends on the delegates received in the constructor.

```csharp
    using System.Text.Json;

    AnyClass anyObject = new AnyClass(JsonSerializer.Serialize, JsonSerializer.Deserialize);
```

## Action, Func and Predicate

Delegate is extremely versatile as we can reference any method with it.
To make the code more semantic we can use 3 specialized types of delegates: they're `Action`, `Func` and `Predicate`.

### Action
```csharp
     public delegate void Action<in T>(T obj);
```

This delegate represents an action that has no return (`void`).
Method arguments can be specified using `Type Parameters`.

```csharp
     Action<string> WriteToSink = Console.WriteLine;

     WriteToSink("This message will be displayed in the Console.");
```

`Action` are commonly used in Xamarin.Forms/.Net MAUI to represent `Commands` (events).

### Function

```csharp
     public delegate TResult Func<in T,out TResult>(T arg);
```

This is the most generic among the three.
It is possible to specify all parameters and return using `Type Parameters`.

```csharp
     static double Divide(int dividend, int divisor) => dividend / divisor;

     Func<int, int, double> DivideMethod = Divide;

     DivideMethod(21, 7); // Returns 3.
```

`DivideMethod`, effectively being a pointer to a method, will execute `Divide` when called.

### Predicate

```csharp
     public delegate bool Predicate<in T>(T obj);
```

A predicate is a method that tests one or more conditions and always returns `bool`.
It is possible to specify all parameters types using `Type Parameters`.

```csharp
     static bool IsEvenNumber(int number) => number % 2 == 0;

     Predicate<int> NumberCheck = IsEvenNumber;

     if(NumberCheck(6)) // Evaluates to true.
     {
         // Process the even number.
     }
```

Predicates have the limitation of one input parameter.

## TL;DR

While the `delegates` type is flexible and applicable in any situation, this is also its downside, as one has to parse its signature to understand its usage.

`Action`, `Func` and `Predicate` allow you to demonstrate intent more clearly by categorizing the types of `delegates` into actions, methods with arbitrary return and methods that test a condition.