+++
author = "Jefferson Rocha"
title = "#002 | The 5 SOLID principles"
date = 2023-01-23T08:33:15Z
description = ""
subtitle = "The SOLID principles define a set of best practices for developing high-quality code."
header_img = ""
short = false
toc = true
tags = ["OO", "SOLID", "software enginnering"]
categories = []
series = []
comment = true
+++

# S.O.L.I.D.

The 5 principles of SOLID are:

**S** - Single Responsibility - Single Responsibility

**O** - Open/Closed Principle - Open/Closed Principle

**L** - Liskov Substitution Principle - Liskov Substitution Principle

**I** - Interface Segregation - Segregation of interfaces

**D** - Depencency Injection - Dependency Injection

## 1 | Single Responsability

The principle of single responsibility determines that each member of the system has only one responsibility.
Methods must have a single and clear function - it is a bad sign if you have to use the conjunction *And* in the method's name.
Having no side effects on the system outside the scope of its excution is a plus.

❌ Don't
```csharp
double SumThenMultiply(double firstNumber, double secondNumber, double multiplyBy)
{
	return (firstNumber + secondNumber) * multiplyBy;
}
```
✔️ Do
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


## 2 | Open/Closed Principle

The *Open/Closed Principle* states that implementations should be *open* to extension while being *closed* to *changes*.


❌ Don't
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
			// Process the video file.
		}
	}
}
```
✔️ Do
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
			// Process the video file.
		}
	}
}
```
In the first example to add a new valid extension it is necessary to change the `enum` **and** the `ProcessVideoFiles` method.
This means that the implementation of the `ProcessVideoFiles` method is not closed to changes.

In the second just adding a new member to the `ValidVideoFileExtension` `enum` is enough for the `ProcessVideoFiles` method to take it into account.
So the second approach is *open to extension* while remaining *closed to changes*.

## 3 | Liskov Substitution Principle

The *Liskov Substitution Principle* states that when extending a class or implementing an interface we must comply with the entire public contract.
This means that we should't have methods with no implementation whatsoever (looking at you `NotImplementedException`).

❌ Don't:
```csharp
class IncompleteFileProvider : IFileProvider
{
    private IDirectoryContents InternalGetDirectoryContents(string subpath) { /* Implementation omitted. */}
    private IFileInfo InternalGetFileInfo(string subpath) { /* Implementation omitted. */}
    private IChangeToken InternalWatch(string filter) { /* Implementation omitted. */}

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
✔️ Do:
```csharp
class CompleteFileProvider : IFileProvider
{
    private IDirectoryContents InternalGetDirectoryContents(string subpath) { /* Implementation omitted. */}
    private IFileInfo InternalGetFileInfo(string subpath) { /* Implementation omitted. */}
    private IChangeToken InternalWatch(string filter) { /* Implementation omitted. */}

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

In the first example the `IncompleteFileProvider` class does not provide all the functionality of the `IFileProvider` interface, so it cannot be used in *any and all situations* where an `IFileProvider` is expected.

In the second example `CompleteFileProvider` does not violate the **Liskov Substitution Principle** as it implements all methods of `IFileProvider`.

## 4 | Interface Segregation

The *Segregation of interfaces* principle defines that we should group responsibilities under the same interface.
This means having more, smaller interfaces.

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
While refactoring:
1. The `Height`, `Width` and `Length` properties have been grouped into `IVehicle`. This makes sense because an `IBus`, for example, is also an `IVehicle`.
2. `NumberOfDoors` sticks to `ICar` as in this scenario it doesn't make sense for all vehicles - think `IBike`, `IShip` or `ITank`.
3. `NumberOfCylinders` and `HorsePower` have been grouped into `IEngine`. This provides a single interface to access all of the vehicle's engine properties.
4. Moved `SuspensionStiffness` to `ISuspension`. The value of this interface lies in better representing the model (vehicle).

## 5 | Depencency Injection

*Dependency Injection* is transferring responsibility for creating dependencies to the consumer of the class.

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

    void MyBusinessMethod()
    {
        var changeToken = _fileProvider.Watch(_allFilesFilter);
        // Implementation omitted for brevity.
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

    void MyBusinessMethod()
    {
        var changeToken = _fileProvider.Watch(_allFilesFilter);
        // Implementation omitted for brevity.
    }

}

```
On the first example the `MyBusinessEntity` dependencies are defined by the class itself - it has full control of the execution

On the second example the `MyBusinessEntity` dependencies are *injected* by the class consumer,
allowing `MyBusinessEntity` to be more flexible (it is no longer coupled with `PhysicalFileProvider` and no longer need to worry about the root directory).
This approach also enables testing using mocks - which would have not been possible on the first example.

## TL;DR

The 5 principles of **SOLID** are great object-oriented design guides.
Following these principles, most of the time, leads to more extensible and testable code.