---
layout: post
title: ref & out Keyword - Modern C# Standards and Conventions
hidden: true
---

This article is part of a series describing [Modern C# Standards and Conventions](http://blog.devbot.net/standards). A list of the other articles in this series can be found below.

## Contents

* [Casing](http://blog.devbot.net/conventions-casing)
* [Underscores](http://blog.devbot.net/conventions-underscores)
* [Implicit vs Explicit](http://blog.devbot.net/conventions-implicit)
* [Comments](http://blog.devbot.net/conventions-comments)
* [Exceptions](http://blog.devbot.net/conventions-exceptions)
* ref & out Keywords - **You are here**
* [Scoping](http://blog.devbot.net/conventions-scoping)
* [Regions](http://blog.devbot.net/conventions-regions)

_If you feel there is something else that should be included here, please drop a comment below, or better yet [send me a PR](https://github.com/smudge202/smudge202.github.io)._

## `ref` Keyword

There is an awful lot of confusion on this subject, whether that be incorrectly relating the `ref` keyword (`ByVal`/`ByRef` in VB) to value vs. reference types, or the actual behaviour the `ref` keyword provides. As such, I want to start by clarifying what this keyword does, test your understanding of it's usage, and the provide some tips on when it should, and should not be used.

### Behaviour

The `ref` keyword is used to pass an argument by reference, as opposed to by value. This means that changes made to the parameter in the _called_ method are reflected in the _calling_ method. This can be done regardless of whether the parameter is a value or reference type, so the two are related only by similar naming.

### Understanding

Here is a really simple test showing the `ref` keyword in use:

```c#
private static void TestCase()
{
	var a = new List<string> { "Foo" };
	var b = a;
	PassByRef(ref b);

	a.ForEach(Console.WriteLine);
	b.ForEach(Console.WriteLine);
}

private static void PassByRef(ref List<string> c)
{
	c.Add("Bar");
	c = new List<string> { "Cheesecake!" };
}
```

The question is, what do you think is output to the Console? Have a guess, and then throw the code into Visual Studio and see for yourself. If it is not what you expect, you most definitely **should not be using the `ref` keyword**.

Even if you do, bear in mind that far fewer people, for better or for worse, do not understand the repercussions of using `ref`. Do you really **need** to use `ref`, or are you bastardising code because you can't be bothered to write it properly?

### When to use

Almost never. You may be forced to use this keyword if you are working across native boundaries or with a poorly written dependency, but otherwise I recommend avoiding it.

Do **not** use ref because your method needs to return more than one result! Return an object that contain members for each result.

## `out` keyword

In many respects similar to the `ref` keyword, the `out` keyword allows a method to _return_ a result as a method argument as opposed to a return type. Unlike the `ref` keyword, any value assigned to an `out` parameter by the _calling_ method is not available to the _called_ method.

Also like the `ref` keyword, I recommend **against** using them. They are common place in .Net when you look at various `TryParse` methods, but personally I would have preferred a construct be used (similar to `Nullable<T>`, if not `Nullable<T>` itself) which is returned in place of `bool` + `out T`.

We can't change .Net, and the uproar that replacing the `TryParse` methods would ensue means they're here to stay. But it doesn't mean we need to continue making their mistakes.

For example:

```c#
public static Nullable<int> TryParse(string s)
{
  int result;
  if (int.TryParse(s, out result))
    return result;
  return null;
}

private void Consumer(string userInput)
{
  // to demonstrate usage of the above method
  var parsedInput = TryParse(userInput);
  if (parsedInput.HasValue)
    actUpon(parsedInput.Value);
}
```

## Next

The next article in this series is [Scoping](http://blog.devbot.net/conventions-scoping).
