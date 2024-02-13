# MediatR Pipeline Behaviour - Validation Behaviour with FluentValidation

## Description

The purpose of the MeediatR Validation Behaviour is to validate the requests before they are processed by the handlers. This is a good practice to validate the requests before processing them. This will help in identifying and fixing issues in the application.

MediatR provides a pipeline behaviour mechanism that allows you to intercept requests and responses as they pass through the pipeline. This can be used to add cross-cutting concerns such as validation, logging, and exception handling to your application.

FluentValidation is a popular library for validating objects in .NET applications. It provides a fluent interface for defining validation rules and supports complex validation scenarios such as conditional validation, cross-property validation, and custom validation logic.

## Required NuGet packages

- MediatR
- FluentValidation
- FluentValidation.DependencyInjectionExtensions

## Code Snippet

### Validation Behaviour

```csharp
public sealed class ValidationBehaviour<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>
     where TRequest : notnull
{
    private readonly IEnumerable<IValidator<TRequest>> _validators;

    public ValidationBehaviour(IEnumerable<IValidator<TRequest>> validators)
    {
        _validators = validators;
    }

    public async Task<TResponse> Handle(TRequest request, RequestHandlerDelegate<TResponse> next, CancellationToken cancellationToken)
    {
        if (!_validators.Any())
        {
            return await next();
        }

        var context = new ValidationContext<TRequest>(request);

        var validationResults = await Task.WhenAll(_validators.Select(v => v.ValidateAsync(context, cancellationToken)));

        var failures = validationResults
            .Where(r => r.Errors.Count != 0)
            .SelectMany(r => r.Errors)
            .Where(f => f != null)
            .ToList();

        if (failures.Count != 0)
        {
            throw new ValidationException(failures);
        }

        return await next();
    }
}
```

### Registering MediatR and the Behaviour in ServiceCollection

```csharp
services.AddMediatR(cfg =>
{
    cfg.RegisterServicesFromAssemblyContaining(typeof(Program));

    cfg.AddOpenBehavior(typeof(ValidatorBehavior<,>));
    // Add other pipeline behaviours here
});

```

### Using the Behaviour in MediatR

#### Command model

```csharp
public record CreateUserCommand(string Username, string Email) : IRequest<User>;
```

#### Validator for the Command

```csharp
public class CreateUserCommandValidator : AbstractValidator<CreateUserCommand>
{
    public CreateUserCommandValidator()
    {
        RuleFor(x => x.Username)
        .NotEmpty()
        .WithMessage("Username cannot be empty");
    RuleFor(x => x.Email)
        .NotEmpty()
        .WithMessage("Email address cannot be empty");
    RuleFor(x => x.Email)
        .EmailAddress()
        .WithMessage("Please specify a valid email address");
    }
}
```

#### Registering Validators in ServiceCollection

```csharp
services.AddValidatorsFromAssembly(Assembly.GetExecutingAssembly());
```

#### Example Usage

```csharp
var request = new CreateUserCommand("Username1", "username@email.com");
var response = await _mediator.Send(request);
```

## Best Practices

- Always validate the requests before processing them.
- Use FluentValidation for complex validation scenarios.
- Use the MediatR pipeline for validation to keep the validation logic separate from the business logic.
- To handle a validation exceptions in the application, you can use a global exception handler, middleware or filter.
- If you don't want to throw an exception, you can return a `Result` object from the validation behaviour instead.

## Additional Resources & References

- [CQRS Validation with MediatR Pipeline and FluentValidation @ Milan Jovanovic](https://www.milanjovanovic.tech/blog/cqrs-validation-with-mediatr-pipeline-and-fluentvalidation)
- [MediatR @ GitHub](https://github.com/jbogard/MediatR)
- [FluentValidation @ GitHub](https://github.com/FluentValidation/FluentValidation)
