+++
author = "Jefferson Rocha"
title = "#004 | Os diferentes tipos de delegates em C#"
date = 2023-02-06T02:06:06Z
description = ""
subtitle = "Em C# há tipos de variáveis para representar todos os elementos da linguagem, e variáveis que representam métodos são chamadas de delegates."
header_img = ""
short = false
toc = true
tags = ["csharp", "dotnet"]
categories = []
series = []
comment = true
+++

# Delegates em C#

`Delegates` são usados para referenciar métodos.
Da mesma forma que `int` é usado para números inteiros e `string` para texto, usamos `delegate` para referenciar métodos.

### Na prática

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

A classe `BusinessSerializer` declara os métodos `Serialize` e `Deserialize`, mas a sua implementação depende dos delegates recebidos no construtor.

```csharp
    using System.Text.Json;

    AnyClass anyObject = new AnyClass(JsonSerializer.Serialize, JsonSerializer.Deserialize);
```

## Action, Func e Predicate

Delegate é extremamente versátil por ser genérico e poder ter uma referência para qualquer método. 
Para deixar o código mais semântico podemos usar 3 tipos especializados de delegates:

### Action
```csharp
    public delegate void Action<in T>(T obj);
```

Este delegate representa uma ação que não tem retorno (`void`). 
Os argumentos do método podem ser especificados usando `Type Parameters`.

```csharp
    Action<string> WriteToSink = Console.WriteLine;

    WriteToSink("Esta mensagem será exibida no Console.");
```

`Action` são comumente usadas em Xamarin.Forms/MAUI para representar `Commands` (eventos).

### Func

```csharp
    public delegate TResult Func<in T,out TResult>(T arg);
```

Este é o mais genérico entre os três.
É possível especificar todos os parâmetros e retorno usando `Type Parameters`.

```csharp        
    static double Divide(int dividend, int divisor) => dividend / divisor;

    Func<int, int, double> DivideMethod = Divide;

    DivideMethod(21, 7); // Retorna 3.
```

`DivideMethod`, sendo efetivamente um ponteiro para um método, vai executar `Divide` ao ser chamado.

### Predicate

```csharp
    public delegate bool Predicate<in T>(T obj);
```

Um predicado é um método que testa um ou mais condições - e retorna `bool`.
É possível especificar todos os parâmetros e retorno usando `Type Parameters`.

```csharp        
    static bool IsEvenNumber(int number) => number % 2 == 0;

    Predicate<int> NumberCheck = IsEvenNumber;

    if(NumberCheck(6)) // Retorna verdadeiro.
    {
        // Processa o número par.
    }
```

Predicados têm a limitação de um parâmetro de entrada.

## TL;DR

Enquanto o tipo `delegates` é flexível e aplicável em qualquer situação, isso também é seu ponto negativo, pois é preciso analisar sua assinatura para entender seu uso.

`Action`, `Func` e `Predicate` permitem demonstrar intenção de forma mais clara ao categorizar os tipos de `delegates` em ações, métodos com retorno arbitrário e métodos que testam uma condição.