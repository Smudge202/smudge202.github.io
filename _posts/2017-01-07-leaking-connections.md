---
layout: post
title: Logging - Good Logging Practices to locate Undisposed Objects
hidden: true
wip: true
---

I tweeted a thank you to the guys over at [Seq](https://getseq.net/) recently because _Seq_ was instrumental in finding and eliminating a very difficult issue we came across. But technology choice alone isn't enough, so I've decided to blog about some of the logging practices we tend to follow as a rule, and some specific practices that we used in a particularly complex application we've been developing over the last couple of months.

<blockquote class="twitter-tweet" data-partner="tweetdeck"><p lang="en" dir="ltr">Logging best practices and <a href="https://twitter.com/getseq_net">@getseq_net</a> have been an absolute god-send for tracking down tricky problems. Much love. <a href="https://t.co/6fubnhSnOo">pic.twitter.com/6fubnhSnOo</a></p>&mdash; Tommy Long (@Smudge202) <a href="https://twitter.com/Smudge202/status/816749326642847749">January 4, 2017</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

## Contents

* TOC
{:toc}

## Foreword

This is going to be a lengthy post, and several of the practices shown are circumstantial, subjective, situational, and so forth. Add these practices to your developer toolbelt, and decide for yourself when they best fit your needs.

The system we've been working on is a series of commercial applications subject to standard IP clauses, so I've included as much code as I can in this post. Wherever possible the code is sans abstraction and obfuscation, but I won't be able to provide the applications themselves as samples.

The system has 3 microservice styled applications which communicate over a service bus. The system is far from being complete, but one of the founding requirements was to replace a legacy application. As such, several of the legacy practices were brought forth into the new system with the intent of iteratively improving, and where necessary, replacing existing mechanics. As such, some of the code shown may not be entirely pleasant but, trust me when I say the requirements made it a necessity.

The code is primarily contained within 15 .Net Core libraries targeting the full .Net Framework (4.5.2) due to dependency requirements. 4 of those libraries are what we call _application hosts_, containing no business logic, designed to host a selection of the libraries as a .Net Core Console Application which can be deployed inside [Docker](https://www.docker.com/) containers, as a Windows Service, or in Azure. (This is a practice that our company tends to follow and has a collection of composition helpers for so that the applications can be ran anywhere, leaving deployment considerations to our IT department.)

In addition to the above, we have a considerable amount of heritage code in other solutions which is NuGet packaged and deployed by TFS/Octopus, which is then pulled down by one of our microservices at runtime and installed/executed within separate `AppDomain`s. As you might imagine, with so many moving parts, logging is essential.

## Service (Collection) Extensions

Our company has adopted the `IServiceCollection` extension method approach of building up our composition roots. That is, our _application host_ will contain code similar to the following:

```csharp
using Microsoft.Extensions.DependencyInjection;

public static int Main(string[] args)
{
    var services = new ServiceCollection();
    services.AddServicesFromOurClassLibrary();
    services.BuildServiceProvider()
        .GetRequiredService<SomeService>()
        .Run();
}
```
_NB: Extremely simplified for sake of brevity._

In turn, our class libraries expose their services as follows:

```csharp
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.DependencyInjection.Extensions;

public static IServiceCollection AddServicesFromOurClassLibrary(this IServiceCollection services)
{
    services.TryAddSingleton<Foo, Bar>();
    return services;
}
```

Whilst it's up to you how you manage the composition of your application, this is akin to the practice used by ASP.Net Core and is the assumption throughout the rest of this article.

## Abstractions

First and foremost, let's get our abstractions installed. _All_ of our class libraries depend upon the [`Microsoft.Extensions.Logging.Abstractions`](https://nuget.org/packages/microsoft.extensions.logging.abstractions) NuGet Package. The package is incredibly lightweight as you would expect of any series of abstractions. Whilst there are other abstractions available, some of which we inevitably need to integrate with through façades, I've found this one of the most well adopted logging abstractions.

Once installed, don't forget to add `services.AddLogging()` to your service extensions. After which, any class can rely upon `ILogger<TCategory>` as a dependency, giving us an easy to use Logging Abstraction.

## Sinks (Basic)

With the above in place, your application hosts become responsible for hooking up log outputs/sinks so that you're logs actually go somewhere. I personally recommend the following as a bare minimum:

* [`Microsoft.Extensions.Logging`](https://nuget.org/packages/microsoft.extensions.logging)
* [`Microsoft.Extensions.Logging.Debug`](https://nuget.org/packages/microsoft.extensions.logging.debug)
* [`Microsoft.Extensions.Logging.Console`](https://nuget.org/packages/microsoft.extensions.logging.console)

Personally, I prefer to swap out the `Microsoft.Extensions.Logging.Console` for the following, which will give me coloured console output:

* [`Serilog.Extensions.Logging`](https://nuget.org/packages/serilog.extensions.logging)
* [`Serilog.Sinks.Literate`](https://nuget.org/packages/serilog.sinks.literate)

By adding the `Serilog.Extensions.Logging` package, you open up your application to dozens of high quality outputs via [Serilog Sinks](https://github.com/serilog/serilog/wiki/Provided-Sinks). For example, if you're working on an application that does not have a Console, such as a Windows service, you might consider adding the following:

* [`Serilog.Sinks.RollingFile`](https://nuget.org/packages/serilog.sinks.rollingfile)

We can now adapt the application host code shown above to something akin to the following:

```csharp
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using Serilog;
using Serilog.Events;

public static int Main(string[] args)
{
    var services = new ServiceCollection();
    services.AddServicesFromOurClassLibrary();
    var provider = services.BuildServiceProvider();

    var loggerFactory = provider.GetRequiredService<ILoggerFactory>();
    loggerFactory.AddDebug(LogLevel.Trace);

    var serilogConfiguration = new LoggerConfiguration()
        .MinimumLevel.Is(LogEventLevel.Verbose)
        .WriteTo.RollingFile("my-app-{Date}.log");

    if (Environment.UserInteractive)
        serilogConfiguration.WriteTo.LiterateConsole();

    Log.Logger = serilogConfiguration.CreateLogger();
    AppDomain.CurrentDomain.DomainUnload += (o, e) => Log.CloseAndFlush();
    loggerFactory.AddSerilog();

    provider.GetRequiredService<SomeService>()
        .Run();
}
```

With the updated code, we now get log messages pushed out to Visual Studio output window, a rolling file adjacent the exe, and if available, we will also get nicely coloured logs in a console window.

_NB: The_ `Environment.UserInteractive` _check may not be necessary for you, but we follow this practice to ensure we don't attempt to output to a console when, for example, the application is running as a Windows Service._

## What to log?

Now we know how to output logs, the next and most important question:

> What should I be logging?

There is no simple answer to this. In my opinion, there are 5 key components to useful logging...

### Exceptions

The first and most obvious thing to log is your exceptions. However, simply wrapping everything in a `try` / `catch` > `log` would be pretty wasteful and probably not as useful as you might think. I don't want to dwell on exception management for too long, but I will mention a few things.

For starts, if you are one of those people in the habit of throwing exceptions for non _exceptional_ circumstances such as failed user input validation, you should stop doing this. If you were throwing an exception to produce log output, you now have an alternative. If you were throwing an exception to produce a different execution flow, this is called _exception logic_ and is typically [considered a bad practice](http://softwareengineering.stackexchange.com/a/107727/30301).

Next up, don't be afraid to allow exceptions to bubble up to a location that may have much more context regarding the current operation. Here's a somewhat abstract example that hopefully demonstrates this point:

```csharp
public async Task DisableUser(Guid userId)
{
    try
    {
        var user = await _data.GetUserAsync(userId);
        // continue to disable user
    }
    catch (DatabaseException dex)
    {
        _logger.LogError(0, dex, "Unable to disable user '{UserID}'", userId.ToString('N'));
        throw new DisableUserException(userId, dex);
    }
}
```
In the above snippet, whilst my `_data` implementation _could_ independently log the `DatabaseException`, I've decided the additional context available in the consumer would be better. I can therefore omit logging and even omit the `try`/`catch` in the data class, and output the logs here alongside the information that we were attempting to Disable a User at the time this occurred.

_NB: For those wondering why I haven't simply re-thrown the original exception after logging the output, this is a practice I try to follow. The_ `DatabaseException` _is an implementation detail of the class shown. I don't want consumers of this class to_ `catch (DatabaseException)`_, nor have to_ `catch (Exception)`_. The_ `DisableUserException` _allows the consumer to maintain cohesion and not depend upon this class' dependency._

### Decisions

This is a lot trickier than logging exceptions, because you'll need to decide what decisions are important to you. Everytime you write an `if` or `switch` statement, you probably branch the execution flow. Therefore, each time you find yourself writing an `if`/`switch`, consider whether it would be useful to output a trace message explaining such.

Whilst I believe this is good advice to follow, I can't really provide examples; each case will be unique. I certainly don't believe it necessary to output _every_ decision made, but the odd _Trace_ / _Debug_ / _Verbose_ log entry can be crucial in tracking down problems, especially if the issue arises in other environments that you're not actively debugging.

### State

Don't forget to include useful state information in your log outputs. Yes, you can output the full stack trace for a particular exception, but how much easier would it be to know the ID that was being processed at the time? Which leads nicely onto the subject of _correlation_.

### Correlation

Nowadays, it's common to be working with applications that are in some fashion, able to process concurrent requests. Whilst the scope of requests typically protect developers from concerns about concurrent ongoing requests, the same is not true for logging. For example, consider the following:

Request 1
`----> Logs Foo -------------------------------------> Logs Bar --->`

Request 2
`xxx----> Logs Foo -------------------------------> Logs Bar --->xxx`

Request 3
`xxxxxx----> Logs Foo -------------------------> Logs Bar --->xxxxxx`

We simply cannot predict or rely upon even identical concurrent requests executing in the same order. In the above, although request 3 starts last, it finishes first. Why is this important? Consider your log output:

```
Foo A
Foo B
Foo C
Bar D
Bar E
Bar F
```
Without correlation, we have no idea that `Foo A` corresponds with `Bar F` and `Foo B` with `Bar E`, etc. This is usually very simple to resolve by adding a little bit more context:

```
Request 1 Foo A
Request 2 Foo B
Request 3 Foo C
Request 3 Bar D
Request 2 Bar E
Request 1 Bar F
```

Now it's very clear which `Foo` and `Bar` messages belong together. It may seem like common sense now, but it's all too easy to forget about correlation!

### Performance



## What level to log?

## Practices

This next section will walk through some additional handy tips and tricks, some applicable to all situations whilst others are specific to the examples given. As per the foreword, you need to decide for yourself when these tips apply to your situation.

### Interpolation

One of the most common _mistakes_ I've seen since the release of C#6 is people interpolating all of their strings. Don't do this for logs! When using a proper logging framework, the `string.Format` style signatures will often store the arguments separately or display them differently. For example:

```csharp
var messageType = "Interpolated";
_logger.LogInformation($"This is an '{messageType}' log entry.");

messageType = "Classic";
_logger.LogInformation("This is an '{messageType} log entry.", messageType);
```

Outputs to the Console (via `Serilog.Sinks.Literate`) as follows:

![log output screenshot](https://puu.sh/tkOl7/43576feba2.png)

Notice that the _interpolated_ entry has no highlighting. That's because it's simply considered a part of the text, whereas the _classic_ entry can be coloured, can be stored into queryable database columns, etc. This is because the parameters are available to the logger instead of being _"burned in"_ at runtime, prior to reaching the logger.

### Disposable Tracking

## Sinks (Advanced)

### Seq

