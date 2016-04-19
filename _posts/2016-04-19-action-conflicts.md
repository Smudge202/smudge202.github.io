---
layout: post
title: Managing Action Conflicts in ASP.Net 5 with Swashbuckle
wip: true
hidden: true
tags: [aspnet5]
---

For several weeks now we've been using [Swashbuckle](github.com/domaindrivendev/swashbuckle) in our [ASP.Net 5](https://get.asp.net/) Web API to assist document and even test our API endpoints. This is a brand new API, and we've been doing everything we can to follow best practices with regards to Web API's. Our discussions and decisions regarding best practices are always focused upon the practices themselves, and not necessarily upon the technical implications of an outcome. Thanks to the superb extensibility of *ASP.NET 5*, this hasn't been an issue. That is, until today, when we came across an issue that [many others](https://github.com/domaindrivendev/Swashbuckle/issues/142) also seem to have had problems with.

> Request matched multiple actions resulting in ambiguity for actions with different parameters

## The Problem

The problem is pretty simple, and I don't believe to be too uncommon. We have an API endpoint which returns a list of addresses based upon the postcode (or zipcode, if you're so inclined).

`GET /addresses/b323pp`

*NB: I googled for an example UK postcode; the above is apparently the postcode for [Birmingham City Council](https://mapit.mysociety.org/area/2514/example_postcode.html).*

Depending on the address service you're using *under the hood*, this will result in several addresses (5 in my case).

I'm intentionally going to avoid the discussion of what is, or is not best practice at this stage, but suffice to say, we decided that it would be appropriate to be able to call our API with both postcode and a house name or number, with the intent of retrieving fewer, or preferably, a single result. I'm not sure how a postcode differs to zipcodes, but postcodes will often return several results, so a filtering mechanic seems quite reasonanble.

We could have gone about this in several ways, but the decision was to route the following request:

`GET /addresses/b323pp?house-number=144`

To implement this, our `AddressController` looks something like the following:

```c#
[HttpGet]
[Route("addresses/{postcode:postcode}")]
[Produces(typeof(IEnumerable<AddressSearchResult>))]
public async Task<IEnumerable<AddressSearchResult>> GetAddressesByPostcode(string postcode)
  => await _addressLookup.GetAddressesByPostcodeAndHouseNumber(postcode, string.Empty);

[HttpGet]
[Route("addresses/{postcode:postcode}")]
[Produces(typeof(IEnumerable<AddressSearchResult>))]
public async Task<IEnumerable<AddressSearchResult>> GetAddressesByPostcodeAndHouseNumber(string postcode, [FromQuery(Name ="house-number")]string houseNumber)
  => await _addressLookup.GetAddressesByPostcodeAndHouseNumber(postcode, houseNumber);
```

The keen-eyed among you will immediately spot the problem; we have two methods for the same `Route`.

## Fixing Swashbuckle

There are actually two problems with the above controller methods. The first exception encountered was:

> Multiple operations with path 'addresses/{postcode}' and method 'GET'. Are you overloading action methods?"

This is an exception thrown by Swashbuckle, version [6.0.0-rc1-final](http://www.nuget.org/packages/Swashbuckle/6.0.0-rc1-final) at time of writing. It's not immediately obvious, but all the ASP.Net 5 variants of Swashbuckle are not actually present in the main Swashbuckle [GitHub Respository](https://github.com/domaindrivendev/swashbuckle). Instead, you can find the code in the [Ahoy Respository](https://github.com/domaindrivendev/ahoy) also owned by the Swashbuckle author, [Richard Morris](https://twitter.com/domaindrivendev).

The specific line of code that throws the above exception can be [found here](https://github.com/domaindrivendev/Ahoy/blob/6.0.0-rc1-final/src/Swashbuckle.SwaggerGen/SwaggerGen/DefaultSwaggerProvider.cs#L93-L95). I managed to find mention of *Conflict Resolution* in the primary Swashbuckle Repository, but any code present in there had clearly not been ported to *Ahoy* yet.

I changed the linked lines of code to no longer throw an exception:

```c#
ApiDescription apiDescription;
if (group.Count() > 1)                    
  apiDescription = _options.ResolveConflict(group.Select(x => x), httpMethod);
else
  apiDescription = group.Single();
```

...and introduced some new functionality onto the `SwaggerDocumentOptions` class:

```c#
internal Func<IEnumerable<ApiDescription>, string, ApiDescription> ResolveConflict { get; private set; }
  = ThrowExceptionOnApiDescriptionConflict;
	
public void ResolveConflictsBy(Func<IEnumerable<ApiDescription>, string, ApiDescription> resolver)
  => ResolveConflict = resolver ?? ThrowExceptionOnApiDescriptionConflict;
	
private static ApiDescription ThrowExceptionOnApiDescriptionConflict(IEnumerable<ApiDescription> apiDescriptions, string httpMethod)
{
  throw new NotSupportedException(string.Format(
    "Multiple operations with path '{0}' and method '{1}'. Are you overloading action methods?",
    apiDescriptions.First().RelativePathSansQueryString(), httpMethod));
}
```

As you can see in the above snippet, I maintained the original behaviour of throwing a `NotSupportedException` when a conflict is detected, but provided the ability to extend the conflict resolution. I'm sure this could have been done in different ways, and I may well tidy it up later, but this certainly works.

To use the above changes, I modified my `Startup` class as follows:

```c#
// various other services get added
services.ConfigureSwaggerDocument(options =>
{
  options.SingleApiVersion(config.ApiVersion);
  options.ResolveConflictsBy(ApiDescriptionConflictResolver.Resolve);
});
```

The implementation of `ApiDescriptionConflictResolver` is as follows:

```c#
internal static class ApiDescriptionConflictResolver
{
  public static ApiDescription Resolve(IEnumerable<ApiDescription> descriptions, string httpMethod)
  {
    var parameters = descriptions
      .SelectMany(desc => desc.ParameterDescriptions)
      .GroupBy(x => x, (x, xs) => new { IsOptional = xs.Count() == 1, Parameter = x }, ApiParameterDescriptionEqualityComparer.Instance)
      .ToList();
    var description = descriptions.First();
    description.ParameterDescriptions.Clear();
    parameters.ForEach(x =>
    {
      if (x.Parameter.RouteInfo != null)
        x.Parameter.RouteInfo.IsOptional = x.IsOptional;
      description.ParameterDescriptions.Add(x.Parameter);
    });
    return description;
  }
}
```

Let's break down what's happening here, as I imagine with some tweaks it could be very reusable.

The `descriptions` passed into the method, in our case, represent the two methods in our controller, `GetAddressesByPostcode` and `GetAddressesByPostcodeAndHouseNumber`. The linq query shown gets all the parameters from both methods (2x `postcode` parameters and 1x `housenumber` parameter), then determines whether a parameter is *Optional* based on whether the parameter is present in both sets of descriptions.

`ApiParameterDescription` is a reference type, so no two parameters would actually be equal by default. To combat this we use a fairly simple `IEqualityComparer` to determine equality:

```c#
internal sealed class ApiParameterDescriptionEqualityComparer : IEqualityComparer<ApiParameterDescription>
{
  private static readonly Lazy<ApiParameterDescriptionEqualityComparer> _instance
  	= new Lazy<ApiParameterDescriptionEqualityComparer>(() => new ApiParameterDescriptionEqualityComparer());
  public static ApiParameterDescriptionEqualityComparer Instance
  	=> _instance.Value;

  private ApiParameterDescriptionEqualityComparer() { }

  public int GetHashCode(ApiParameterDescription obj)
  {
    unchecked
    {
      var hash = 17;
      hash = hash * 23 + obj.ModelMetadata.GetHashCode();
      hash = hash * 23 + obj.Name.GetHashCode();
      hash = hash * 23 + obj.Source.GetHashCode();
      hash = hash * 23 + obj.Type.GetHashCode();
      return hash;
    }
  }

  public bool Equals(ApiParameterDescription x, ApiParameterDescription y)
  {
    if (!x.ModelMetadata.Equals(y.ModelMetadata)) return false;
    if (!x.Name.Equals(y.Name)) return false;
    if (!x.Source.Equals(y.Source)) return false;
    if (!x.Type.Equals(y.Type)) return false;
    return true;
  }
}
```

This class utilises both Jon Skeet's [Singleton Advice](http://csharpindepth.com/Articles/General/Singleton.aspx), and his implementation of [property based hashcode comparisons](http://stackoverflow.com/a/263416/707618), resulting in two different instances of `ApiParameterDescription` being determined as equal if the `ModelMetadata`, `Name`, `Source`, and `Type` are equal.

*NB: The `RouteInfo` property is intentionally ommitted.*

Ok, with all those pieces of the puzzle in place, navigating to the default swagger URI (`~/swagger/ui`) shows the API signature exactly as intended!

![swagger documentation](https://raw.githubusercontent.com/smudge202/smudge202.github.io/master/images/swagger-conflict.PNG)

Note that only one method is shown, with the `house-number` correctly identified as being both a `query` *parameter type*, and as *optional*. Perfect! But the battle isn't quite over...

*NB: All of the above can be seen on [this pull request](https://github.com/Gilmond/Ahoy/pull/1). I hope to get it into a state that we can PR back to Swashbuckle.*

## MVC Says No

Unfortunately, whilst we've cleared up the method ambiguity for the Swagger documentation, we've done nothing to the same effect for MVC. When I clicked `Try it out!` in the above screenshot, I received the following MVC exception:

> Request matched multiple actions resulting in ambiguity for actions with different parameters

So close, but so far. Fortunately, `ASP.Net 5` is compeltely open source, so it didn't take me long to track down [the method](https://github.com/aspnet/Mvc/blob/6.0.0-rc1/src/Microsoft.AspNet.Mvc.Core/Infrastructure/DefaultActionSelector.cs#L37-L99) that [throws this exception](https://github.com/aspnet/Mvc/blob/6.0.0-rc1/src/Microsoft.AspNet.Mvc.Core/Infrastructure/DefaultActionSelector.cs#L90-97).

However, not only is ASP.Net 5 open source, but it's also incredibly extensible compared to it's predecessors. We can see that [line 72](https://github.com/aspnet/Mvc/blob/6.0.0-rc1/src/Microsoft.AspNet.Mvc.Core/Infrastructure/DefaultActionSelector.cs#L72) calls the virtual [`SelectBestActions`](https://github.com/aspnet/Mvc/blob/6.0.0-rc1/src/Microsoft.AspNet.Mvc.Core/Infrastructure/DefaultActionSelector.cs#L106) method to allow consumers to *interfere* with conflict resolution!

So, I inherited from the `DefaultActionSelector` and added an `override` for the method:

```c#
internal sealed class RoutingActionSelector : DefaultActionSelector
{
  private readonly IHttpContextAccessor _contextAccessor;

  public RoutingActionSelector(
    IActionDescriptorsCollectionProvider actionDescriptorsCollectionProvider,
    IActionSelectorDecisionTreeProvider decisionTreeProvider,
    IEnumerable<IActionConstraintProvider> actionConstraintProviders,
    ILoggerFactory loggerFactory,
    IHttpContextAccessor contextAccessor)
    : base(actionDescriptorsCollectionProvider, decisionTreeProvider, actionConstraintProviders, loggerFactory)
  {
    _contextAccessor = contextAccessor;
  }

  protected override IReadOnlyList<ActionDescriptor> SelectBestActions(IReadOnlyList<ActionDescriptor> actions)
  {
    if (actions.Count <= 1)
      return base.SelectBestActions(actions);

    var queryParametersCount = _contextAccessor.HttpContext.Request.Query.Count;
    return actions
      .Where(x => x.Parameters.Count(p => p.BindingInfo?.BindingSource.Id == "Query") == queryParametersCount)
      .ToList()
      .AsReadOnly();
  }
}
```

As you can see, the `DefaultActionSelector` requires several dependencies, but that's no issue. By specifying the same dependencies on my own class, the new [dependency injection framework](https://github.com/aspnet/dependencyinjection) manages that all for me.

By overriding the default implementation, it's important to be aware that this class will get for every request that makes it into the MVC pipeline, so it was equally important to *short-circuit* the execution as quickly as possible for the cases I don't care about. The first two lines achieve that in a fairly simple way, immediately returning if there's 1 or less `actions` (i.e. no conflict).

Whilst I'm sure there are better ways to match a HTTP request to actions, I went with a nice and simple implementation of only returning actions where the number of expected query parameters matched the number of query parameters in the request. In our case, the `GetAddressByPostcode` action expects no query parameters, whilst `GetAddressByPostcodeAndHouseNumber` expects a single query parameter.

The final piece of the puzzle is to make one more change to your `Startup` file:

```c#
services.AddTransient<IActionSelector, RoutingActionSelector>();
```

This places our implementation of the `IActionSelector` into the list of services available to your application.

It is crucial that you add your service **after** calling `services.AddMvc()`. This is because, when multiple implementations of the same service are added to the service collection, the Microsoft DI Framework will always pick the **Last** implementation added.

With all these little pieces in place, I ran swashbuckle again and *hey, presto!*. We're in business.
