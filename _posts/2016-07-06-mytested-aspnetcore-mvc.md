---
layout: post
title: Review of 'My Tested ASP.NET' for ASP.NET Core
hidden: true
wip: true
---

I was recently contacted by [Ivaylo Kenov](https://www.linkedin.com/in/kenov) asking if I could take the time to review his new [fluent testing framework](https://github.com/ivaylokenov/MyTested.AspNetCore.Mvc) for [ASP.NET Core MVC](http://www.asp.net/). I hadn't come across any of Ivaylo's work before, but tend to enjoy evaluating new technologies and frameworks so decided to share my review.

_The sample code referred to below can be found [here](https://github.com/smudge202/my-tested-asp-net-core-review)._

## Pricing

I want to get this out of the way early, but it's worth noting that '[My Tested ASP.NET Core MVC](https://mytestedasp.net/Core/Mvc)', whilst free for a [many uses](https://mytestedasp.net/Core/Mvc#free-usage-modal), does have a [price tag](https://mytestedasp.net/Core/Mvc#pricing). The not-insignificant cost will surely drive expectations ever higher for paying customers, which having conversed with Ivaylo considerably since first drafting this article, I'm sure will be met in terms of support. I do hope though that those with paid packages which include the ability to request priority features and bug fixes do not hamper progress of the library or detract value from it. After all, if a paying customer demands a _bad_ feature which by rights is included in their package, will Ivaylo be able to say _no_, as would normally be the case in well maintained open source projects?

On the flip side, I'm glad to see there does not appear to be a feature difference between paid and non-paid; instead the cost is seemingly for sake of licensing and some additional support. However, all of the alternatives I've personally used and describe below are completely free, so the cost of _'My Tested ASP.NET Core MVC'_ will definitely be held **against** it.

## Overview

Having scanned the very [fancy website](https://mytestedasp.net/) (so fancy in fact, that my poor display driver crashed on a works machine), I get the impression that the _'My Tested *'_ series has been around for a while. I was surprised that I haven't heard of the libraries before, especially after some research (and tactful nudges from Ivaylo since) revealed that the library has been both featured in a [.Net Blog](https://blogs.msdn.microsoft.com/dotnet/2016/06/28/the-week-in-net-6282016/), and more recently added to the [MVC Repository](https://github.com/aspnet/Mvc/issues/4905). Hopefully this goes towards showing some maturity (and excellent marketing) for the product and not the relative obscurity of my own experience. 

Still, I was surprised - and disappointed - to see that the aforementioned pricing model was only used for the new _ASP.NET Core MVC (Preview)_ iteration, not the previous test frameworks.

At a glance, the framework appears to encompass two key areas:

  * Instantiation and execution of the ASP Pipeline (though it's hard to tell exactly how _involved_ this is from the website).
  * Fluent Assertions (not to be mistaken for the well know [Fluent Assertions](http://www.fluentassertions.com/) framework).


## Fluent Assertions

I'll say now that I'm pretty disappointed that the _Fluent Assertions_ aspect of the _'My Tested ASP.NET Core MVC'_ framework wasn't created as an extension of the official [Fluent Assertions Library](https://github.com/dennisdoomen/fluentassertions) (apologies for the ambiguities in that statement). I converse with [Dennis Doomen](https://twitter.com/ddoomen) (author of _Fluent Assertions_) regularly on twitter, through [his blog](http://www.continuousimprover.com/), and every now and then via the [Fluent Assertions Gitter Chat](https://gitter.im/dennisdoomen/FluentAssertions). Both Ivaylo and Dennis are great guys, so I hope they can get their heads together and work something out.

There are of course other _fluent testing libraries_ such as [Shouldly](https://github.com/shouldly/shouldly) whom I don't have any personal contact with. However, whilst I periodically review my personal technology choices, I haven't yet strayed from _Fluent Assertions_ since first discovering it some years ago.

Whatever the reasoning behind the decisions made thus far, I will be comparing the functionality of _'My Tested ASP.NET Core MVC'_ to mechanisms I would use without it, which for me at least, typically involves using the [Fluent Assertions NuGet Package](https://www.nuget.org/packages/fluentassertions). It is worth mentioning (and praise) however, that whilst the library does not have any formal support for other fluent libraries **yet**, it is [in the pipeline](https://github.com/ivaylokenov/MyTested.AspNetCore.Mvc/issues/142).

## ASP.NET Core Pipeline Testing

The second half of the _'My Tested ASP.NET Core MVC'_ library appears to be responsible for running your web site to _some_ extent. This is a key point and one I intend to verify as thoroughly as limited time permits.

For better or for worse, several of the websites I've helped get into production, from _ASP.NET 5 RC1_, _ASP.NET Core RC2_, through to _ASP.Net Core RTM_, utlise slightly _advanced_ features, digging deep into the Middleware and intricacies of MVC (as you may recall from my previous article on [action conflict resolution](blog.devbot.net/action-conflicts/)). To test these features I've always - since becoming aware of it - used the [Microsoft Test Host](https://www.nuget.org/packages/Microsoft.AspNet.TestHost) which has the benefits of not only being simple, but being the tool of choice for the [ASP.NET Team](https://github.com/aspnet).

It's important to note here the difference between _Integration_ testing for which the MS Test Host is intended, and _Unit_ testing. It _can_ be simple to test the code within a Controller method through Unit testing, but when you start involving routing attributes, constraints, model binding, authorization, and any number of additional MVC features, most of which will rely upon MVC features we tend to take for granted, things become much more complicated. Both the approaches I show below are able to account for these additional features, though I've yet to examine just how far _'My Tested ASP.NET Core MVC'_ is able to go considering it does not take the heavy approach of starting a Test Server, instead tying itself directly into the services responsbile for these MVC features.

## Comparison

I'm going to run through a sample scenario for the remainder of this article, and plan to add at least one follow up article with additional comparisons. I'll be testing functionality using both the alternatives I've listed, and _'My Tested ASP.NET Core MVC'_ framework 

_Note: having typed the framework name_ 'My...' _several times now, I can't help but think a catchy and_ much shorter _name wouldn't have gone amiss!._

### Routing Test

I'm going to try to be unbias and as fair as possible in my choice of tests to compare. The first test I picked is a simple routing example taken from the _'My Tested ASP.NET Core MVC'_ website. This was the first test I noticed on the website, however, I've since spoken with Ivaylo and agree that whilst this is a supported scenario, the library is much more conversant with Controller tests. For the time being though, here is the routing test I copied from the website:

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

I expect the above test to compile, but still fail because the default route added by MVC expects the parameter of my action to be called `id` in order for the route to succeed. If we examine the [UseMvcWithDefaultRoute](https://github.com/aspnet/Mvc/blob/2e2784aa3d2a2abead5d3366714ddcc4cd63f14f/src/Microsoft.AspNetCore.Mvc.Core/Builder/MvcApplicationBuilderExtensions.cs#L43-L56) call present in my website's `Startup` class, we can confirm the default route is `{controller=Home}/{action=Index}/{id?}` (note it specifically looks for a parameter by the name of `id`). The test further validates this because it does in fact fail with the following message:

```
Test Name:	MyControllerShouldHaveRouteForActionWithId
Test Source:	<snip>\tests\Example.WebApplication.Tests.MyTestedAspNetCore\MyControllerTests.cs : line 11
Test Outcome:	Failed
Test Duration:	0:00:00.565
Result Message:	Expected route '/My/Action/1' to contain route value with 'dummy' key but such was not found.
```

That's good. And changing the parameter name to `id` allows the test to pass as expected.

The alternate implementation is [somewhat complicated](http://stackoverflow.com/questions/32114999/unit-testing-routing-in-asp-net-core-1-0-ex-mvc-6) because it is not possible to test routes in MVC directly without mocking and instantiating significant amounts of the MVC pipeline. Therefore, the advice is typically simply to run the test as an _integration test_, as follows:

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

I want to say now, I didn't write the above _conventional_ code in any way at all to try and catch out _'My Tested ASP.NET Core MVC'_ or vice versa. It genuinely is the practice we follow at my company to perform most of our tests. Clearly, the practice I've used is much more inline with what we consider _integration_ testing, because the test will literally run the startup defined in my actual Web Site, it will actually _host_ a web server, and then call into it with the specified URI and HTTP Content.

I was very surprised at first to see the result of that test:

```
Test Name:	MyControllerShouldHaveRouteForActionWithId
Test Source:	<snip>\tests\Example.WebApplication.Tests.Conventional\MyControllerTests.cs : line 15
Test Outcome:	Failed
Test Duration:	0:00:00.578
Result Message:	Cannot return null from an action method with a return type of 'Microsoft.AspNetCore.Mvc.IActionResult'.
```

I had accidentally created a controller that could not be used because you're not allowed to return null, as decribed in the message above. What's more interesting than the difference in exceptions here, is that `MyController.Action` is in fact called by the second test; it must be in order for MVC to recognise the fact that I'm returning a null. Which means, the second test has in fact routed to the controller with the `dummy` named parameter, whilst _'My Tested ASP.NET Core MVC'_ did not.

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

> A big thanks to some unnamed friends for pointing out my obvious mistakes here, and especially to Ivaylo for taking the time to try and explain the error in my ways.

Yes, the request does in fact route to the controller action, but because my `dummy` parameter does not match the `id` parameter of the default route, and because the `id` parameter in the default route is optional, MVC has invoked the method with my `dummy` parameter unbound (i.e. `dummy == default(int) == 0`). Well, whilst obvious in hindsight, it certainly wasn't what I first expected. Unless my Controller actually utilises the parameter somehow (echoing it back in the View or passing the parameter to another service) there is in fact no way for me to test the route binding with the Test Host in the manner _'My Tested ASP.NET Core MVC'_ has. Or at least, I can't think of a way.

I could of course delve into the routing mechanisms of MVC, but I assure you, having tinkered in routing, that would be **far** more involved.

## Conclusion

I intend to make one or two additional comparisons in my next article, testing out much more advanced features to ensure _'My Tested ASP.NET Core MVC'_ is able to handle it. But, having spoken with Ivaylo and reviewed the implementation details of the Fluent API provided, I believe I'd be hard pressed to _trip it up_. The library certainly does things that would either be overly tedious and longwinded, or outright impossible using the mechanisms I've grown used to.

Hooking into the relevant MVC pipeline components when testing has numerous advantages (so long as it is done very well!). _'My Tested ASP.NET Core MVC'_ really is a very good library, which serves only to make my conclusion ever more bittersweet.

### Fluent

I don't want to dwell on this aspect given my half-rant above, but how amazing would this library have been if it was a natural extension of _Fluent Assertions_!? I don't know if it's something that either Dennis or Ivaylo share an interest in, but as a consumer, it's certainly something I would greatly appreciate. Hopefully I can nag both of them to get some action on this!

### Performance

Whilst it is not really evident in the test times shown above (which can hardly be considered anything like performance measurements!) _'My Tested ASP.NET Core MVC'_ is much more intrinsically involved in sub-components of the MVC pipeline. I haven't gone as far as confirming such, but I believe it safe to assume that _'My Tested ASP.NET Core MVC'_ will provide a performance boost over the much heavier integration testing I've shown. 

As a _purist_ of pre-optimisation mantra, I tend to entirely ignore performance until it becomes a problem. However, I know it's crucial that we keep test times to a minimum so that running ever-growing suites of them not become a burden. I haven't hit such an obstacle yet, but should I ever do so, _'My Tested ASP.NET Core MVC'_ would certainly make lowering test times much simpler.

### Behaviour-Driven Development

One of the comparisons I intend to make in my next article is that of the conversion of a behaviourally driven test, which are the ones I typically use. Whilst I'm sure the unit level testing demonstrated above is important to many people, my company and I prefer the BDD approach. Our tests tend to reflect that and instead describe the behaviour that is important from a product owner's point of view. Whilst that might entail assertations of the _nitty gritty_ detail defined in the tests shown, it's often much broader.

Whilst I've no doubt that _'My Tested ASP.NET Core MVC'_ could make some of the finer nuances simpler, that's also heavily where the library's emphasis lies. Huge swathes of functionality would be unused by _me and mine_, which makes the cost much harder to burden.

### Cost

We all like to be paid for the hard work we do. With the world of collaboration growing, and resources from online services to NuGet packages increasingly made available, it's easy to forget how much work may have gone into a feature we take for granted. However, with regards to open source software, I've always been a strong believer in writing for one's self. The packages I've published and repositories I've collaborated on have almost always related and contributed to the progress of a project at work, and in rare occaisions where that isn't true, the contributions have been towards projects of my own making, in my own time.

I don't expect monetary reward to ever come about from any of them; in fact if any such project should become even mildly popular I would be much more grateful for the professional acknowledgment implied. The best way for an open source project to drive revenue, in my opinion, is for the industry to become sufficiently enthralled by their experience (and subsequent dependence). Such projects may receive donations from grateful enterprises and individuals alike. Equally pleasant are opportunities like that of [Jon Channon](https://twitter.com/jchannon), [James Humphries](https://twitter.com/yantrio), and [Andreas Håkansson](https://twitter.com/thecodejunkie) who were [sponsored](http://blog.jonathanchannon.com/2016/03/30/vq-communications-funds-coreclr-nancyfx/) to update [NancyFX](http://nancyfx.org/) to _ASP.Net Core_, and I'm sure they are not alone in these sponsorship deals.

The pricing model, as adopted by _'My Tested ASP.NET Core MVC'_ has two problems, in my humble opinion. The first is that there are simply too many well documented alternatives to give it sufficient uptake and prominence. Which, in part, leads to the second problem; those of us who regularly switch between internal commerical projects that would be required to pay for _'My Tested ASP.NET Core MVC'_, and open source projects maintained either in our _"spare"_ time or in addition to our corporate activities, are left in a slightly difficult position.

I would struggle to justify the costs of something so easy to replace or replicate at work, making any expertise gained in out-of-hours activities of much less value. I would need to be aware of **both** sets of practices, and if I know how to do things the _hard_ way, it makes much more sense to practice this out of hours and gain valuable experience which I could not only utilise at work, but better guarantee being a transferable skill should I move on to greener pastures (as opposed to fighting the same cost vs. value of _'My Tested ASP.NET Core MVC'_ again and again).

More than that, the knowledge - albeit basic - of MVC required to form the more conventional tests is yet again well placed expertise in a technology stack I'm clearly utilising.

## Summary

I'm grateful to see the [.Net Core](http://dot.net) and [ASP.Net Core](http://www.asp.net/) ecosystem so quickly grow and thrive. The drive for technologies we all depend upon to move to _Net Standard_ (or compatible, at least) has been something I've tried to help with for well over a year now, and something I hope to continue as time and work permits. Whilst the issues I've described may not make  _'My Tested ASP.NET Core MVC'_ a great fit for myself and my peers (yet), it is certainly something I encourage people to contribute towards and keep an eye out for.

## Thanks

I often seek advice whilst writing these blog articles, for which [Dan Kirkham](https://twitter.com/herecydev) and [Adam Lee](https://twitter.com/incurus) often concede to my coaxing and help out. But a quick thanks to [Jon Channon](https://twitter.com/jchannon) for his help and review of an earlier draft.

I also want to thank Ivaylo personally. Having conversed, I believe this article will be considered much more the _challenge_ I intended to provide, which he will no doubt overcome, than any form of discouragement. Furthermore, despite an almost scathing first draft of this article, Ivaylo was graceful enough to point out the inaccuracies that would have been a source of embarrassment for me!
