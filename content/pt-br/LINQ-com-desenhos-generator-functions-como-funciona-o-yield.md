+++
author = "Jefferson Rocha"
title = "Generator-Function in C# - como o yield funciona?"
date = "2023-01-16"
description = ""
header_img = ""
short = false
toc = true
tags = ["csharp", "async", "dotnet"]
categories = ["csharp", "async", "dotnet"]
series = []
comment = true
+++

Vamos ver um pequeno exemplo para examinar o comportamento. 
Criamos uma pequena aplicação de console que tem um "gerador de string". 
Implementaremos o gerador com a palavra-chave `yield`.
Isso significa que estamos obtendo nossas strings "uma por vez" e não todo o conteúdo de uma só vez. 
Para tornar as coisas mais interessantes, saímos do loop quando recebemos dois elementos do gerador.

```csharp
using System;
using System.Collections.Generic;					

public class Program
{
    public static void Main()
    {
        var loopCounter = 0;
        Console.WriteLine("Chamando gerador");
        var strings = Generator();
        Console.WriteLine("Obtendo itens do gerador");
        foreach (var elem in strings)
        {
            Console.WriteLine($"Do gerador: {elem}");
            loopCounter++;
            
            if (loopCounter == 2)
                break;
        }				
    }
    
    private static IEnumerable<string> Generator()
    {
        Console.WriteLine("Iniciar gerador");
        Console.WriteLine("Fornecendo primeira string ao consumidor");
        yield return "Olá";
        Console.WriteLine("Fornecendo segunda string ao consumidor");
        yield return "Mundo";
        Console.WriteLine("Fornecendo última string ao consumidor");
        yield return "último";
        Console.WriteLine("Concluído");
    }
}
```

Que gera a seguinte saída:

```txt
Chamando gerador
Obtendo itens do gerador
Iniciar gerador
Fornecendo primeira string ao consumidor
Do gerador: Olá
Fornecendo segunda string ao consumidor
Do gerador: Mundo
```

Agora, como seria todo o processo se não usássemos a palavra-chave `yield` e retornássemos a lista inteira de uma só vez?

```csharp
using System;
using System.Collections.Generic;					

public class Program
{
    public static void Main()
    {
        var loopCounter = 0;
        Console.WriteLine("Chamando gerador");
        var strings = Generator();
        Console.WriteLine("Obtendo itens do gerador");
        foreach(var elem in strings)
        {
            Console.WriteLine($"Do gerador: {elem}");
            loopCounter++;
            
            if(loopCounter == 2)
                break;
        }				
    }
    
    private static IList<string> Generator()
    {
        List<string> gen = new();
        Console.WriteLine("Início do gerador");
        Console.WriteLine("Adicionando primeira string ao consumidor");
        gen.Add("Hello");
        Console.WriteLine("Adicionando segunda string ao consumidor");
        gen.Add("World");
        Console.WriteLine("Adicionando última string ao consumidor");
        gen.Add("last");
        Console.WriteLine("Feito");
        return gen;
    }
}
```

Gera a seguinte saída:

```txt
Chamando gerador
Início do gerador
Adicionando primeira string ao consumidor
Adicionando segunda string ao consumidor
Adicionando última string ao consumidor
Feito
Obtendo itens do gerador
Do gerador: Hello
Do gerador: World
```

## Principais diferenças entre `IEnumerable` com `yield` e uma `List`

Vimos algumas diferenças na saída, então vamos resumir:

O bloco do gerador não é executado até chegarmos ao consumidor pedindo o primeiro elemento (vimos que `var string = Generator()` não resultará em nenhuma saída no console).
A execução do método gerador para nas fronteiras de `yield`.
Se não consumirmos mais itens, não chegaremos ao fim do bloco gerador

## Por que isso importa?

Bem, acho que quase todos vocês usaram a função `Any()` do LINQ, certo? 
Se tivermos uma enumeração de itens e quisermos verificar se algum deles atende ao nosso predicado.

```csharp
var numbers = new[] { 1, 2, 3, 4 };
var hasPositiveValues = numbers.Any(n => n > 0);
```

Aqui, o que acontece é que basicamente precisamos verificar apenas o primeiro elemento de `numbers` e sabemos que os requisitos foram cumpridos e podemos retornar `true` com segurança.
Claro, estamos operando em dados que vivem na RAM. Nada demais. Mas imagine que consumimos uma interface que pode ou não ser custosa:

```csharp
IEnumerable<int> numbers = SomeSlowOperation();
var hasPositiveValues = numbers.Any(n => n > 0);
```

Agora, enumeramos apenas um por um e podemos poupar potencialmente muito tempo e recurso. 
No pior dos casos, ainda temos que percorrer a lista inteira. 
Se o seu tipo de dados fosse IList<int>, tudo seria materializado de uma vez, mesmo se precisássemos apenas do primeiro elemento (estou olhando para você `Take(1)`/`First()`).
Claro, também há uma armadilha:

