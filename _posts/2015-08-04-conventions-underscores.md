---
layout: post
title: Underscores - Modern C# Standards and Conventions
hidden: true
---

This article is part of a series describing [Modern C# Standards and Conventions](http://blog.devbot.net/standards). A list of the other articles in this series can be found below.

## Contents

* [Casing](http://blog.devbot.net/conventions-casing)
* Underscores - **You are here**
* [Implicit vs Explicit](http://blog.devbot.net/conventions-implicit)
* [Comments](http://blog.devbot.net/conventions-comments)
* [Exceptions](http://blog.devbot.net/conventions-exceptions)
* [ref & out Keywords](http://blog.devbot.net/conventions-refs)
* [Scoping](http://blog.devbot.net/conventions-scoping)
* [Regions](http://blog.devbot.net/conventions-regions)

_If you feel there is something else that should be added here, please drop a comment below or better yet, [send me a PR](https://github.com/smudge202/smudge202.github.io)_

## Underscores

In my opinion, there are only three legitimate uses for underscores, two of which are a matter of personal preference.

### Private Instance Members

I use an underscore to prefix a backing member (private instance) which removes the need to use the `this` keyword in my constructors. Some people prefer to use the `this` keyword and be completely rid of underscores. Here is an example of both approaches:

```c#
internal sealed class UsingUnderscores
{
  private readonly IFoo _foo;

  public MyClass(IFoo foo)
  {
    _foo = foo;
  }
}

\\ versus

internal sealed class UsingThisKeyword
{
  private readonly IBar bar;

  public MyClass(IBar bar)
  {
    this.bar = bar;
  }
}
```

I don't _dislike_ the second approach, but I do I _prefer_ the former.

### Test Names

It isn't a practice I follow personally, but I have no problem when I come across test names that use underscores to split the words as opposed to using Pascal casing. For example:

```c#
[Fact]
public static void Given_Foo_When_Bar_Then_Does_Stuff()
{ /* ... */ }

// versus

[Fact]
public static void GivenFooWhenBarThenDoesStuff()
{ /* ... */ }
```

Like many of you, I've grown accustomed to reading pascal and camel cased code. The rest of my code follows these casing rules, so I prefer to be consistent and treat tests as some form of _special case_. It is just code after all.

### MVC Partials

I suspect there are a few exceptions to the above rules, but only one such comes to mind right now. That is partial views in ASP.Net MVC framework, which follow the pattern of being prefixed with an underscore to differentiate them from normal views. I suppose it's useful, but I'm sure there are other/better ways. For every rule there seems to be an exception, so let this be the only one.

It drives me crazy when I see underscores used in namespaces, class names, and so forth. 

> **This is DOT net, not UNDERSCORE net**.
