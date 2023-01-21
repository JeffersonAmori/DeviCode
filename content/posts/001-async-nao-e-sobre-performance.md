+++
author = "Jefferson Rocha"
title = "#001 - Async não é sobre Performance"
date = "2023-01-16"
description = ""
subtitle = "Quando pensamos em código assíncrono logo nos vem a mente ganhos em performance, mas esse não é exatamente o caso."
header_img = ""
short = false
toc = true
tags = ["csharp", "performance tuning", "async", "dotnet"]
categories = ["csharp", "async", "dotnet"]
series = []
comment = true
+++

## Contexto
Antes de nos aprofundarmos precisamos saber que:

1. Task é a promessa de um valor futuro.
2. Async não é a mesma coisa que paralelismo (multi-thread).
3. Métodos podem ser **CPU-bound** ou **IO-bound**. CPU-bound depende do processador, enquanto IO-bound se refere a discos ou rede.
4. Sempre que o compilador encontra a keyword `await` é criado uma state machine para armazenar o estado do contexto.

## O que acontece por trás das cenas
Quando aguardamos um método com a keyword `await` nós sinalizamos para o runtime que, caso a thread que está aguardando o resultado da operação assíncrona pode retornar para a thread pool e executar outras tarefas.

Porém se o método que está sendo executado for CPU-bound a thread que aguardou o método continuará ocupada com sua execução. Neste caso o resultado é o mesmo que executar o método de forma síncrona, porém com o overhead de criar a state machine.

O benefício de métodos assíncronos aparecem quando temos tarefas IO-bound. Isso significa que tarefas como chamadas HTTP ou leitura de um arquivo em disco, que não dependem do processador, podem ser agendadas para algum momento no futuro, e enquanto isso a thread que está aguardando este método retorna para a thread pool e pode lidar com novas requisições.

## O porque da escalabilidade
`async`/`await` não significa que o código da aplicação vai executar mais rápido, e sim que podemos atender a mais requisições ao mesmo tempo se disponibilizarmos a threads que estão aguardando outros recursos além da CPU.

O que ter em mente ao utilizar `async`/`await`
Não acesse o result diretamente:
```csharp
public int AddOneAsync()
{
    var result = GetValueAsync().Result;
    return result + 1;
}
```

Use sempre `await`:
```csharp
public int AddOneAsync()
{
    var result = await GetValueAsync();
    return result + 1;
}
```
Isso previne o bloqueio da thread atual e evita possível deadlocks.

---

Seja deliberado na escolha entre usar `await` ou retornar a `Task`.

Buscando melhorar a performance nós podemos converter nossos métodos que apenas aguardam um outro método
```csharp
public `async` Task<string> GetStringAsyncWrapperAsync(string uri)
{
  return `await` _htttClient.GetStringAsync(uri);
}
```
Para um método que retorna a `Task`:
```csharp
public Task<string> GetAsyncWrapper(string uri)
{
  return _htttClient.GetAsync(uri);
}
```
Isso provê o benefício de não termos que alocar uma nova state machine (o que é marginalmente mais rápido e aloca menos memória) porém também proporciona uma experiência de debug degradada, pois se ocorrer uma exceção dentro de qualquer `Task` sendo retornada a stacktrace aponta para o primeiro método que não usou `await`.