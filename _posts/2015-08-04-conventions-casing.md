---
layout: post
title: Casing - Modern C# Standards and Conventions
hidden: true
---

This article is part of a series describing [Modern C# Standards and Conventions](http://blog.devbot.net/standards). A list of the other articles in this series can be found below.

## Contents

* Casing - **You are here**
* [Underscores](http://blog.devbot.net/conventions-underscores)
* [Implicit vs Explicit](http://blog.devbot.net/conventions-implicit)
* [Comments](http://blog.devbot.net/conventions-comments)
* [Exceptions](http://blog.devbot.net/conventions-exceptions)
* [ref & out Keywords](http://blog.devbot.net/conventions-refs)
* [Scoping](http://blog.devbot.net/conventions-scoping)
* [Regions](http://blog.devbot.net/conventions-regions)

_If you feel there is something else that should be included here, please drop a comment below, or better yet [send me a PR](https://github.com/smudge202/smudge202.github.io)._

## Casing Techniques

To get the ball rolling, you need to understand the names we use to describe casing techniques.  

**Camel Casing**, also known as _lower camel_, is when the first letter of the first word is lower case, but all other words will have an upper case first letter (in some strange way, I guess this is like a Camel's humps?). As a simple example:

> _foo**B**ar_

**Pascal Casing** is almost identical to _camel_ except the first letter of the first word is also upper case. You may sometimes see this referred to as _Proper_ and I'm sure a slew of other words to express the same intent. As a simple example:

> _**F**oo**B**ar_

With those two key techniques out of the way, we should easily be able to define which cases in your code should be _camel_, which should be _pascal_, and which case should be _upper_ (all uppercase).

There are some [Microsoft guidelines to casing](https://msdn.microsoft.com/en-us/library/x2dbyw72(v=vs.71).aspx) if you're interested, but here's the gist of it:

### Camel

* Private members
* Method arguments
* Local variables

### Pascal 

* Type names (class, enum, struct)
* Namespace names
* Public, Internal, and Protected members
* Generic Arguments (the declarations, i.e. `Func<TResult>`)

_It's increasingly common to see constants declared in Pascal case, however, upper-case declarations are also ok for `const`_

## Acronyms

So the above describes how to case your **words**, but what do you do about acronyms? As it turns out, it's pretty simple though probably not entirely what you expected:

* For 3 letters or more, treat the Acronym as a normal word (i.e. `var xmlReader = new XmlReader()`)
* For 2 letter acronyms, use all upper, **except** in the case of a well known acronym such as `Id`.

For the most part you should avoid defining Acronyms in your code. There is almost no restriction on member/type name length, so describe your code's intent so that it's easy to read. 

_If you need the code obfuscated, use a tool to do so, don't be a tool_.

## Next

The next article in this series is [Underscores](http://blog.devbot.net/conventions-underscores).
