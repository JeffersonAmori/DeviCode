+++
author = "Jefferson Rocha"
title = "#002 | Os 5 princípios do SOLID"
date = 2023-01-23T08:33:15Z
description = ""
subtitle = "Os princípios do SOLID definem um conjundo de boas práticas para desenvolvimento de código de alta qualidade."
header_img = ""
short = false
toc = true
tags = ["OO", "SOLID", "software enginnering"]
categories = []
series = []
comment = true
+++

# S.O.L.I.D.

Os 5 princípios do SOLID são:

**S** - Single Responsability - Única Responsabilidade.

**O** - Open/Closed Principle - Princípio do aberto/fechado

**L** - Liskov Substitution Principle - Princípio de Substituição de Liskov

**I** - Interface Segregation - Segregação de interfaces

**D** - Depencency Injection - Injeção de Dependência

## 1 | Single Responsability - Única Responsabilidade

O princípio de responsabilidade única determina que cada membro do sistema tenha apenas uma responsabilidade.
Métodos devem ter uma única e clara função - se for preciso utilizar a conjunção *E* no nome do método é um mal sinal.
Preferencialmente os métodos não devem ter efeitos colaterais no sistema.

❌ Não faça isso:
```csharp
double SumThenMultiply(double firstNumber, double secondNumber, double multiplyBy)
{
	return (firstNumber + secondNumber) * multiplyBy;
}
```
✔️ Faça isso:
```csharp
double Sum(double firstNumber, double secondNumber)
{
	return firstNumber + secondNumber;
}

double Multiply(double firstNumber, double secondNumber)
{
	return firstNumber * secondNumber;
}

```


## 2 | Open/Closed Principle - Princípio do aberto/fechado

O *Princípio do aberto/fechado* define que as implementações devem estar *abertas* para extensão enquanto são fechadas para *alterações*.


❌ Não faça isso:
```csharp
enum ValidVideoFileExtension
{
	Mp4 = 0,
	Avi,
	Mkv
}

void ProcessVideoFiles(IEnumerable<FileInfo> files)
{
	foreach(var file in files)
	{
		if(file.Type == ValidVideoFileExtension.Mp4 || 
		   file.Type == ValidVideoFileExtension.Avi || 
		   file.Type == ValidVideoFileExtension.Mkv)
		{
			// Processa o arquivo de vídeo...
		}
	}
}
```
✔️ Faça isso:
```csharp
enum ValidVideoFileExtension
{
	Mp4 = 0,
	Avi,
	Mkv
}

void ProcessVideoFiles(IEnumerable<FileInfo> files)
{
	var allValidVideoFileExtensions = Enum.GetValues(typeof(ValidVideoFileExtension));
	foreach(var file in files)
	{
		if(allValidVideoFileExtensions.Contains(file.Extension))
		{
			// Processa o arquivo de vídeo...
		}
	}
}
```

No primeiro exemplo para adicionar uma nova extensão válida é necessário alterar o `enum` **e** o método `ProcessVideoFiles`.
Isso significa que a implementação do método `ProcessVideoFiles` não está fechada para mudanças.

No segundo apenas adicionar um novo membro ao `enum ValidVideoFileExtension` é o suficiente para que o método `ProcessVideoFiles` o leve em consideração.
Logo a segunda abordagem é *aberta para extensão* enquanto se mantém *fechada para alterações*.

## 3 | Liskov Substitution Principle - Princípio de Substituição de Liskov

O *Princípio de Substituição de Liskov* diz que, ao extender uma classe ou implementar uma interface devemos cumprir com todo o contrato público. 
Isso significa que, por exemplo, não é possível ter métodos semimplementação (olhando pra você `NotImplementedException`).

❌ Não faça isso:
```csharp
class IncompleteFileProvider : IFileProvider
{
    private IDirectoryContents InternalGetDirectoryContents(string subpath) { /* Implementação omitida. */}
    private IFileInfo InternalGetFileInfo(string subpath) { /* Implementação omitida. */}
    private IChangeToken InternalWatch(string filter) { /* Implementação omitida. */}

    public IDirectoryContents GetDirectoryContents(string subpath)
    {
        throw new NotImplementedException();
    }

    public IFileInfo GetFileInfo(string subpath)
    {
        ArgumentNullException.ThrowIfNull(subpath, nameof(subpath));

        return InternalGetFileInfo(subpath);
    }

    public IChangeToken Watch(string filter)
    {
        ArgumentNullException.ThrowIfNull(filter, nameof(filter));

        return InternalWatch(filter);
    }
}

```
✔️ Faça isso:
```csharp
class CompleteFileProvider : IFileProvider
{
    private IDirectoryContents InternalGetDirectoryContents(string subpath) { /* Implementação omitida. */}
    private IFileInfo InternalGetFileInfo(string subpath) { /* Implementação omitida. */}
    private IChangeToken InternalWatch(string filter) { /* Implementação omitida. */}

    public IDirectoryContents GetDirectoryContents(string subpath)
    {
        ArgumentNullException.ThrowIfNull(subpath, nameof(subpath));

        return InternalGetDirectoryContents(subpath);
    }

    public IFileInfo GetFileInfo(string subpath)
    {
        ArgumentNullException.ThrowIfNull(subpath, nameof(subpath));

        return InternalGetFileInfo(subpath);
    }

    public IChangeToken Watch(string filter)
    {
        ArgumentNullException.ThrowIfNull(filter, nameof(filter));

        return InternalWatch(filter);
    }
}
```

