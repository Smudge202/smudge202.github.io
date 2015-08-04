---
layout: post
title: Implicit vs Explicit - Modern C# Standards and Conventions
hidden: true
---

This article is part of a series describing [Modern C# Standards and Conventions](http://blog.devbot.net/standards). A list of the other articles in this series can be found below.

## Contents

* [Casing](http://blog.devbot.net/conventions-casing)
* [Underscores](http://blog.devbot.net/conventions-underscores)
* Implicit vs Explicit - **You are here**
* [Comments](http://blog.devbot.net/conventions-comments)
* [Exceptions](http://blog.devbot.net/conventions-exceptions)
* [ref & out Keywords](http://blog.devbot.net/conventions-refs)
* [Scoping](http://blog.devbot.net/conventions-scoping)
* [Regions](http://blog.devbot.net/conventions-regions)

_If you feel there is something else that should be included here, please drop a comment below, or better yet [send me a PR](https://github.com/smudge202/smudge202.github.io)._

## What does it mean?

In short, an _implicit_ declaration is one in which the type is not defined, instead it is _implied_ (or _inferred_) by the declaration. For example:

```c#
var data = GetResults();
```

In contrast, an _explicit_ declaration defines the type as part of the declaration. For example:

```c#
List<string> data = GetResults();
```

I often come across people that believe the _implicit_ declaration to be in some way related to _dynamics_. This is **completely wrong** and should be beaten out of said people. If you do not know what _dynamics_ are, don't worry - they do not relate to this article.

## Pros and Cons

To be able to compare the two, we should take a look at the pros and cons of each approach, and hopefully in doing so allow you to conclude that _implicit_ declarations are better.

### Readability

I believe this to be the key proponent of using `explicit` declarations. That is, as per the example above, it is more obvious what the type of `data` is. The only real counter to this is that IDE's nowadays are superb at providing and displaying this information. However, let's leave this as a _pro_ for _explicit_ whilst we examine the benefits of _implicit_ declarations.

### Refactoring

There are three key points when it comes to refactoring. Of course, we all follow the [Open/Close Principle](https://en.wikipedia.org/wiki/Open/closed_principle), but every now and then the _real world_ slaps us in the face and forces us to change the signature of existing code. Let's start by expanding upon the example above:

```c#
public void Implicit()
{
  var data = GetData();
  foreach (var item in data)
    Console.WriteLine(item);
}

public void Explicit()
{
  List<string> data = GetData();
  foreach (string item in data)
    Console.WriteLine(item);
}
```

Here we see that both methods fetch some data from some unknown `GetData` method, and then output the contents to the `Console`. As per the readability section above, it is clear in the `Explicit` code that `data` is of type `List<string>`. However, this leads to the assumption that `GetData` returns `List<string>` which may not be the case, leading us to our first point.

#### Casting

In the example above, if I change the return type of `GetData` to something that inherits from `List<string>`, the code will continue to _work_ in both the _implicit_ and _explicit_ cases. But, in the _explicit_ case I have **changed the behaviour**, because it is **casting** the return type.

#### Cascading

What if the `GetData` method were changed to return an `IEnumerable<int>`? The person making this change may be forced to then _cascade_ this change to the `Explicit` method too. By using `var` in the _implicit_ method, I maximise compatibility with unimportant changes in lower level components.

Consider the case in which you are not the author of the `GetData` method, instead it is brought in by NuGet or reference to a third party assembly. Updating versions in the _Explicit_ case could appear to be a nightmare with potentially thousands of annoying cast exceptions, in turn leading to skipping the update.. leading to future updates being skipped? What if that skipped update would have patched a critical security issue? Ok, too many _whatifs_ and assumptions...

#### Manual Verification

Some might argue that being forced to make changes in the higher level components is a good thing because it forces the developer to manually check the code. But in reality, developers will just change the explicit type by hand (or worse, explicitly cast the return type), without consideration to the code utilising those variables.

The only time they'll look at the consumer code, is when the compiler forces them to through compilation errors. Personally, I follow TDD and have trust that my tests will notice breaking changes; manually re-typing or casting my variables is just a waste of time.

## Next

The next article in this series is [Comments](http://blog.devbot.net/conventions-comments).
