---
layout: post
title: Adding Dependency Injection to ASPNET 5 (RC1U1) Console Applications
tags: [aspnet5]
hidden: true
wip: true
---

As it stands in the RC1 release of *ASPNET 5* and *DNX*, (soon™ to become *ASPNETCORE* and *dotnet CLI* respectively), you are not limited to only creating Web Applications, but also Class Libraries and Console Applications. Having been on the proverbial *bandwagon* for quite some time now, I have several ASPNET5 web applications in production now, and dozens of class libraries. We recently came about the requirement to introduce a command line interface (CLI) to run some background jobs for one of our sites; in this post I'll explain what we did to not only produce a professional CLI, but also to utilise our existing class libraries through dependency injection.

## Brief Background

For those of you that have been following what was *vNext* progress for some time, you may remember a time when you could create a constructor in your console app's `Program.cs`, and the DNX bootstrapper would perform some amount of dependency injection for you. This was later removed in favour of some static properties, such as `PlatformServices.Default.Runtime`.

In case it's not obvious, all of the existing commands available for ASPNET 5 (`dnu`, `dnx`, `dnvm`) are of course CLI's created by the ASPNET team. My goal here is to provide a mechanism to achieve a dependency injected CLI with as little code as possible, so you'll find I've *stolen* as much of the Microsoft practices (and code...) as possible to make life easier.

## Class Libraries

Since discovering the early versions of the Microsoft Dependency Injection packages, I have followed the practice of defining my class libraries with *Service Extensions*. For anyone that has worked with *ASPNET5*, you'll recognise the following from your `Startup` class:

```c#
public void ConfigureServices(IServiceCollection services)
{
  services.AddMvc();
}
```

That `AddMvc` method simply adds dependency injection bindings to the `IServiceCollection`, and is a really useful pattern to follow for your class libraries. To create such a method, in your class library, add `Microsoft.Extensions.DependencyInjection.Abstractions` to your `project.json`, and then in a class called `ServiceExtensions`, add something like the following:

```c#
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.DependencyInjection.Extensions;

public static class ServiceExtensions
{
  public static IServiceCollection AddMyServices(this IServiceCollection services)
  {
    services.TryAddTransient<MySelfBoundClass>();
    services.TryAddTransient<IAnimal, Dog>();
    // etc
    return services;
  }
}

```

Class Libraries written this way can not only be easily incorporated into ASPNET5 websites, but if you follow the advice below, also into Console Applications.

> Whether you use `TryAdd*` or `Add*` is up to you, and I'll assume you know (or can google) the difference between `Transient`, `Singleton`, etc.

## Getting Started

If you take a look at the [DNX CLI](https://github.com/aspnet/dnx/tree/dev/src/Microsoft.Dnx.Tooling), specifically the [`Program` class](https://github.com/aspnet/dnx/blob/dev/src/Microsoft.Dnx.Tooling/Program.cs), you'll find what I believe to be a very easy to understand and follow piece of code. As will become increasingly common in ASPNETCORE, your `Program.cs` is responsible for setting up a _bootstrapper_, which in turn executes your application. In this case, the bootstrapper is the `CommandLineApplication` class.

There are some very obvious methods available on this class (such as `Options`, `Command`, `Argument`, etc), which combined with the example of the DNX application itself, makes it very easy to reuse. The `CommandLineApplication` class itself is actually defined in a NuGet package, which scanning the [`project.json`](https://github.com/aspnet/dnx/blob/dev/src/Microsoft.Dnx.Tooling/project.json#L35-L38) is called `Microsoft.Extensions.CommandLineUtils.Sources` and is a _build dependency_. However, a quick scan of [NuGet.org](https://www.nuget.org/packages?q=Microsoft.Extensions.CommandLineUtils.Sources) quickly reveals that the package has not been published to the official NuGet feeds.

No problem though, the packages for RC1U1 can all be found on one of the [ASPNET Team's MyGet Feeds](https://www.myget.org/gallery/aspnetmaster). You could add this feed to your global NuGet sources [using the command line](https://docs.nuget.org/consume/command-line-reference#sources-command) but I don't recommend it. Instead, I opted to add a [`NuGet.config` file]() to my project, and appended the feed:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageRestore>
    <add key="enabled" value="True" />
    <add key="automatic" value="True" />
  </packageRestore>
  <activePackageSource>
    <add key="aspnetrc1" value="https://www.myget.org/F/aspnetmaster/api/v3/index.json" />
  </activePackageSource>
  <solution>
  <add key="disableSourceControlIntegration" value="true" />
  </solution>
  <packageSources>
    <add key="nuget.org" value="https://api.nuget.org/v3/index.json" protocolVersion="3" />
    <add key="nuget.org" value="https://www.nuget.org/api/v2/" />
    <add key="aspnetrc1" value="https://www.myget.org/F/aspnetmaster/api/v3/index.json" />
  </packageSources>
</configuration>
```

Once the source has been added (and in most cases, close/open the solution too), you will be able to add `Microsoft.Extensions.CommandLineUtils.Sources` to the `project.json` for your Console Application. Don't forget to specify it as a build dependency!

```json
"dependencies": {
  "Microsoft.Extensions.CommandLineUtils.Sources": {
    "version": "1.0.0-rc1-final",
    "type": "build"
  }
}
```
