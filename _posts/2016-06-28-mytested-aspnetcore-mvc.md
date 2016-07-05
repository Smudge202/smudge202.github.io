---
layout: post
title: Review of 'My Tested ASP.NET' for ASP.NET Core
hidden: true
wip: true
---

I was recently contacted by [Ivaylo Kenov](https://www.linkedin.com/in/kenov) asking if I could take the time to review his new [fluent testing framework](https://github.com/ivaylokenov/MyTested.AspNetCore.Mvc) for [ASP.NET Core MVC](http://www.asp.net/). The timing of the request was a little unfortunate with the [test library NuGet package](https://www.nuget.org/packages/MyTested.AspNetCore.Mvc) still targetting ASP.NET Core RC2 a couple of days after the [1.0 release](https://blogs.msdn.microsoft.com/webdev/2016/06/27/announcing-asp-net-core-1-0/). However, one could hardly be expected to update libraries so vehemently, so I thought I'd take a look anyway and compare the functionality to that of alternatives I've used.

## Pricing

I want to get this out of the way early, but it's worth noting that ['My Tested ASP.NET Core MVC'](https://mytestedasp.net/Core/Mvc), whilst free for personal and open source projects, does have [costs outside of those usages](https://mytestedasp.net/Core/Mvc#pricing). All of the alternatives I've personally used and describe below are completely free, so the cost of _'My Tested ASP.NET Core MVC'_ will definitely be held **against** it.

## Overview

Having scanned the very [fancy website](https://mytestedasp.net/) (so fancy in fact that my poor display driver crashed on a works machine), I get the impression that the _'My Tested *'_ series has been around for a while. I was surprised that I haven't heard of the libraries before, but hopefully it goes towards showing some maturity for the product and not the relative obscurity of my own experience. I was also surprised to see that the aforementioned pricing model was not used for the previous test frameworks; only the new _ASP.NET Core MVC (Preview)_ iteration.

At a glance, the framework appears to encompass two key areas:

  * Instantiation and Execution of the ASP Pipeline (though it's hard to tell exactly how _involved_ this is from the website).
  * Fluent Assertions (not to be mistaken for the well know [Fluent Assertions](http://www.fluentassertions.com/) framework).


## Fluent Assertions

I'll say now that I'm pretty disappointed that the _Fluent Assertions_ aspect of the _'My Tested ASP.NET Core MVC'_ framework wasn't created as an extension of the official [Fluent Assertions Library](https://github.com/dennisdoomen/fluentassertions) (apologies for the ambiguities in that statement). I converse with [Dennis Doomen](https://twitter.com/ddoomen) (author of _Fluent Assertions_) regularly on twitter, through [his blog](http://www.continuousimprover.com/), and every now and then via the [Fluent Assertions Gitter Chat](https://gitter.im/dennisdoomen/FluentAssertions). Dennis has always been very helpful and very open to extending _Fluent Assertions_; it may be that Dennis and Ivaylo spoke with one another at some stage and agreed that the division be for the best, but I'd be surprised if that were the case.

There are of course other _fluent testing libraries_ such as [Shouldly](https://github.com/shouldly/shouldly) though, whilst I periodically review my personally technology choices, I haven't yet strayed from _Fluent Assertions_ since first discovering it some years ago.

Whatever the reasoning behind the decisions made, I will be comparing the functionality of _'My Tested ASP.NET Core MVC'_ to mechanisms I would use without it, which for me at least, typically involves using the [Fluent Assertions NuGet Package](https://www.nuget.org/packages/fluentassertions).

## ASP.NET Core Pipeline Testing

The second half of the _'My Tested ASP.NET Core MVC'_ library appears to be responsible for running your web site to _some_ extent. This is a key point and one I intend to verify as thoroughly as limited time permits.

For better or for worse, several of the _ASP.NET 5 RC1_ and _ASP.NET Core RC2_ websites I've helped get into production utlise somewhat _advanced_ features, digging deep into the Middleware and intricacies of MVC. To test these features I've always (since becoming aware of it) used the [Microsoft Test Host](https://www.nuget.org/packages/Microsoft.AspNet.TestHost) which has the benefits of not only being simple, but being the tool of choice for the [ASP.NET Team](https://github.com/aspnet).

It's important to note here the difference between _Integration_ testing (for which the MS Test Host is intended) and _Unit_ testing, which I'll discuss separately below.

## Comparison

I'm going to run through a series of sample scenarios, testing functionality using both the alternatives I've listed, and _'My Tested ASP.NET Core MVC'_ framework (having typed the framework name _'My...'_ several times now, I can't help but think a catchy and _much shorter_ name wouldn't have gone amiss!).

### Routing Test

I'm going to try and be unbias and as far as possible in my choice of tests to compare. The first test is a simple routing example taken from the _'My Tested ASP.NET Core MVC'_ website:

```c#
[Fact]
public void MyControllerShouldHaveRouteForActionWithId()
{
	MyMvc
	.Routes()
	.ShouldMap(request => request
		.WithMethod(HttpMethod.Post)
		.WithPath("/My/Action/1"))
	.To<MyController>(c => c.Action(1));
}
```
I took the test as-is, and added the code required to make the test compile:

```c#
public class MyController
{
	public IActionResult Action(int dummy) => null;
}
```

I expect the above test to compile, but still fail because the default route added by MVC expects the parameter of my action to be called `id` in order for the route to succeed. The test does in fact fail with the following message:

```
Test Name:	MyControllerShouldHaveRouteForActionWithId
Test Source:	<snip>\tests\Example.WebApplication.Tests.MyTestedAspNetCore\MyControllerTests.cs : line 11
Test Outcome:	Failed
Test Duration:	0:00:00.565
Result Message:	Expected route '/My/Action/1' to contain route value with 'dummy' key but such was not found.
```

That's good. And changing the parameter name to `id` allows the test to pass as expected.

The alternate implementation is [somewhat complicated]() because it is not possible to test routes in MVC directly without mocking and instantiating significant amounts of the MVC pipeline. Therefore, the advice is typically simply to run the test as an _integration test_, as follows:

```c#
[Fact]
public async Task MyControllerShouldHaveRouteForActionWithId()
{
	var uri = "/My/Action/1";
	var builder = new WebHostBuilder().UseStartup(typeof(Startup));
	using (var client = new TestServer(builder).CreateClient())
	{
		var response = await client.PostAsync(uri, null);
		response.IsSuccessStatusCode.Should().BeTrue(
			because: $"{nameof(MyController)}.{nameof(MyController.Action)} should be reachable on '{uri}'.");
	}
}
```

Whilst I suspect there are other ways to achieve the desired result such as directly testing MVC's routing components, if not for my experience with _ASP.NET Core_ already, the last test would probably have been a bit of a challenge to put together, despite the many examples available. What seemed so simple using _'My Tested ASP.NET Core MVC'_, is clearly a little more involved, and considerably less readable in the form I've provided above.

_In actual fact, I utilised a number of helper methods I've built up over the last few months, so the test, when I first created it, only contained 3 lines of much more readable code; but I added the relevant helper method code to the test to show it fairly._

I want to say now, I didn't write the above _conventional_ code in any way at all to try and catch out _'My Tested ASP.NET Core MVC'_ or vice versa. It genuinely is the practice we follow at my company to perform most of our tests. Clearly, the practice I've used is much more inline with what we consider _integration_ testing, because the test will literally run the startup defined in my actual Web Site, it will actually host a web server, and then call into it with the specified URI and HTTP Content.

In light of the result of that test, having reset the Action parameter back to `dummy` to allow the test to fail, I can only assume that the integration level testing is _not_ what occurs in _'My Tested ASP.NET Core MVC'_:

```
Test Name:	MyControllerShouldHaveRouteForActionWithId
Test Source:	<snip>\tests\Example.WebApplication.Tests.Conventional\MyControllerTests.cs : line 15
Test Outcome:	Failed
Test Duration:	0:00:00.578
Result Message:	Cannot return null from an action method with a return type of 'Microsoft.AspNetCore.Mvc.IActionResult'.
```

Purely (and genuinely) by accident, I managed to create a controller that could not be used. What's more interesting than the difference in exceptions here, is that `MyController.Action` is in fact called by the second test; it must be in order for MVC to recognise the fact that I'm returning a null. Which means, the second test has in fact routed to the controller with the `dummy` named parameter, whilst _'My Tested ASP.NET Core MVC'_ did not.

The most important question then, is which of the two tests are correct?

I changed the controller implementation as follows:

```c#
public class MyController : Controller
{
	public IActionResult Action(int dummy) => View();
}
```

_Note: The parameter name is still `dummy`, however I am no longer returning a `null`._

I then added the View as per normal MVC conventions and ran the actual website with the same URI the tests were using:

![screen shot of website routing the request](https://github.com/smudge202/my-tested-asp-net-core-review/blob/master/imgs/hello-world.png?raw=true)

## Summary

I had actually intended to perform several comparisons, the first _in favour_ of _'My Tested ASP.NET Core MVC'_, or so I assumed when I literally picked the first test example for the website! If anyone is interested, let me know and I'll add additional examples such as more advanced routing, model binding, and so forth which I anticipated _'My Tested ASP.NET Core MVC'_ struggling with.
