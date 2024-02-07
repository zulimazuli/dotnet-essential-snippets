# MediatR Pipeline Behavior - Logging Behavior

## Description

Logging Behavior helps in identifying issues, monitoring performance, and tracking user behavior. When using MediatR for CQRS, it is important to log requests and responses to understand the flow of data through the system. This section provides an example of how to use a logging behavior with MediatR to log requests and responses.

The code snippet demonstrates how to create a logging behavior that logs the request and response with different log levels: `Information` and `Debug`. This separation of log levels allows you to filter and analyze logs based on the level of detail required and makes sure that sensitive data is not logged at the `Information` level.

## Required NuGet packages

- MediatR
- Microsoft.Extensions.Logging (or any other logging provider, e.g., Serilog)

## Code Snippet

```csharp
public sealed class LoggingBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>
    where TRequest : IBaseRequest
{
    private readonly ILogger<LoggingBehavior<TRequest, TResponse>> _logger;

    public LoggingBehavior(ILogger<LoggingBehavior<TRequest, TResponse>> logger)
    {
        _logger = logger;
    }

    public async Task<TResponse> Handle(TRequest request, RequestHandlerDelegate<TResponse> next, CancellationToken cancellationToken)
    {
        _logger.LogInformation("Handling request: {Name}", typeof(TRequest).Name);

        // Log the request details in debug level. You don't want to log sensitive data in production, i.e., passwords, credit card numbers, etc.
        _logger.LogDebug("Handling {Name} with request: {@Request}", typeof(TRequest).Name, request);

        var response = await next();

        _logger.LogInformation("Handled request: {Name}", typeof(TRequest).Name);
        _logger.LogDebug("Handled {Name} with response: {@Response}", typeof(TRequest).Name, response);

        return response;
    }
```

### Registering the Snippet in the application

```csharp
services.AddMediatR(cfg =>
{
    cfg.RegisterServicesFromAssemblyContaining(typeof(Program));

    cfg.AddOpenBehavior(typeof(LoggingBehavior<,>));
    // Add other behaviors here
});
```

## Best Practices

- Always log both the request and response for better traceability.
- Use the appropriate log level for different types of logs.
- Avoid logging sensitive data such as passwords, credit card numbers, or personally identifiable information (PII). If you need to log parts of the request for debugging or auditing purposes, make sure to exclude or obfuscate sensitive fields.
- Consider implementing a custom logger that can intelligently log requests, recognizing and excluding fields that contain sensitive data.
- Ensure compliance with data protection regulations (like GDPR, CCPA, etc.) when handling and logging sensitive data.

## Additional Resources & References

- [MediatR @ GitHub](https://github.com/jbogard/MediatR)
- [Logging in .NET Core and ASP.NET Core @ Microsoft](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/logging/?view=aspnetcore-8.0)
