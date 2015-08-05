---
layout: post
title: Scoping - Modern C# Standards and Conventions
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
* Scoping - **You are here**
* [Regions](http://blog.devbot.net/conventions-regions)

_If you feel there is something else that should be included here, please drop a comment below, or better yet [send me a PR](https://github.com/smudge202/smudge202.github.io)._

## Access Modifiers

The access modifiers available in C# are pretty well known, but let's recap quickly. For Types (class, struct, enum), the scopes are:

* `public` - the type is visible to anyone that references your assembly, and from anywhere within your assembly.
* `internal` - the type is visible within your assembly, and to any assemblies matching [InternalsVisibleToAttributes](https://msdn.microsoft.com/en-us/library/system.runtime.compilerservices.internalsvisibletoattribute(v=vs.110).aspx).
* `private` - can only be used when the type is nested, and means the type is only visible within the parent type(s).
* `sealed` - although not considered an access modifier, it does effect scope in that even when the type is visible to others, it cannot be inherited.

Similarly, the scopes for members:

* `public` - the member is visible to anyone with visibility to the declaring type.
* `protected` - the member is visible to anyone inheriting (and thus has visibility of) the declaring type.
* `internal` - the member is visible anywhere within the same assembly with access to the declaring type (and similarly for assemblies matching any `InternalsVisibileToAttribute`).
* `protected internal` - the member's visibility is a combination (max) of `protected` and `internal`.
* `private` - the member is only visible within the declaring type.

_I'm intentionally skipping the subject of reflection, because a consumer using reflection on an otherwise non-visible member does so at their own risk._

## Scoping

So how do access modifiers play into _scoping_? Scope is generally considered to be a combination of visibility, and lifecycle. There are basically two considerations for lifecycles:

* Code Blocks - any variable defined within a code block, such as an `if`, `using`, `foreach`, etc. and also methods themselves, only exist within the block.
* Declaring Type - any variable or member defined beyond the scope of a method, exists in parallel with it's declaring type.

In the case of reference types, additional references can be created pointing to variables and members, and in the case of the `ref` keyword, references to both value and reference types can also be passed around.

I appreciate that might be a confusing statement for some, so let's examine the concepts by examples in the following sections. Hopefully I can explain each by code alone because my MSPaint skills are none to be desired.

### Additional References to a Reference Type

This one is nice and easy, but let's take a step further to examine implications.

```c#
private class Foo
{
  internal object Bar { get; set; } = new object();
}

public Bar AdditionalReference { get; set;}

private void TestCase()
{
  var foo = new Foo();
  AdditionalReference = foo.Bar;
}
```

So let's assume you've called the `TestCase` method in the example above. An instance of the class `Foo` is created and assigned to a local variable, `foo`. The `Bar` property is assigned to as a result of constructing `Foo`. We then create an additional reference to that same `new object()` by assigning `foo.Bar` to the `AdditionalReference` property. After which, the method exits and for all intents and purposes, the instance of `Foo` we created is no longer available.

However, whilst `Foo` may be gone, it's property `Bar` is not. Or at least, the object that was assigned to it, is not. By assigning `foo.Bar` to the `AdditionalReference` property, in the case of a reference type (which `object` is), all we are doing is creating a new _pointer_ to the same object. Thus, an _additional reference_.

Now consider that `Bar` was infact something more sensitive to prolonged lifecycles. What if it was a database connection, or compute heavy task? Did the author of `Foo` want these sensitive objects _leaked_? Did the author foresee how such an object might be mistakenly (or intentionally) prolonged in lifespan, and exposed beyond the declared visibility?

### Implications of `ref`

The previous example was somewhat innocent, but the implications of the `ref` keyword can go far beyond that. `ref` makes it possible to pass the pointer of both value and reference types. I have **never** in all my years of programming, come across a case where `ref` was truly necessary except when crossing a native boundary. In every case, a slight restructure of the code would have left it much easier to read.

In a previous chapter I provided a [`ref` test](http://blog.devbot.net/conventions-refs/#test) for which the outcome should be very obvious. However, that does not seem to be the case. _You_ may understand `ref`s perfectly, but you must also understand there are a huge number of people that do not! And not through fault of their own; as mentioned above, do you **need** it? I would only teach junior's about `ref`, as I am in this article, in order for them to be avoid it. It's a lazy and **unprofessional** means to extend the number of results a method can return.

If you haven't taken the `ref` test, I encourage you to do so.

## The Point

Now that we better understand access modifiers, visibility, and scoping, you may be wondering what the point of this article is. So here we are:

> Keep the scope of **everything** to the lowest you possibly can.

We've examined above some of the unfortunate circumstances that can arise when consumers of your code do not understand the implications of further exposure and extended life spans. So keep as much of your implementation as private and transient as possible.

Keeping your scopes minimal should be somewhat obvious in C# where the default access modifier (when none is supplied) for a type is `internal` and the default for a member is `private`. If only the default modifier for a class was also `sealed`...

However, developers seem to make huge amounts of their code `public`. Intentionally. I can only assume this is because they consider it _easier_ and don't want to faff around when it comes to calling into their code. However, every exposed type and member increases the complexity of maintaining your code. Every exposed member requires consideration if you want to change it, especially in the case of `public` members which may be referenced outside the realms of your own code, and thus the references of a given type or member not visible to you.

SOLID developers work towards the Open/Close principle so that method signatures and such hopefully never change, but reality bites hard sometimes, and being agile means not future-proofing. Try as we might to make our classes open to extension, closed to modification, we will inevitably need to make modifications, if only to make a type more extensible.

This can be further assisted by correctly [abstracting](http://blog.devbot.net/abstracting) your code, but if nothing else, next time you declare something, consider the scope. Can it be `private`? If not, can it be `internal`? Can it be `sealed`? If you later have to go back and increase a modifier, you are at least given the opportunity to consider the ramifications of doing so.

Default yourself to using the lowest possible scope; consider carefully when declaring something above your _default_.

## Next

The next article in this series is [Regions](http://blog.devbot.net/conventions-regions).
