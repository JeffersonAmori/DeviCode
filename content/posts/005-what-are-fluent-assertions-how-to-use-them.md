+++
author = "Jefferson Rocha"
title = "#005 | What are Fluent Validation and how to use them"
date = 2023-02-13T03:32:12Z
description = ""
subtitle = "Usually attributes are used to decouple validation from the model, but there is a better way."
header_img = ""
short = false
toc = true
tags = ["Fluent validation", "csharp", "dotnet"]
categories = []
series = []
comment = true
+++

# Why would anyone use Fluent Validations?

Consider the following `Employee` class:

```csharp
class Employee
{
    public int Id { get; set; }
    public string Name { get; set; }
    public DateTime DateOfBirth {get; set; }
}
```

When we use fluent validation, we are not forced to separate our validation logic into another class, but it enables us to do so.
For example, an app might have a general validation for any given employee, but the Finance module has more rules for the same class.

## Where does one get started?

Run the following to install the FluentValidation package into the project:

```
Install-Package FluentValidation
```

Now we're good to go.

## Creating the model to validate

Let's define the general validation for the general employee. 
The general `Employee` validator checks if the `Id` property is not empty:

```csharp
class GeneralEmployeeValidator : AbstractValidator<Employee>
{
    public GeneralEmployeeValidator()
    {
        RuleFor(x => x.Id).NotEmpty();
    }
}
```

## The default validator

And then the more specific `Finance` validator can build on top of it by inherting `GeneralEmployeeValidator` and calling `:base()`:

```csharp
class FinanceEmployeeValidator : GeneralEmployeeValidator
{
    public FinanceEmployeeValidator() : base()
    {
        RuleFor(x => x.Name).NotEmpty();
    }
}
```

## The Finance validator

With this approach we clearly separate the declarion of the domain (`Employee`) and the validation logic for it (both `GeneralEmployeeValidator` and `FinanceEmployeeValidator`).
Of course there's nothing stopping us from creating a whole new validator. If we want we could an exclusive validator for the `DateOfBirth`:

```csharp
class DateOfBirthEmployeeValidator : AbstractValidator<Employee>
{
    public DateOfBirthEmployeeValidator()
    {
        RuleFor(x => x.DateOfBirth).NotEmpty();
    }
}
```

## Composing a validation chain

And then we can chain then fogether:

```csharp
    // Creating the Employee
    var employee = new Employee() { /* ... */};

    // Creating the validator
    var financeValidator = new FinanceEmployeeValidator();
    var dateOfBirthValidator = new DateOfBirthEmployeeValidator();

    // Validating the Employee using both validators
    financeValidator.ValidateAndThrow(employee);
    dateOfBirthValidator.ValidateAndThrow(employee);
```

# TL;DR

Using `FluentValidation` enables the separation of the declaration of the model (`Employee`) and the validation logic, supporting multiple validations for the same model.