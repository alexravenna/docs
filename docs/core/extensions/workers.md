---
title: Worker Services
description: Learn how to implement a custom IHostedService and use existing implementations in C#. Discover various worker implementations, templates, and service patterns.
author: IEvangelist
ms.author: dapine
ms.date: 05/28/2025
ms.topic: overview
---

# Worker services in .NET

There are numerous reasons for creating long-running services such as:

- Processing CPU-intensive data.
- Queuing work items in the background.
- Performing a time-based operation on a schedule.

Background service processing usually doesn't involve a user interface (UI), but UIs can be built around them. In the early days with .NET Framework, Windows developers could create Windows Services for these purposes. Now with .NET, you can use the <xref:Microsoft.Extensions.Hosting.BackgroundService>, which is an implementation of <xref:Microsoft.Extensions.Hosting.IHostedService>, or implement your own.

With .NET, you're no longer restricted to Windows. You can develop cross-platform background services. Hosted services are logging, configuration, and dependency injection (DI) ready. They're a part of the extensions suite of libraries, meaning they're fundamental to all .NET workloads that work with the [generic host](generic-host.md).

[!INCLUDE [worker-template-workloads](includes/worker-template-workloads.md)]

## Terminology

Many terms are mistakenly used synonymously. This section defines some of these terms to make their intent in this article more apparent.

- **Background Service**: The <xref:Microsoft.Extensions.Hosting.BackgroundService> type.
- **Hosted Service**: Implementations of <xref:Microsoft.Extensions.Hosting.IHostedService>, or the <xref:Microsoft.Extensions.Hosting.IHostedService> itself.
- **Long-running Service:** Any service that runs continuously.
- **Windows Service**: The *Windows Service* infrastructure, originally .NET Framework-centric but now accessible via .NET.
- **Worker Service**: The *Worker Service* template.

## Worker Service template

