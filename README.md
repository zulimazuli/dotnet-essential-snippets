# Dotnet Essential Snippets

.NET Best Practices and Code Snippets Repository

## Introduction

This repository contains a collection of best practices, code snippets, and design patterns for .NET developers. The goal is to provide a comprehensive resource for developers who are looking to improve their skills and learn from the experiences of others. Whether you're starting a new .NET project or looking to improve an existing one, this repository has something for you.

This repository is organized into sections, each of which focuses on a specific topic. Each section contains a README file that provides an overview of the topic, along with code snippets and best practices. To get started, simply click on the link to the section that interests you.

## Table of Contents

- [Global Exception Handler](#global-exception-handler)
- [Serilog Structured Logger with AWS Sink](#serilog-structured-logger-with-aws-sink)
- [MediatR Behaviors - Logging Behavior](#mediatr-behaviours---logging-behaviour)
- [MediatR Behaviors - Validation Behavior with FluentValidation](#mediatr-behaviours---validation-behaviour-with-fluentvalidation)
- [Contributing](#contributing)
- [To be added](#to-be-added)

## Topics

### Global Exception Handler

The global exception handler is a mechanism to handle exceptions that are not caught by the application code. It is a good practice to have a global exception handler to handle uncaught exceptions and log them. This will help in identifying and fixing issues in the application.

Go to the snippet: [Global Exception Handler](ExceptionHandlers/GlobalExceptionHandler.md)

### Serilog Structured Logger with AWS Sink

Logging is an essential part of any application. It helps in identifying issues, monitoring performance, and tracking user behavior. Serilog is a popular logging library for .NET applications. It provides a structured logging approach, which makes it easier to analyze logs and extract useful information.

This section provides an example of how to use Serilog with the AWS sink to log structured logs to AWS CloudWatch.

Go to the snippet: [Serilog Structured Logger with AWS Sink](Logging/SerilogStructuredLoggerWithAwsSink.md)

### MediatR Behaviors - Logging Behavior

Logging Behavior helps in identifying issues, monitoring performance, and tracking user behavior. When using MediatR for CQRS, it is important to log requests and responses to understand the flow of data through the system.

This section provides an example of how to use a logging behavior with MediatR to log requests and responses.

Go to the snippet: [MediatR Behaviors - Logging Behavior](MediatRBehaviours/LoggingBehavior.md)

### MediatR Behaviors - Validation Behavior with FluentValidation

When using CQRS and MediatR, it is a good practice to validate the requests before they are processed by the handlers. This will help in identifying and fixing issues in the application. FluentValidation is a popular library for validating objects in .NET applications. It provides a fluent interface for defining validation rules and supports complex validation scenarios such as conditional validation, cross-property validation, and custom validation logic.

This section provides an example of how to use MediatR with FluentValidation to validate requests before they are processed by the handlers.

Go to the snippet: [MediatR Behaviours - Validation Behaviour with FluentValidation](MediatRBehaviours/ValidationBehaviourWithFluentValidation.md)

## Contributing

We welcome contributions from the community! If you have a best practice, code snippet, or design pattern that you'd like to share, please open a pull request. We only ask that you follow these guidelines:

- **Template**: Use the provided template to structure your contribution. [Snippet Template](snippetTemplate.md)
- **Code Quality**: Ensure that your code is well-documented, follows best practices, and is free of errors.
- **Examples**: Provide examples of how your code can be used in a real-world scenario.
- **Best Practices**: If you're sharing a best practice, explain why it's important and how it can benefit developers.
- **Organization**: Place your code snippet or best practice in the appropriate section of the repository.
- **Documentation**: Update the README file to include your contribution and any relevant information.
- **License**: Ensure that your code is compatible with the MIT License.

## To be added

- Custom Exceptions
- S3 Bucket Integration
- Local Development with Cognito
- Outbox Pattern
- Unit of Work
- Middleware Examples
- Authentication and Authorization
- Performance Monitoring
- Feature Flags
- Containerization and Docker Support
- CI/CD Pipelines
- Database Migration Strategies
- Testing Strategies
- and more...

## License

This repository is licensed under the MIT License. See the [LICENSE](LICENSE) file for more information.

The purpose of this license is to allow the use of the code snippets and best practices in this repository in any project, commercial or non-commercial, without any restrictions. The software is provided "as is", without warranty of any kind, express or implied.
