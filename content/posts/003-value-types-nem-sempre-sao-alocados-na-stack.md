+++
author = "Jefferson Rocha"
title = "#003 | Value types nem sempre são alocados na stack"
date = 2023-01-30T05:22:47Z
description = ""
subtitle = "Quando aprendemos C# uma das primeiras coisas que nos é ensinado é que variáveis de tipo valor (ValueType) são sempre alocadas na stack, mas isso nem sempre é verdade."
header_img = ""
short = false
toc = true
tags = ["csharp", "dotnet", "memory management"]
categories = []
series = []
comment = true
+++

## Background

O .Net possui duas formas de alocar variáveis na memória: a **stack** e a **heap**.

A **stack** armazena todas as variáveis do tipo valor dentro do escopo da execução do frame atual.
Já a **heap** (que em português significa literalmente cesta) armazena todos os objectos do tipo referência.

A grande diferença entre essas duas estruturas é que a memória alocada na stack é liberada após a execução do frame, enquanto o que está na heap é desalocado pelo garbage collector.

⭐**Importante**: a aplicação pára todo o processamento enquanto a o *garbage collector* é executado, por isso muitas alocações na **heap** podem resultar em degradação da performance.

## Quando `ValueType`s são alocados na heap

Quando são declarados como propriedades de uma classe. 
Neste caso a variável de tipo valor é alocada com seu objeto-pai.

```csharp
class Calculator
{
	// Alocado na heap junto com a instância de Calculator.
	private int _currentvalue = default;

	int Sum(int x, int y)
	{
		// x e y são alocados na stack e a memória é 
		// devolvida pro sistema após execução do método.
		return x + y;
	}
}
```

No exemplo acima as variáveis `x` e `y` são alocadas na stack quando o método `Sum` é executado e desalocadas no fim da execução.

Já a variável `_currentvalue`, apesar de ser de tipo valor, é alocada na heap junto com a instância de `Calculator`.

## Como garantir que o objeto sempre será alocado na stack?

No C# 7.2 foi introduzido o tipo `ref struct` que garante que sempre será alocado na stack e nunca escapará para a heap.
Para isso há uma série de restrições, como por exemplo, não é permitido seu uso como uma propriedade de um tipo que possa ser alocado na heap.

### Na prática

O seguinte código não compila:

```csharp
    public ref struct MyRefStruct { }

    public class AnyClass
    {
        MyRefStruct _myRefStruct;
    }
```
E gera a seguinte mensagem:
> Field or auto-implemented property cannot be of type 'MyRefStruct' unless it is an instance member of a ref struct.


## TL;DR

Variáveis de tipo valor são alocadas na stack somente quando são declaradas dentro de um métodos.
Quando um objecto possui uma propriedade de tipo valor ela é alocada na heap juntamente com seu objeto-pai.