The Worker Service template is available in the .NET CLI and Visual Studio. For more information, see [.NET CLI, `dotnet new worker` - template](../tools/dotnet-new-sdk-templates.md#web-others). The template consists of a `Program` and `Worker` class.

:::code language="csharp" source="snippets/workers/background-service/Program.cs":::

The preceding `Program` class:

- Creates a <xref:Microsoft.Extensions.Hosting.HostApplicationBuilder>.
- Calls <xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionHostedServiceExtensions.AddHostedService%2A> to register the `Worker` as a hosted service.
- Builds an <xref:Microsoft.Extensions.Hosting.IHost> from the builder.
- Calls `Run` on the `host` instance, which runs the app.

### Template defaults

The Worker template doesn't enable server garbage collection (GC) by default, as there are numerous factors that play a role in determining its necessity. All of the scenarios that require long-running services should consider performance implications of this default. To enable server GC, add the `ServerGarbageCollection` node to the project file:

```xml
<PropertyGroup>
    <ServerGarbageCollection>true</ServerGarbageCollection>
</PropertyGroup>
```

_**Tradeoffs and considerations**_

| Enabled | Disabled |
|--|--|
| Efficient memory management: Automatically reclaims unused memory to prevent memory leaks and optimize resource usage. | Improved real-time performance: Avoids potential pauses or interruptions caused by garbage collection in latency-sensitive applications. |
| Long-term stability: Helps maintain stable performance in long-running services by managing memory over extended periods. | Resource efficiency: May conserve CPU and memory resources in resource-constrained environments. |
| Reduced maintenance: Minimizes the need for manual memory management, simplifying maintenance. | Manual memory control: Provides fine-grained control over memory for specialized applications. |
| Predictable behavior: Contributes to consistent and predictable application behavior. | Suitable for Short-lived processes: Minimizes the overhead of garbage collection for short-lived or ephemeral processes. |

For more information regarding performance considerations, see [Server GC](../../standard/garbage-collection/workstation-server-gc.md#server-gc). For more information on configuring server GC, see [Server GC configuration examples](../runtime-config/garbage-collector.md#workstation-vs-server).

### Worker class

As for the `Worker`, the template provides a simple implementation.

:::code language="csharp" source="snippets/workers/background-service/Worker.cs":::

The preceding `Worker` class is a subclass of <xref:Microsoft.Extensions.Hosting.BackgroundService>, which implements <xref:Microsoft.Extensions.Hosting.IHostedService>. The <xref:Microsoft.Extensions.Hosting.BackgroundService> is an `abstract class` and requires the subclass to implement <xref:Microsoft.Extensions.Hosting.BackgroundService.ExecuteAsync(System.Threading.CancellationToken)?displayProperty=nameWithType>. In the template implementation, the `ExecuteAsync` loops once per second, logging the current date and time until the process is signaled to cancel.

### The project file

The Worker template relies on the following project file `Sdk`:

```xml
<Project Sdk="Microsoft.NET.Sdk.Worker">
```

For more information, see [.NET project SDKs](../project-sdk/overview.md).

### NuGet package

An app based on the Worker template uses the `Microsoft.NET.Sdk.Worker` SDK and has an explicit package reference to the [Microsoft.Extensions.Hosting](https://www.nuget.org/packages/Microsoft.Extensions.Hosting) package.

### Containers and cloud adaptability

With most modern .NET workloads, containers are a viable option. When creating a long-running service from the Worker template in Visual Studio, you can opt in to **Docker support**. Doing so creates a *Dockerfile* that containerizes your .NET app. A [*Dockerfile*](https://docs.docker.com/engine/reference/builder) is a set of instructions to build an image. For .NET apps, the *Dockerfile* usually sits in the root of the directory next to a solution file.

:::code language="dockerfile" source="snippets/workers/background-service/Dockerfile":::

The preceding *Dockerfile* steps include:

- Setting the base image from `mcr.microsoft.com/dotnet/runtime:8.0` as the alias `base`.
- Changing the working directory to */app*.
- Setting the `build` alias from the `mcr.microsoft.com/dotnet/sdk:8.0` image.
- Changing the working directory to */src*.
- Copying the contents and publishing the .NET app:
  - The app is published using the [`dotnet publish`](../tools/dotnet-publish.md) command.
- Relayering the .NET SDK image from `mcr.microsoft.com/dotnet/runtime:8.0` (the `base` alias).
- Copying the published build output from the */publish*.
- Defining the entry point, which delegates to [`dotnet App.BackgroundService.dll`](../tools/dotnet.md).

> [!TIP]
> The MCR in `mcr.microsoft.com` stands for "Microsoft Container Registry", and is Microsoft's syndicated container catalog from the official Docker hub. The [Microsoft syndicates container catalog](https://azure.microsoft.com/blog/microsoft-syndicates-container-catalog) article contains additional details.

When you target Docker as a deployment strategy for your .NET Worker Service, there are a few considerations in the project file:

:::code language="xml" source="snippets/workers/background-service/App.WorkerService.csproj" highlight="8,13":::

In the preceding project file, the `<DockerDefaultTargetOS>` element specifies `Linux` as its target. To target Windows containers, use `Windows` instead. The [`Microsoft.VisualStudio.Azure.Containers.Tools.Targets` NuGet package](https://www.nuget.org/packages/Microsoft.VisualStudio.Azure.Containers.Tools.Targets) is automatically added as a package reference when **Docker support** is selected from the template.

For more information on Docker with .NET, see [Tutorial: Containerize a .NET app](../docker/build-container.md). For more information on deploying to Azure, see [Tutorial: Deploy a Worker Service to Azure](cloud-service.md).

> [!IMPORTANT]
> If you want to leverage *User Secrets* with the Worker template, you'd have to explicitly reference the `Microsoft.Extensions.Configuration.UserSecrets` NuGet package.

## Hosted Service extensibility

The <xref:Microsoft.Extensions.Hosting.IHostedService> interface defines two methods:

- <xref:Microsoft.Extensions.Hosting.IHostedService.StartAsync(System.Threading.CancellationToken)?displayProperty=nameWithType>
- <xref:Microsoft.Extensions.Hosting.IHostedService.StopAsync(System.Threading.CancellationToken)?displayProperty=nameWithType>

These two methods serve as *lifecycle* methods - they're called during host start and stop events respectively.

> [!NOTE]
> When overriding either <xref:Microsoft.Extensions.Hosting.BackgroundService.StartAsync%2A> or <xref:Microsoft.Extensions.Hosting.BackgroundService.StopAsync%2A> methods, you must call and `await` the `base` class method to ensure the service starts and/or shuts down properly.

> [!IMPORTANT]
> The interface serves as a generic-type parameter constraint on the <xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionHostedServiceExtensions.AddHostedService%60%601(Microsoft.Extensions.DependencyInjection.IServiceCollection)> extension method, meaning only implementations are permitted. You're free to use the provided <xref:Microsoft.Extensions.Hosting.BackgroundService> with a subclass, or implement your own entirely.

## Signal completion

In most common scenarios, you don't need to explicitly signal the completion of a hosted service. When the host starts the services, they're designed to run until the host is stopped. In some scenarios, however, you might need to signal the completion of the entire host application when the service completes. To signal the completion, consider the following `Worker` class:

:::code source="snippets/workers/signal-completion-service/App.SignalCompletionService/Worker.cs":::

In the preceding code, the <xref:Microsoft.Extensions.Hosting.BackgroundService.ExecuteAsync(System.Threading.CancellationToken)?displayProperty=nameWithType> method doesn't loop, and when it's complete it calls <xref:Microsoft.Extensions.Hosting.IHostApplicationLifetime.StopApplication?displayProperty=nameWithType>.

> [!IMPORTANT]
> This will signal to the host that it should stop, and without this call to `StopApplication` the host will continue to run indefinitely. If you intend to run a short-lived hosted service (run once scenario), and you want to use the Worker template, you must call `StopApplication` to signal the host to stop.

For more information, see:

- [.NET Generic Host: IHostApplicationLifetime](generic-host.md#ihostapplicationlifetime)
- [.NET Generic Host: Host shutdown](generic-host.md#host-shutdown)
- [.NET Generic Host: Hosting shutdown process](generic-host.md#hosting-shutdown-process)

### Alternative approach

For a short-lived app that needs dependency injection, logging, and configuration, use the [.NET Generic Host](generic-host.md) instead of the Worker template. This lets you use these features without the `Worker` class. A simple example of a short-lived app using the generic host might define a project file like the following:

:::code language="xml" source="snippets/hosts/ShortLived.App/ShortLived.App.csproj":::

It's `Program` class might look something like the following:

:::code language="csharp" source="snippets/hosts/ShortLived.App/Program.cs":::

The preceding code creates a `JobRunner` service, which is a custom class that contains the logic for the job to run. The `RunAsync` method is called on the `JobRunner`, and if it completes successfully, the app returns `0`. If an unhandled exception occurs, it logs the error and returns `1`.

In this simple scenario, the `JobRunner` class could look like this:

:::code language="csharp" source="snippets/hosts/ShortLived.App/JobRunner.cs":::

You'd obviously need to add real logic to the `RunAsync` method, but this example demonstrates how to use the generic host for a short-lived app without the need for a `Worker` class, and without the need for explicitly signaling the completion of the host.

## See also

- <xref:Microsoft.Extensions.Hosting.BackgroundService> subclass tutorials:
  - [Create a Queue Service in .NET](queue-service.md)
  - [Use scoped services within a `BackgroundService` in .NET](scoped-service.md)
  - [Create a Windows Service using `BackgroundService` in .NET](windows-service.md)
- Custom <xref:Microsoft.Extensions.Hosting.IHostedService> implementation:
  - [Implement the `IHostedService` interface in .NET](timer-service.md)
