# Serilog Structured Logger with AWS Sink

## Description

This section provides an example of how to use Serilog with the AWS sink to log structured data to CloudWatch. Serilog is a popular logging library for .NET applications, and the AWS sink allows you to send log events to Amazon CloudWatch Logs.

## Required NuGet packages

- Serilog.AspNetCore
- Serilog.Settings.Configuration
- Serilog.Exceptions
- AWS.Logger.SeriLog

## Code Snippet

### Serilog Configuration

```csharp
public static class LoggerExtensions
{
    public static void UseSerilogLogger(this IHostBuilder builder)
    {
        builder.UseSerilog((context, provider, config) =>
        {
            config
                .ConfigureReaders(provider, context.Configuration)
                .ConfigureEnrichers()
                .ConfigureCloudWatchLogging(context.Configuration);
        });
    }

    private static LoggerConfiguration ConfigureEnrichers(this LoggerConfiguration config)
    {
        return config
            .Enrich.FromLogContext() // Enrich log events with the current execution context
            .Enrich.WithExceptionDetails(); // Enrich log events with exception details
    }

    private static LoggerConfiguration ConfigureReaders(this LoggerConfiguration config, IServiceProvider provider, Configuration configuration)
    {
        return config
            .ReadFrom.Configuration(configuration) // Read configuration from appsettings.json
            .ReadFrom.Services(provider); // Read configuration from DI
    }

    private static LoggerConfiguration ConfigureCloudWatchLogging(this LoggerConfiguration config, IConfiguration configuration)
    {
        return config.WriteTo.AWSSeriLog(configuration); // Write log events to AWS CloudWatch

        // You can also specify a text formatter to render the log message with properties included
        // return config.WriteTo.AWSSeriLog(configuration, textFormatter: new Serilog.Formatting.Json.JsonFormatter(renderMessage: true));
    }
}
```

### Registering the Logger in Program.cs

```csharp
builder.Host.UseSerilogLogger();
```

### Configuring Serilog in appsettings.json

```json
{
  "Serilog": {
    "LogGroup": "/app-name/log-group",
    "Region": "eu-central-1",
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft": "Warning",
        "System": "Warning",
        "Microsoft.AspNetCore": "Warning"
      }
    }
  }
}
```

## Example Usage

```csharp
class SomeClass : ISomeClass
{
    private readonly ILogger<SomeClass> _logger; // Inject the logger with the class name to include source context

    public SomeClass(ILogger<SomeClass> logger)
    {
        _logger = logger;
    }

    public void DoSomething()
    {
        _logger.LogInformation("User {UserId} logged in", userId);

        _logger.LogError(exception, "An error occurred while processing the request {RequestId}", requestId);

        _logger.LogWarning("Transaction {TransactionId} is pending", transactionId);
    }
}
```

## Best Practices

- Do not use interpolated strings to log events. Use structured logging to log events in a consistent format.
- Enrich log events with context information to help in troubleshooting and analysis.
- Use the AWS sink to send log events to CloudWatch for centralized logging and monitoring.
- Configure the minimum log level to filter out unnecessary log events.
- Use a text formatter to render the log message with properties included.
- Store the log group name and region in the configuration file for easy management (it can be injected from environment variables).
- Use the `LogInformation`, `LogError`, and `LogWarning` methods to log events at different log levels.
- Use the `LogError` method to log exceptions and include the exception details in the log event.

## Additional Resources & References

- [Serilog Documentation](https://serilog.net/)
- [Serlilog for ASP.NET Core @ GitHub](https://github.com/serilog/serilog-aspnetcore)
- [Log Correctly To AWS CloudWatch @ Rahulp Nath](https://www.rahulpnath.com/blog/amazon-cloudwatch-logs-dotnet/)
- [Structured Logging In ASP.NET Core With Serilog @ Milan Jovanovic](https://www.milanjovanovic.tech/blog/structured-logging-in-asp-net-core-with-serilog)
- [Parsing logs and structured logging @ AWS](https://docs.aws.amazon.com/lambda/latest/operatorguide/parse-logs.html)
