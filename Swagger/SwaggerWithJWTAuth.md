# Swagger Configuration with JWT Authentication

## Description

Swagger is a popular tool for documenting APIs. It provides a user-friendly interface for testing and exploring the API endpoints.
This snippet provides a basic configuration for Swagger with JWT authentication in an ASP.NET Core application. It includes an extension method for adding Swagger to the application and configuring it with JWT authentication, as well as an operation filter for including security information in the Swagger documentation.

## Required NuGet packages

- Swashbuckle.AspNetCore

## Code Snippet

### Swagger Extensions

This extension class provides methods for adding Swagger to the application and configuring it with JWT authentication.
This class should be placed in a separate file, e.g. `Swagger/SwaggerExtensions.cs` in the API project.

```csharp
public static class SwaggerExtensions
{
    /// <summary>
    /// Adds the default Swagger setup to the application.
    /// </summary>
    /// <param name="app"></param>
    /// <param name="developmentOnly"> Specifies whether to enable the Swagger setup only in development environment. </param>
    /// <returns></returns>
    public static IApplicationBuilder UseSwaggerWithUi(this WebApplication app, bool developmentOnly = true)
    {

        if (developmentOnly && !app.Environment.IsDevelopment())
        {
            return app;
        }

        app.UseSwagger();
        app.UseSwaggerUI(options =>
        {
            // Optionally, set custom values for Swagger UI
            var endpointSection = app.Configuration.GetSection("Swagger").GetSection("Endpoint");
            var endpointName = endpointSection.GetValue<string>("Name") ?? $"{app.Environment.ApplicationName} v1";
            var endpointUrl = endpointSection.GetValue<string>("Url") ?? "/swagger/v1/swagger.json";

            options.SwaggerEndpoint(endpointUrl, endpointName);
        });

        // Optionally, add a redirect from the root of the app to the swagger endpoint
        app.MapGet("/", () => Results.Redirect("/swagger")).ExcludeFromDescription();

        return app;
    }

    public static IServiceCollection AddSwaggerDocumentation(this IServiceCollection services, IConfiguration configuration)
    {
        services.AddEndpointsApiExplorer();
        services.AddSwaggerGen(options =>
        {
            // Add optional document filters
            // e.g. options.DocumentFilter<CamelCaseDocumentFilter>();
            // e.g. options.DocumentFilter<FeatureGateDocumentFilter>();

            // set custom values for Swagger document from app configuration or hardcode them
            var openApiSection = configuration.GetSection("Swagger");
            var documentSection = openApiSection.GetSection("Document");
            var title = documentSection.GetValue<string>("Title") ?? "API";
            var version = documentSection.GetValue<string>("Version") ?? "v1";
            var description = documentSection.GetValue<string>("Description") ?? "API Documentation";

            options.SwaggerDoc(version, new OpenApiInfo
            {
                Title = title,
                Version = version,
                Description = description
            });

            // Add all XML comments to the Swagger document, i.e. controller methods summaries and remarks
            var xmlFiles = Directory.GetFiles(AppContext.BaseDirectory, "*.xml", SearchOption.TopDirectoryOnly).ToList();
            xmlFiles.ForEach(xmlFile => options.IncludeXmlComments(xmlFile));

            // Add security definition for JWT Bearer token
            options.AddBearerSecurityDefinition();

            // Add security definition for OAuth2 (see other snippets for implementation)
            // options.AddOAuthSecurityDefinition(configuration);

            // Add operation filters
            options.OperationFilter<AuthorizationOperationFilter>();
        });

        return services;
    }

    private static SwaggerGenOptions AddBearerSecurityDefinition(this SwaggerGenOptions options)
    {
        options.AddSecurityDefinition(
            "Bearer",
            new OpenApiSecurityScheme
            {
                In = ParameterLocation.Header,
                Description = "Please insert JWT without Bearer into the field",
                Name = "Authorization",
                Type = SecuritySchemeType.Http,
                BearerFormat = "JWT",
                Scheme = "Bearer"
            });

        return options;
    }
}
```

### Adding Swagger to the application

Add Swagger to ServiceCollection and to the pipeline in the `Program.cs` file.

```csharp
builder.Services.AddSwaggerDocumentation(builder.Configuration);

// ... Other services

var app = builder.Build();

// Configure the HTTP request pipeline.

app.UseSwaggerWithUi();

// ... Other middleware
```

### AuthorizationOperationFilter

The class `AuthorizationOperationFilter` applies a filter to the Swagger operations to include security information if the operation does not have the AllowAnonymousAttribute.
Could be placed either in the `Filters` folder or in the `Swagger` folder, depending on the project structure.

```csharp
internal sealed class AuthorizationOperationFilter : IOperationFilter
{
    public void Apply(OpenApiOperation operation, OperationFilterContext context)
    {
        var anonymousAttributes = context.MethodInfo.GetCustomAttributes(true).OfType<AllowAnonymousAttribute>();

        if (anonymousAttributes.Any())
        {
            return;
        }

        var securityRequirement = new OpenApiSecurityRequirement()
        {
            {
                new OpenApiSecurityScheme
                {
                    Reference = new OpenApiReference
                    {
                        Type = ReferenceType.SecurityScheme,
                        Id = "Bearer"
                    },
                    Scheme = "JWT",
                    Name = "Bearer",
                    In = ParameterLocation.Header,
                },
                Array.Empty<string>()
            }
        };
        operation.Security = new List<OpenApiSecurityRequirement> { securityRequirement };

        operation.Responses.TryAdd("401", new OpenApiResponse { Description = "Unauthorized" });
        operation.Responses.TryAdd("403", new OpenApiResponse { Description = "Forbidden" });
    }
}
```

### Configuration in appsettings.json

Add the following configuration to the `appsettings.json` file to customize the Swagger document and endpoint.

```json
"Swagger": {
    // "Endpoint": { // Optional, if not provided, default values will be used
        // "Name": "Sample.API V1",
        // "Url": "/swagger/v1/swagger.json"
    // },
    "Document": {
        "Title": "API Documentation",
        "Version": "v1",
        "Description": "This is a sample API"
    }
}
```

## Best Practices

- Add summary and remarks to the controller methods to include them in the Swagger documentation.
- Secure the Swagger endpoint in production environments to prevent unauthorized access to the API documentation.

## Additional Resources & References

- [Swashbuckle.AspNetCore @ GitHub](https://github.com/domaindrivendev/Swashbuckle.AspNetCore)
- [Get started with Swashbuckle and ASP.NET Core @ MS](https://learn.microsoft.com/en-us/aspnet/core/tutorials/getting-started-with-swashbuckle?view=aspnetcore-8.0&tabs=visual-studio)
- [eShopOnContainers @ GitHub](https://github.com/dotnet/eShop)
