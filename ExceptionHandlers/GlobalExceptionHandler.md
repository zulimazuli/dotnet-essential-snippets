# Global Exception Handler

## Description

The global exception handler is a mechanism to handle exceptions that are not caught by the application code. It is a good practice to have a global exception handler to handle uncaught exceptions and log them. This will help in identifying and fixing issues in the application.

## Code Snippet

### Global Exception Handler Implementation

```csharp
internal sealed class GlobalExceptionHandler(ILogger<GlobalExceptionHandler> logger) : IExceptionHandler
{
    public async ValueTask<bool> TryHandleAsync(
        HttpContext httpContext,
        Exception exception,
        CancellationToken cancellationToken)
    {
        logger.LogError(
            exception, "Exception occurred: {Message}", exception.Message);

        var problemDetails = new ProblemDetails
        {
            Status = StatusCodes.Status500InternalServerError,
            Title = "Server error"
        };

        httpContext.Response.StatusCode = problemDetails.Status.Value;

        await httpContext.Response
            .WriteAsJsonAsync(problemDetails, cancellationToken);

        return true;
    }
}
```

### Registering the Global Exception Handler and add ProblemDetails to the response

````csharp
builder.Services.AddExceptionHandler<GlobalExceptionHandler>();
builder.Services.AddProblemDetails();

### Adding the Global Exception Handler to the request pipeline
```csharp
app.UseExceptionHandler();
````

## Best Practices

- Use a global exception handler to handle uncaught exceptions in the application.
- Log the exception details to help in identifying and fixing issues in the application.
- Return a 500 Internal Server Error response to the client when an uncaught exception occurs.
- Use the `ProblemDetails` class to create a consistent error response format.
- Ensure that the global exception handler is registered in the application's middleware pipeline.

## Additional Resources & References

- [Global Error Handling in ASP.NET Core 8 @ Milan Jovanovic](https://www.milanjovanovic.tech/blog/global-error-handling-in-aspnetcore-8)
- [ProblemDetails Class @ Microsoft](https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.problemdetails?view=aspnetcore-8.0)
- [Exception Handling @ Microsoft](https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.diagnostics.iexceptionhandler?view=aspnetcore-8.0)
