![Publish Packages](https://github.com/serverlessworkflow/sdk-net/workflows/Publish%20Packages/badge.svg) [![Gitpod ready-to-code](https://img.shields.io/badge/Gitpod-ready--to--code-blue?logo=gitpod)](https://gitpod.io/#https://github.com/serverlessworkflow/sdk-net)


# Serverless Workflow Specification - .NET SDK

Provides .NET 5.0 API/SPI and Model Validation for the [Serverless Workflow Specification](https://github.com/serverlessworkflow/specification)

With the SDK, you can:

- [x] Read and write workflow JSON and YAML definitions
- [x] Programmatically build workflow definitions
- [x] Validate workflow definitions (both schema and DSL integrity validation)

### Status

| Latest Releases | Conformance to spec version |
| :---: | :---: |
| [0.7.4.4](https://github.com/serverlessworkflow/sdk-net/releases/) | [v0.7](https://github.com/serverlessworkflow/specification/tree/0.6.x) |

### Getting Started

```bash
dotnet nuget add package ServerlessWorkflow.Sdk
```

```csharp
services.AddServerlessWorkflow();
```

### How to use

#### Building workflows programatically

```csharp
var workflow = WorkflowDefinition.Create("MyWorkflow", "MyWorkflow", "1.0")
  .StartsWith("inject", flow => 
      flow.Inject(new { username = "test", password = "123456" }))
  .Then("operation", flow =>
      flow.Execute("fakeApiFunctionCall", action =>
      {
          action.Invoke(function =>
              function.WithName("fakeFunction")
                  .SetOperationUri(new Uri("https://fake.com/swagger.json#fake")))
              .WithArgument("username", "${ .username }")
              .WithArgument("password", "${ .password }");
      })      
          .Execute("fakeEventTrigger", action =>
          {
              action.Consume(e =>
                  e.WithName("fakeEvent")
                      .WithSource(new Uri("https://fakesource.com"))
                      .WithType("fakeType"));
          }))
  .End()
  .Build();
```

#### Reading workflows

```csharp
var reader = WorkflowReader.Create();
using(Stream stream = File.OpenRead("myWorkflow.json"))
{
  var definition = reader.Read(stream, WorkflowDefinitionFormat.Json);
}
```

#### Writing workflows

```csharp
  var writer = WorkflowWriter.Create();
  using(Stream stream = new MemoryStream())
  {
      writer.Write(workflow, stream);
      stream.Flush();
      stream.Position = 0;
      using(StreamReader reader = new StreamReader(stream))
      {
          var yaml = reader.ReadToEnd();
          Console.WriteLine(yaml);
          Console.ReadLine();
      }
  }
```

#### Validating workflows

```csharp
var validator = serviceProvider.GetRequiredService<IValidator<WorkflowDefinition>>();
var validationResult = validator.Validate(myWorkflow);
```
