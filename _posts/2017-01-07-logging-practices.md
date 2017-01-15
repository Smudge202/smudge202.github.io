---
layout: post
title: Logging - Tips and Tricks
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

The code is primarily contained within 15 .Net Core libraries targeting the full .Net Framework (4.5.2) due to dependency requirements. 4 of those libraries are what we call _application hosts_, containing no business logic, designed to host a selection of the libraries as a .Net Core Console Application which can be deployed inside [Docker](https://www.docker.com/) containers, as a Windows Service, or in Azure. (This is a practice that our company tends to follow and has a collection of composition helpers for so that the applications can be ran anywhere, leaving deployment considerations to our IT department.)

In addition to the above, we have a considerable amount of heritage code in other solutions which is NuGet packaged and deployed by TFS/Octopus, which is then pulled down by one of our microservices at runtime and installed/executed within a separate `AppDomain`. As you might imagine, with this many moving parts, logging is essential.

### Service (Collection) Extensions

Our company has adopted the `IServiceCollection` extension method approach of building up our composition roots. That is, our _application host_ will contain code similar to the following:

```csharp
using Microsoft.Extensions.DependencyInjection;

public static int Main(string[] args)
{
    var services = new ServiceCollection()
        .AddServicesFromOurClassLibrary();
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

Whilst it's up to you how you manage the composition of your application, this is based on the practice used by ASP.Net Core and is the assumption throughout the rest of this article.

## Abstractions

First and foremost, let's get our abstractions installed. _All_ of our class libraries depend upon the [`Microsoft.Extensions.Logging.Abstractions`](https://nuget.org/packages/microsoft.extensions.logging.abstractions) NuGet Package. The package is lightweight as you would expect of any series of abstractions. Whilst there are other abstractions available, some of which we inevitably need to integrate with through façades, I've found this one of the most well adopted logging abstractions, especially given the recent flurry of activity surrounding .Net Core.

Once installed, don't forget to add `services.AddLogging()` to your service extensions. After which, any class can rely upon `ILogger<TCategory>` as a dependency, giving us an easy to use logging abstraction.

_NB: This step alone will not **output** any logs._

## Sinks (Basic)

With the above in place, your application hosts become responsible for hooking up log outputs/sinks so that you're logs actually go somewhere. I recommend the following as a bare minimum:

* [`Microsoft.Extensions.Logging`](https://nuget.org/packages/microsoft.extensions.logging)
* [`Microsoft.Extensions.Logging.Debug`](https://nuget.org/packages/microsoft.extensions.logging.debug)
* [`Microsoft.Extensions.Logging.Console`](https://nuget.org/packages/microsoft.extensions.logging.console)

Personally, I prefer to swap out the `Microsoft.Extensions.Logging.Console` for the following, which will give me coloured console output:

* [`Serilog.Extensions.Logging`](https://nuget.org/packages/serilog.extensions.logging)
* [`Serilog.Sinks.Literate`](https://nuget.org/packages/serilog.sinks.literate)

By adding the `Serilog.Extensions.Logging` package, you open up your application to dozens of high quality outputs via [Serilog Sinks](https://github.com/serilog/serilog/wiki/Provided-Sinks). For example, if you're working on an application that does not have a Console, such as a Windows service, you might consider adding:

* [`Serilog.Sinks.RollingFile`](https://nuget.org/packages/serilog.sinks.rollingfile)

We can now adapt the application host code shown above to something like:

```csharp
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using Serilog;
using Serilog.Events;

public static int Main(string[] args)
{
    var services = new ServiceCollection()
        .AddServicesFromOurClassLibrary();
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

With the updated code, we get log messages pushed out to Visual Studio output window, a rolling file adjacent the exe, and if available, we will also get nicely coloured logs in a console window.

_NB: The_ `Environment.UserInteractive` _check may not be necessary for you, but we follow this practice to ensure we don't attempt to output to a console when, for example, the application is running as a Windows Service._

## What to Log

Now we know how to output logs, the next and most important question:

> What should I be logging?

There is no simple answer to this, no rules that should be followed; the subject of logging will always be subjective. In my opinion though, there are 5 key components to useful logging...

### Exceptions

The first and most obvious thing to log is your exceptions. However, simply wrapping everything in a `try` / `catch` > `log` would be pretty wasteful and probably not as useful as you might think. I don't want to dwell on exception management for too long, but I will mention a few things.

For starts, if you are in the habit of throwing exceptions for non _exceptional_ circumstances, such as failed user input validation, you should stop doing this. If you were throwing an exception to produce log output, you now have an alternative. If you were throwing an exception to produce a different execution flow, this is called _exception logic_ and is typically [considered a bad practice](http://softwareengineering.stackexchange.com/a/107727/30301).

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
        _logger.LogError(0, dex, "Unable to disable user '{UserID}'", userId.ToString("N"));
        throw new DisableUserException(userId, dex);
    }
}
```

In the above snippet, whilst my `_data` implementation _could_ independently log the `DatabaseException`, I've decided the additional context available in the consumer would be better. I can therefore omit logging and even omit the `try`/`catch` in the data implementation (not shown here), and output the logs in the data consumer alongside information that we were attempting to disable a user when this exception occurred.

_NB: For those wondering why I haven't simply re-thrown the original exception after logging the output, this is a practice I try to follow. The_ `DatabaseException` _is an **implementation detail** of the class shown. I don't want consumers of this class to_ `catch (DatabaseException)`_, nor have to_ `catch (Exception)`_. The_ `DisableUserException` _or something slightly more generic if preferred, allows the consumer to maintain cohesion and not depend upon this class' dependencies directly._

### Decisions

This is a lot trickier than logging exceptions, because you'll need to decide what decisions are important to you. Everytime you write an `if` or `switch` statement, you probably branch the execution flow. Therefore, each time you find yourself writing an `if`/`switch`, consider whether it would be useful to output a trace message explaining such.

Whilst I believe this is good advice to follow, I can't really provide examples; each case will be unique. I certainly don't believe it necessary to output _every_ decision made, but the odd _Trace_ / _Debug_ / _Verbose_ log entry can be crucial in tracking down problems, especially if the issue arises in other environments that you're not actively debugging.

### State

Don't forget to include useful state information in your log outputs. Yes, you can output the full stack trace for a particular exception, but how much easier would it be in replicating an exception if you also have the Customer ID, Order ID, etc. that was being processed at the time?

In addition to this, if you've followed my advice and added Serilog integration, you also gain access to [Enrichers](https://github.com/serilog/serilog/wiki/Enrichment). For example, I can add the following NuGet packages:

* [`Serilog.Enrichers.Environment`](https://nuget.org/packages/serilog.enrichers.environment)
* [`Serilog.Enrichers.Process`](https://nuget.org/packages/serilog.enrichers.process)
* [`Serilog.Enrichers.Thread`](https://nuget.org/packages/serilog.enrichers.thread)

With these packages installed, I can update our application host to _Enrich_ the the serilog configuration as shown here:

```csharp
var serilogConfiguration = new LoggerConfiguration()
    .Enrich.FromLogContext()
    .Enrich.WithMachineName()
    .Enrich.WithEnvironmentUserName()
    .Enrich.WithProcessId()
    .Enrich.WithThreadId()
    .MinimumLevel.Is(LogEventLevel.Verbose)
    .WriteTo.RollingFile("my-app-{Date}.log");
```

Whilst this enrichment won't be particularly useful to the sinks so far mentioned, it can provide considerable value when combined with the recommendations in the _Advanced Sinks_ section towards the end of this article.

### Correlation

Nowadays, it's common to be working with applications that are, in some fashion, able to process concurrent requests. When we're writing code that handles a web request in an ASP.Net Website for example, we can often just think about that one request and not worry about how the framework might be managing multiple of these requests at the same time. However, this concurrency can have a profound effect on logging:

Request 1
`----> Logs Foo A -------------------------------------> Logs Bar F --->`

Request 2
`-------> Logs Foo B -------------------------------> Logs Bar E --->`

Request 3
`----------> Logs Foo C -------------------------> Logs Bar D --->`

We simply cannot predict or rely upon even identical concurrent requests executing in the same order. In the above, although request 3 starts last, it finishes first. Why is this important? Consider your log output:

```text
Foo A
Foo B
Foo C
Bar D
Bar E
Bar F
```

Without correlation, we have no idea that `Foo A` corresponds with `Bar F` and `Foo B` with `Bar E`, etc. This is usually very simple to resolve by adding a little bit more context:

```text
Request 1 Foo A
Request 2 Foo B
Request 3 Foo C
Request 3 Bar D
Request 2 Bar E
Request 1 Bar F
```

Now it's much more clear which `Foo` and `Bar` messages belong together. It may seem like common sense now, but it's all too easy to forget about correlation!

### Performance

The final subject for consideration are performance metrics. If you're going to do this, I recommend doing so somewhat scarecly. For example, we log the time an overall request takes, which may involve any number of execution flows and thousands of lines of code, so provides only a very high level metric for the overall system performance. I don't recommend using logging for lower level performance monitoring.

For more intricate performance monitoring, consider using [Benchmark.Net](http://benchmarkdotnet.org/) to test smaller pieces of code in isolation, and/or outputting metrics using [ETW](https://blogs.msdn.microsoft.com/vancem/2012/08/13/windows-high-speed-logging-etw-in-c-net-using-system-diagnostics-tracing-eventsource/).

The point being, logging can be useful for highlighting an area that you should more thoroughly investigate, but the output should also be taken with a pinch of salt.

## What Level to Log

There are only a couple pieces of advice I can give you on this subject.

### Log Levels

 The first is to make sure you understand the order of the log levels in the frameworks you choose to use. For example, at one stage, the Microsoft Logging Abstractions used the following levels:

* `Debug`
* `Verbose`
* `Information`
* `Warning`
* `Error`
* `Critical`

However, [this was changed](https://github.com/aspnet/Announcements/issues/124), so the **current** Microsoft logging levels are as follows:

* `Trace`
* `Debug`
* `Information`
* `Warning`
* `Error`
* `Critical`

Whether you agree with the order of these levels or not, it is important to be aware of it. Similarly, the Serilog logging levels are:

* `Verbose`
* `Debug`
* `Information`
* `Warning`
* `Error`
* `Fatal`

As you can see, the updated Microsoft logging levels do map better to the Serilog levels, though they're not identical. These log levels are typically used to _filter_ your log outputs. For example, in local development and test environments, you may choose you want to output _all_ of the logs, so you'll set the Minimum log level in your application hosts to the lowest level available. However, in later stages of your deployment cycle you may choose to reduce the log spam of high throughput environments so that _Production_ for example, may output only `Warning` and above.

Also make sure you're aware what the _default_ levels are of your logging frameworks, i.e. when no minimum level is configured or specified, what level will your framework use by default. For both Microsoft and Serilog, this tends to be `Information`.

### Log Spam

As alluded to in the previous section, _log spam_ can be an issue. When writing new functionality I tend to add a lot of logging at `Trace` and `Debug` levels, knowing that running the code in a production environment would very quickly consume disk space and generally overload my sinks. Whilst testing, I'll then try to decide which of my log entries have proven to be the most useful in either tracking down issues with the code, or in confirming that the code has worked as intended. Anything I deem to be less useful can either be changed to a lower log level, or removed completely if it provides no value.

In contrast, any log messages that prove to be crucial can have their log levels promoted. **Make sure you test your log entries** when you test your code! The last thing you want is to start outputting millions of log entries per second when your code hits production, bringing the application to a standstill or masking real problems!

## Practices

This next section will walk through some additional handy tips and tricks, some applicable to all situations whilst others are specific to the examples given. As per the foreword, you need to decide for yourself when these tips apply to your situation.

### Interpolation

One of the most common _mistakes_ I've seen since the release of C#6 is people interpolating all of their strings. Don't do this for logs! When using a proper logging framework, the `string.Format` style signatures will often store the arguments separately or display them differently. For example:

```csharp
var messageType = "Interpolated";
_logger.LogInformation($"This is an '{messageType}' log entry.");

messageType = "Classic";
_logger.LogInformation("This is an '{messageType}' log entry.", messageType);
```

The above will output to the Console (via `Serilog.Sinks.Literate`) as follows:

![log output screenshot](https://puu.sh/tkXsf/3e945c79ad.png)

Notice that the _interpolated_ entry has no highlighting. That's because it's simply considered a part of the text, whereas the _classic_ entry can be coloured, can be stored into queryable database columns, etc. This is because the parameters are available to the logger instead of being _"burned in"_ at runtime, prior to reaching the logger. If you've ever had to regex through log files to find the lines you want, you'll know exactly what I mean here.

### Disposable Tracking

At last we finally reach the subject that drove this blog post. The application we've been working on was working perfectly in development and lower stages of our deployment cycle. However, when we put the application through load testing it completely fell apart.

Thanks to our application of the above logging practices, not to mention the advanced sink tips below, it was immediately obvious that we were not managing our disposable connections correctly. This particular issue is often very easy to find because most usages of `IDisposable` objects will be in a `using` statement. Given the right Roslyn analysers, you can simply check the Warnings page in VS and find the problem. However, the complexity of our application and the necessity to integrate with legacy code meant that in places we couldn't simply wrap `using` statements around everything. For example, we use the `IServiceScopeFactory` to instantiate parts of the legacy code and rely on the Microsoft Dependency Injection container to track our transient disposables and tear them down for us when we `Dispose` the scope.

So, we needed a smarter way to track our connections, whilst the code that consumed said connections were essentially a black box to us given it could have been anywhere in millions of lines of heritage code.

Thanks to our aforementioned _service collection extensions_ practice, the first step was simply to append a diagnostics connection factory to the `IServiceCollection`:

```csharp
services.AddSingleton<DiagnosticsConnectionFactory>();
services.AddSingleton(provider => provider
    .GetRequiredService<DiagnosticsConnectionFactory>()
    .CreateConnection());
```

The next step was to start logging when a connection was allocated, to what class the connection was allocated to, and of course, when that class released the connection. I've reduced the actual code used which also entailed pooling / throttling for sake of brevity, but the following should give you an idea of how it works:

```csharp
public class DiagnosticsConnectionFactory
{
    // this is the connection provider that was originally
    // used before we added this diagnostics provider.
    private readonly OriginalConnectionProvider _originalProvider;
    private readonly ILogger _logger;

    public DiagnosticsConnectionFactory(
        OriginalConnectionProvider originalProvider,
        ILogger<DiagnosticsConnectionFactory> logger)
    {
        _originalProvider = originalProvider;
        _logger = logger;
    }

    public IDbConnection CreateConnection()
    {
        // the int provided to the StackFrame constructor determines how many frames
        // to skip up the stack. Zero would point to this method, each number greater
        // to further up the stack. We found it most useful to move two frames up the
        // stack because one frame would typically simply point us to the ctor of an
        // entity framework context. What we actually wanted to see was the consumer
        // of said context. Tweak this number or do something more intelligent as
        // you deem fit.
        var stackFrame = new System.Diagnostics.StackFrame(2);
        var allocateeMethod = stackFrame.GetMethod();
        var allocatee = $"{allocateeMethod.DeclaringType?.Name ?? "Unknown"}.{allocateeMethod.Name}";
        var connection = new DiagnosticsDbConnection(
            _originalProvider.CreateConnection, DeallocateConnection, allocatee);
        _logger.LogDebug("Database connection '{ConnectionID}' allocated to '{Allocatee}'. (Stack: '{StackFrame}')",
            connection.Id, connection.Allocatee, stackFrame);
        return connection;
    }

    private void DeallocateConnection(DiagnosticsDbConnection connection)
    {
        connection.DisposeUnderlyingConnection();
        _logger.LogDebug("Database connection '{ConnectionID}' that was allocated to '{Allocatee}' has been released.",
            connection.Id, connection.Allocatee);
    }

    [DebuggerDisplay("Connection {" + nameof(Id) + "}")]
    private class DiagnosticsDbConnection : IDbConnection
    {
        private readonly Lazy<DbConnection> _connection;
        private readonly Action<DiagnosticsDbConnection> _deallocate;

        public string Id { get; }
        public string Allocatee { get; }

        public DiagnosticsDbConnection(
            Func<DbConnection> connection,
            Action<DiagnosticsDbConnection> deallocate,
            string allocatee)
        {
            Id = Guid.NewGuid().ToString("N");
            _connection = new Lazy<DbConnection>(connection);
            _deallocate = deallocate;
            Allocatee = allocatee;
        }

        // this is the crucial element, when the consumer attempts to Dispose
        // the connection, it will actually just pass control back to our
        // connection provider.
        public void Dispose() => _deallocate(this);

        // whether you're pooling or not, you still need the ability to
        // dispose the actual connection.
        internal void DisposeUnderlyingConnection() => _connection.Value.Dispose();

        // the rest of this class contains what is hopefully an
        // obvious pass-through of all IDbConnection members.
        // for example:
        public ConnectionState State => _connection.Value.State;
    }
}
```

Whilst the above isn't a practice that we will follow often, and certainly isn't something I would want to deploy to Production, it very quickly highlighted our connection problems to us when we used this implementation under load.

## Sinks (Advanced)

So, you're armed with pretty much all the advice I can justify putting into this blog post at least. What else is there? So, so much more. If nothing else, before you leave, check out some of the very cool stuff you can do with some slightly more advanced sinks.

### Seq

Despite following [Nicholas Bulmhardt](https://twitter.com/nblumhardt) on twitter for quite some time, I only recently became aware of [_Seq_](https://getseq.net/). I've been waiting for our IT department to setup an [ELK](https://www.elastic.co/webinars/introduction-elk-stack) instance for me, so decided to see if there were any simple alternatives I could get on with myself. Having spent less than a minute installing _Seq_ to my local machine, I went ahead and added the [`Serilog.Sinks.Seq`](https://nuget.org/packages/serilog.sinks.seq) package and the single line of code required to enable it:

```csharp
serilogConfiguration.WriteTo.Seq("http://localhost:5341");
```

I should point out, this blog post isn't in anyway affiliated or sponsored by _Seq_, but I can't praise it enough. It was so simple, and made our logs _so_ accessible. With the above in place, you can jump into your _Seq_ web portal and watch the logs live, filter events, write queries, and so much more. The [documentation](http://docs.getseq.net/docs) is very thorough and easy to follow, the web interface easy to use, and the value added is immense. It does have a cost, but the guys at _Seq_ were very helpful (actually extended our trial period for us whilst we waited for Finance to get the PO sorted).

My only gripe is, whilst the web portal is incredibly RESTful, generating URL's for your queries and filters, the same does not appear to be true when you enable the live update. You'll find yourself enabling the live feed constantly because everytime you expand a log entry or make a change, the page (correctly) disables the live update so the log entry you're working with doesn't scroll off the page. It would be great however to have a query value for this in the URL so that we can, for example, put the logs on an _information radiator_ without having to configure an automatic refresh, or simply bookmark the feed with updates enabled.

One other feature that is definitely worth mentioning and enabling. Follow the guidance [shown here](http://docs.getseq.net/docs/using-serilog#section-dynamic-level-control) to enable _Dynamic Level Control_. Once done, you can control the Minimum Log Levels we discussed earlier, at runtime, which again can prove immensely useful. Being able to temporarily change your Production environment from `Information` to `Debug` log levels while you're trying to track down a difficult to reproduce issue is an awesome piece of functionality to have at your disposal, especially given how very simple it is to set up.

## Summary

As has been mentioned throughout this article, logging practices are mostly opinion and conjecture. I hope people find some of the information here useful, but don't take it for gospel. Look around for other blogs and articles on logging (feel free to drop a comment below if you have a post you think I should link). For example, although it's not mentioned here, our systems also make considerable use of [Application Insights](https://azure.microsoft.com/en-gb/services/application-insights/), [Google Analytics](https://analytics.google.com), and so forth.

Make sure you understand how your log frameworks and tooling work, and be sure to test your logging the same way you would test any piece of functionality you add to your code. Try to separate log output configuration from the code that actually produces log output, usually through use of abstractions. Finally, try not to _re-invent the wheel_ with custom logging. There are simply too many high quality options out there to be wasting time and money _homebrewing_.