No primeiro exemplo a classe `IncompleteFileProvider` não fornece todas as funcionalidades da interface `IFIleProvider`, logo não pode ser usado em *todas e quaisquer situações* em que é esperado um `IFIleProvider`.

No segundo exemplo `CompleteFileProvider` não viola o **Princípio de Substituição de Liskov** pois implementa todos os métodos de `IFIleProvider`.

## 4 | Interface Segregation - Segregação de interfaces

O princípio da *Segregação de interfaces* define que devemos agrupar responsibilidades sob a mesma interface. 
Isso significa ter mais e menores interfaces.

❌ Não faça isso:
```csharp
interface ICar
{
    double Height { get; }
    double Width { get; }
    double Length { get; }
    int NumberOfDoors { get; }
    int NumberOfCylinders { get; }
    int HorsePower { get; }
    string SuspensionStiffness { get; }
}
```
✔️ Faça isso:
```csharp
interface ICar : IVehicle
{
    int NumberOfDoors { get; }
    IEngine Engine { get; }
    ISuspension Suspension { get; }
}

interface IVehicle
{
    double Height { get; }
    double Width { get; }
    double Length { get; }
}

interface IEngine
{
    int NumberOfCylinders { get; }
    int HorsePower { get; }
}

interface ISuspension
{
    string Stiffness { get; }
}

```

Durante a refatoração:
1. As propriedades `Height`, `Width` e `Length` foram agrupadas em `IVehicle`. Isso faz sentido pois um `IBus` por exemplo também é um `IVehicle`.
2. `NumberOfDoors` se mantém em `ICar` pois neste cenário não faz sentido para todos os veículos - pense `IBike`, `IShip` ou `ITank`.
3. `NumberOfCylinders` e `HorsePower` foram agrupadas em `IEngine`. Isso provê uma interface única para acessar todas as propriedades do motor do veículo.
4. `SuspensionStiffness` foi movido para `ISuspension`. O valor desta interface está em representar melhor o modelo (veículo).

## 5 | Depencency Injection - Injeção de Dependência

*Injeção de Dependência* é transferir a responsabilidade de criação das dependências para o consumidor da classe. 

❌ Não faça isso:
```csharp
class MyBusinessEntity
{
    private IFileProvider _fileProvider;
    private string _rootDirectory = "C:\temp";
    private string _allFilesFilter = "*.*";

    MyBusinessEntity()
    {
        _fileProvider = new PhysicalFileProvider(_rootDirectory);
    }

    MyBusinessMethod()
    {
        var changeToken = _fileProvider.Watch(_allFilesFilter);
        // continua o processamento...
    }
}
```
✔️ Faça isso:
```csharp
class MyBusinessEntity
{
    private IFileProvider _fileProvider;
    private string _allFilesFilter = "*.*";

    MyBusinessEntity(IFileProvider fileProvider)
    {
        _fileProvider = fileProvider;
    }

    MyBusinessMethod()
    {
        var changeToken = _fileProvider.Watch(_allFilesFilter);
        // continua o processamento...
    }

}

```

No primeiro exemplo as dependências de `MyBusinessEntity` são definidas pela própria classe - ela tem controle total da execução

No segundo exemplo as dependência de `MyBusinessEntity` são *injetadas* pelo consumidor da classe,
permitindo que `MyBusinessEntity` seja mais flexível (não está mais acoplado com `PhysicalFileProvider` e não precisa mais se preocupar com o diretório raiz.
Essa abordagem também habilita testes usando mocks que antes seriam impossíveis.

## TL;DR

Os 5 princípios do **SOLID** são ótimos guias de design em orientação a objetos.
Seguir estes princípios, na maioria das vezes, nos leva a um código mais extensível e testável.