```csharp
IEnumerable<int> numbers = SomeSlowOperation();

var evenNumbers = numbers.Where(n => n % 2 == 0);
var oddNumbers = numbers.Where(n => n % 2 == 1);
```

Agora, todo o gerador é executado duas vezes. Então, esteja ciente de múltiplas enumerações de `IEnumerable`.

E como isso funciona agora?
E o que tem a ver com `async`/`await`? A resposta é simples: ambos criam uma state machine.
Então, vamos dar uma olhada em como (simplificadamente) a state machine funciona:

![state_machine](../../../images/LINQ-com-desenhos/yield.png)


1. Nossa função `yield` basicamente corta em vários *estados* cortados diretamente na borda de `yield`.
2. Agora, se o consumidor solicitar um valor, a state machine executará tudo até o primeiro `yield`, configurará a state machine para a próxima chamada e retornará o valor.
3. O consumidor mais uma vez solicita outro valor. A state machine saltará para o próximo estado para retornar o valor atrás do segundo `yield` e novamente configurará a state machine e retornará o valor.

## Como isso parece em ação?

Cada função de `yield`/gerador será sua própria classe representando a state machine. Muito brevemente, é assim:

```csharp
public IEnumerable<string> GetStrings()
{
    Console.WriteLine("Entrar na função");
    yield return "Primeiro";
    Console.WriteLine("Outro");
    yield return "Segundo";
}
```

Se tornará isso:

```csharp
public GetStringsStateMachine : IEnumerable<string>, IEnumerator<string>
{
    private int currentState;    // Representa o estado que será executado quando entrar na função novamente
    private string currentValue; // Representa o valor retornado

    public GetStringsStateMachine()
    {
    }

    IEnumerator<string>.Current => currentValue; // É assim que um consumidor (como foreach) solicitaria um valor.

    private bool MoveNext()
    {
        switch (currentState)
        {
            default:
                return false;
            case 0:
                Console.WriteLine("Entrar na função");
                currentValue = "Primeiro";
                currentState = 1; // Ir para o próximo passo na state machine
                return `true`; // Verdadeiro significa que ainda temos valores e não estamos concluídos
            case 1:
                Console.WriteLine("Outro");
                currentValue = "Segundo";
                return false; // False significa que estamos concluídos. Não há mais valores
        }
    }
}
```

Vejamos que, basicamente, dividimos a função na fronteira do `yield` com seu conteúdo dentro de alguns casos de switch. Uma state machine simples.
A state machine implementa `IEnumerator` e `IEnumerable` em si para usar a "mágica" do `MoveNext()`. Se tivermos itens, nossa state machine colocará seu estado para o próximo nível, salvará o valor atual para que ele possa ser exposto ao mundo exterior e retornará `true`, se houver mais elementos a vir, ou false.

Agora vamos dar uma olhada em um consumidor típico:

```csharp
foreach (var item in GetStrings())
{
    // ...
}
```

Se torna isso:

```csharp
IEnumerator<string> enumerador = GetStrings().GetEnumerator();
while (enumerador.MoveNext())
{
    string atual = enumerador.Current;
}

private static IEnumerable<string> GetStrings()
{
    return new GetStringsStateMachine();
}
```

Agora vamos desembrulhar isso.

1. Estamos criando a state machine. Chamamos o método `GetStrings()` para inicializar a state machine. 
Mas espere um segundo? Como o compilador pode gerar o mesmo método com o mesmo nome? Bem, ele não o faz. 
Vimos anteriormente que o compilador gera toda a state machine. E o que resta no seu método original é exatamente isso. 
A inicialização da sua state machine.
2. Tudo o que precisamos fazer é obter o enumerador da state machine.
3. Enquanto temos itens (lembre-se que `MoveNext` retorna verdadeiro enquanto ainda temos itens) fazemos nosso loop.
4. Para obter o item atual, usamos `IEnumerator<string>.Current` para obtê-lo. Muito simples.

Agora, como você pode imaginar, isso não é tudo. É um pouco mais complicado. 
E se tivermos diferentes threads aqui? Ou exceções? 
A verdadeira state machine é um pouco mais complexa. Mas você encontrará todos os princípios que mostrei lá também.
Se quiser dar uma olhada, dê uma olhada no [sharplab.io](https://sharplab.io/#v2:EYLgtghglgdgNAExAagD4AEBMBGAsAKHQAYACdbAFgG4CD0BmMzEgWQE8BhAGwgGdeSAbwIlRJAA4AnKADcIAFwCmZCiQCqvRQGV50mAHNeACgCUIscPxjrJAGYB7SYogBjABYkjcySShKwvjAkAOKK8jp6hqZmVjailnFx5nEAvskkabGJYulSsgrK5PQAPOREAHwhYRGwUTHWCYnkAJxGAEQAojBKPrYArjAu8lD2MG0mNFk25NhkAOwkbQBiUJK88m2T2S3tAIIw9vJuipLjW03Ys+gLbVqKLqMIm+mZKUA==).
Se você não conhece esse site, recomendo usá-lo e ler sobre ele. Mostra o que o compilador faz com seu código.