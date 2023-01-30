+++
author = "Jefferson Rocha"
title = "Generator-Function in C# - como o yield funciona?"
date = "2023-01-30"
description = ""
header_img = ""
short = false
toc = true
tags = ["csharp", "yield", "IEnumerable", "LINQ", "IQueryable"]
categories = ["csharp", "yield", "IEnumerable", "LINQ", "IQueryable"]
series = []
comment = true
+++

Vamos começar devagar: `IQueryable` estende `IEnumerable`. O que isso significa? 
Bem, o que quer que você queira fazer com `IEnumerable`, você também pode fazer com `IQueryable`. 
Portanto, deve haver algumas diferenças conceituais entre os dois. 
Pequeno aviso: quando falo de `IEnumerable` ou `IQueryable`, refiro-me à variação genérica. 
Porém a maioria do que é dito aqui também se aplica à versão não genérica que foi usada antes do .NET 2.

Em primeiro lugar, podemos ver que ambos vivem em namespaces totalmente diferentes:

* `IEnumerable` - `System.Collections.Generic`
* `IQueryable` - `System.Linq`

Então vemos que há uma grande diferença. `IEnumerable` é usado para coleções e `IQueryable` é usado para consultar dados. 
Mas ambos têm um comportamento comum: elas iteram apenas "para frente" (`forward collections`).
É preciso materializá-los, por exemplo, via `ToList()`.

`IQueryable` fornece principalmente duas novas propriedades:

* `IQueryProvider` - Executa uma expressão contra outra coisa (por exemplo, LINQ to SQL - traduz a expressão para SQL válido)
* `Expressão` - Pode ser uma árvore de expressão inteira (por exemplo, predicados, seletores de valor, ...)

Com esses dois conceitos, você pode construir todo um modelo complexo e traduzi-lo para a conexão subjacente (por exemplo, LINQ to XML / LINQ to SQL, ...). 
`IEnumerable`, simplificando, aceita apenas um `delegate`. Você pode ver isso muito claramente se, por exemplo, observar a definição de `Where`:

* A assinatura da versão `IEnumerable` é: `Where(Func<T, bool> predicate)`
* A assinatura da versão `IQueryable` é: `Where(Expression<Func<T, bool>> predicate)`

## O que acontece se usarmos `IEnumerable` para consultar dados no banco de dados

Considere o seguinte código:

```csharp
IEnumerable<Funcionario> funcionarios = GetFuncionarios();
empregados.Where(e => e.Workload > 50).ToList();
```

E o mesmo exemplo, mas com um `IQuerayable`:

```csharp
IQueryable<Funcionario> funcionarios = GetFuncionarios();
empregados.Where(e => e.Workload > 50).ToList();
```
Vamos supor que `GetFuncionarios` nos forneça um `DbContext` que desce para o SQL-Server.

No primeiro exemplo todos os Funcionarios são carregados para o cliente e depois o predicado é executado neles.

No segundo exemplo, todos os funcionários são filtrados no servidor e apenas os que atendem ao predicado são retornados ao cliente.

Se o seu conjunto de dados for grande o suficiente, isso fará uma grande diferença. Outro exemplo seria Paginação. 
`Skip` e `Take` não funcionarão com `IEnumerable` em conexão com um banco de dados. 
Você recuperaria todos os objetos na memória e executaria `Skip` e `Take` posteriormente (no cliente).

## Resumo
Uma pequena visão geral:

* Ambos `IEnumerable` e `IQueryable` são coleções de encaminhamento - eles não são materializados imediatamente
* Consultar dados do banco de dados `IEnumerable` carregará os dados na memória em um filtro posteriormente no cliente
* A consulta de dados do banco de dados `IQueryable` irá filtrar primeiro e depois enviar os dados filtrados para o cliente
* `IQueryable` é adequado para consultar dados de memória externa
* Pode haver cenários em que o provedor de consulta subjacente não pode traduzir sua expressão para algo significativo, então você deve mudar para `IEnumerable`