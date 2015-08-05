---
layout: post
title: Regions - Modern C# Standards and Conventions
hidden: true
---

This article is part of a series describing [Modern C# Standards and Conventions](http://blog.devbot.net/standards). A list of the other articles in this series can be found below.

## Contents

* [Casing](http://blog.devbot.net/conventions-casing)
* [Underscores](http://blog.devbot.net/conventions-underscores)
* [Implicit vs Explicit](http://blog.devbot.net/conventions-implicit)
* [Comments](http://blog.devbot.net/conventions-comments)
* [Exceptions](http://blog.devbot.net/conventions-exceptions)
* [ref & out Keywords](http://blog.devbot.net/conventions-refs)
* [Scoping](http://blog.devbot.net/conventions-scoping)
* Regions - **You are here**

_If you feel there is something else that should be included here, please drop a comment below, or better yet [send me a PR](https://github.com/smudge202/smudge202.github.io)._

## Regions

In the same way [comments are not necessary](http://blog.devbot.net/conventions-comments), neither are regions. Albeit for slightly different reasons.

Let's explore the reasons people choose to use regions first, in order to understand how they can be avoided.

### Lost in Code

The first reason people use regions, is to help developers find the location in code they want. As we'll see later, hopefully that isn't too difficult because there shouldn't be too much code in the file. However, to counter this common approach I personally follow a very simple structure in every class I define:

```c#
internal sealed class Class : Abstraction
{
  private readonly object _dependency;
  private object _state = new object();
  
  public Class(object dependency)
  {
    _dependency = dependency;
  }
  
  public void ImplementedMethod()
  {
    _state = ManageStateIfNeccessary();
  }
  
  private object ManageStateIfNecessary()
  {
    return _state ?? new object();
  }
}
```

Clearly, the example above is not intended to accomplish any particular task, simply to show the layout I follow. From top to bottom, my layout is:

* Class Declaration including any inheritance / implementing.
* `private readonly` fields to hold the class' dependencies.
* `private` fields to hold any state my class requires (hopefully none).
* The constructor(s), which should assign my dependencies.
* `public` methods implementing whichever methods I need to in accordance with my inherited/implemented abstraction.
* `private` methods to assist in aforementioned public methods and delegate out small tasks for readability.

I've excluded several member types (constants, statics, nested types, etc) for sake of brevity. But, by following this structure every single time I define a class, finding code is easy. Regions in such a case would only serve to obscure my code, not make it easier.

### Too much responsibility

Regions are also often introduced when a class/file is becoming unwieldy, allowing developers to `Collapse to Defintion` (Ctrl+M, Ctrl+O in Visual Studio), and conversely _expand regions_ or `Stop Outlining` (Ctrl+M, Ctrl+P in Visual Studio). Common approaches involve grouping methods into one region, fields into another, properties into a further region, etc.

Regardless of how you group your code within regions, the chances are you are doing so to hide a [code smell](https://en.wikipedia.org/wiki/Code_smell). **Hiding an issue does not resolve it!**

If your methods are growing to long, the number of properties and fields too high, then your class is responsible for too much! I'd like to think everyone has heard of, and hopefully tries to follow the [Single Responsibility Principle](https://en.wikipedia.org/wiki/Single_responsibility_principle), but in reality I know a large portion of developers lack this understanding, or more commonly, the required knowledge to implement SRP.

## Defining Responsibility

It's all good and well saying a class is only _responsible_ for one thing, but how do you define that responsibility? If the responsibility you define is too broad, you end up in the same old mess of classes becoming bloated. For example:

`This class is responsible for processing an order`

Hopefully it's fairly evident that the statement above is both ambiguous, and broad. Depending on your domain of course, the act of _processing an order_ might be incredibly simple and a great responsibility definition. In most domains I've come across, that isn't the case. 

When we define a responsibility that is too broad, we inevitably take on additional responsibilities. The class becomes responsible for validating user-input, for checking current stock, for creating order records in the database, all of which may fit within the definition of _"processing an order"_.

So how about making the responsibilities granular? What harm can that do? Leading by example:

`This class is responsible for assigning the Quantity field on the Order Line record`

Well that certainly seems to be a much smaller responsibility. However, in defining such a small responsibility, we create a composition nightmare for ourselves whereby the use case "Process an Order" requires hundreds or thousands of classes seamlessly working in conjunction with one another, in order to complete the action.

This is the _tripping point_ I find most people stumble upon when first learning and following the Single Responsibility Principle. They define these tiny pieces of functionality which of course can be re-used, interchanged, extended, and appear to follow the dream. But, implementing anything more than a few simple use cases becomes tedious and exhausting. Finding your way around the solution becomes impossible, and small changes in functionality and new features take vasts amount of time to implement.

The trick is to find a balance.

## Magic

Unfortunately, I don't know of any hard and fast way to define a responsibility; there is no one-size-fits-all. This is one of the situations in which senior developers prove their worth, because it essentially boils down to experience. Both knowledge of the domain, and being well versed in clean code.

Far too many companies seem to lack this experience within their teams (blind leading the blind?). So if you don't have access to a senior, but want to walk the path of clean coding, you're going to have to learn by trial and error. Explore these concepts with your team, ensure they're as enthusiastic as you are, and push forward. 

Yes, you will make some mistakes, but there are plenty of people out there that are willing to help. Learn from your mistakes and consider your design.

## SOS!!

I don't have unlimited free time, but I encourage people to contact me through twitter or github (links below) if you get yourself really stuck, or want to run a design concept by someone like minded. Similar help exists on the web, you just have to hunt it down.

## Next

Unfortunately, you've reached the end of this series. If you've enjoyed it or found any useful information, please do share it with your friends, colleagues, barman, pets, etc. Similarly, I appreciate any feedback, good and bad, via the comments below.

With the basics out of the way, I'm looking to start a new series which tackles some best practices. Not just in theory, but in practice, providing any tips I can on **how to _introduce_ these concepts** to your current company.
