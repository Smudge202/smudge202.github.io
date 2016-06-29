---
layout: post
title: Review of My Tested ASP.NET for ASP.NET Core
hidden: true
wip: true
---

I was recently contacted by [Ivaylo Kenov](https://www.linkedin.com/in/kenov) asking if I could take the time to review his new [fluent testing framework](https://github.com/ivaylokenov/MyTested.AspNetCore.Mvc) for [ASP.NET Core MVC](http://www.asp.net/). The timing of the request was a little unfortunate with the [test library NuGet package](https://www.nuget.org/packages/MyTested.AspNetCore.Mvc) still targetting ASP.NET Core RC2 a couple of days after the [1.0 release](https://blogs.msdn.microsoft.com/webdev/2016/06/27/announcing-asp-net-core-1-0/). However, one could hardly be expected to update libraries so vehemently, so I thought I'd take a look anyway and compare the functionality to that of alternatives I've used.

## Pricing

I want to get this out of the way early, but it's worth noting that [My Tested ASP.NET Core MVC](https://mytestedasp.net/Core/Mvc), whilst free for personal and open source projects, does have [costs outside of those usages](https://mytestedasp.net/Core/Mvc#pricing). All of the alternatives I've personally used and describe below are completely free, so the cost of _My Tested ASP.NET Core MVC_ will definitely be held **against** it.

## Overview

Having scanned the very [fancy website](https://mytestedasp.net/) (so fancy in fact that my poor display driver crashed on a works machine), I get the impression that the _My Tested *_ series has been around for a while. I was surprised that I haven't heard of the libraries before, but hopefully it goes towards showing some maturity for the product. I was also surprised to see that the aforementioned pricing model was not used for the previous test frameworks; only the new _ASP.NET Core MVC (Preview)_ iteration.

At a glance, the framework appears to encompass two key areas:

  * Instantiation and Execution of the ASP Pipeline (though it's hard to tell exactly how _involved_ this is).
  * Fluent Assertions (not to be mistaken for the well know [Fluent Assertions](http://www.fluentassertions.com/) framework).


## Fluent Assertions

I'll say now that I'm pretty disappointed that the _Fluent Assertions_ aspect of the _My Tested ASP.NET Core MVC_ framework wasn't created as an extension of the official [Fluent Assertions Library](https://github.com/dennisdoomen/fluentassertions) (apologies for the ambiguities in that statement). I converse with [Dennis Doomen](https://twitter.com/ddoomen) (author of _Fluent Assertions_) regularly on twitter, through [his blog](http://www.continuousimprover.com/), and every now and then via the [Fluent Assertions Gitter Chat](https://gitter.im/dennisdoomen/FluentAssertions). Dennis has always been very helpful and very open to extending _Fluent Assertions_; it may be that Dennis and Ivaylo spoke with one another at some stage and agreed that the division be for the best, but I'd be surprised if that were the case.

There are of course other _fluent testing libraries_ such as [Shouldly](https://github.com/shouldly/shouldly) though, whilst I periodically review my personally technology choices, I haven't yet strayed from _Fluent Assertions_ since first discovering it some years ago.

Whatever the reasoning behind the decisions made, I will be comparing the functionality of _My Tested ASP.NET Core MVC_ to mechanisms I would use without it, which for me at least, typically involves using the [Fluent Assertions NuGet Package](https://www.nuget.org/packages/fluentassertions).

## ASP.NET Core Pipeline Testing

The second half of the _My Tested ASP.NET Core MVC_ library appears to be responsible for running your web site to _some_ extent. This is a key point and one I intend to verify as thoroughly as limited time permits.

For better or for worse, several of the _ASP.NET 5 RC1_ and _ASP.NET Core RC2_ websites I've helped get into production utlise somewhat _advanced_ features, digging deep into the Middleware and intricacies of MVC. To test these features I've always (since becoming aware of it) used the [Microsoft Test Host](https://www.nuget.org/packages/Microsoft.AspNet.TestHost) which has the benefits of not only being simple, but of being the tool of choice for the [ASP.NET Team](https://github.com/aspnet).

It's important to note here the difference between _Integration_ testing (for which the MS Test Host is intended) and _Unit_ testing, which I'll discuss separately below. 
