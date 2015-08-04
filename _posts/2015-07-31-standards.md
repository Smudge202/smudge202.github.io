---
layout: post
title: C# Coding Standards and Conventions
published: false
---

Unlike my typical rants and rages, I've decided to put together a _quick_ article on some of the more basic coding standards and conventions I follow, as well as the reason(s) I do so. A lot of the battle of conventions is to offer some form of standardisation, making code easier to read and understand by other developers (and your future-self!), but there are also some legacy practices that have unfortunately lived through the ages that need to be weeded out. On the opposing front, many new practices and language features have been slow on the uptake, perhaps for consistency's sake, but possibly for lack of understanding.

So, without further ado, and in no particular order, this is how to _Smudgify_ your code.

## Casing

There are a [fair few rules to casing](https://msdn.microsoft.com/en-us/library/x2dbyw72(v=vs.71).aspx) but here's the gist of it:

### Lower Camel ( _foo**B**ar_ )

* Private members
* Method arguments
* Local variables

### Pascal (_**F**oo**B**ar_ )

* Class names
* Namespace names
* Public, Internal, and Protected members
* Generic Arguments (the declarations, i.e. `Func<TResult>`)

_It's increasingly common to see Constants declared in Pascal case, however upper-case declarations are also ok for `const`_

### Acronyms

* For 3 letters or more, treat the Acronym as a normal word (i.e. `var xmlReader = new XmlReader()`)
* For 2 letter acronyms, use all upper **except** in the case of the well known few such as `Id`.

_For the most part, try to avoid using Acronyms. There is almost no restriction on member name length, so describe your code's intent so that it's easy to read. If you need the code obfuscated, use a tool to do so, don't be a tool_.

## Underscores

There are only two legitimate uses for underscores, and both of them are a matter of personal preference. 

### Backing Members

I use an underscore to prefix a backing member (private instance) to remove the need for the `this` keyword in my constructors. Some people prefer to use the `this` keyword and be completely rid of underscores.

i.e.

```c#
private readonly IFoo _foo;

public MyClass(IFoo foo)
{
  _foo = foo;
}

\\ versus

private readonly IBar bar;

public MyClass(IBar bar)
{
  this.bar = bar;
}
```

### Test Names

It isn't a practice I follow personally, but I have no problem when I come across test names that use underscores. For example:

```c#
[Fact]
public static void Given_Foo_When_Bar_Then_Does_Stuff()
{ /* ... */ }

// versus

[Fact]
public static void GivenFooWhenBarThenDoesStuff()
{ /* ... */ }
```

For whatever reason, I find it easier to read the latter than the former, so I don't use underscores in test names. It drives me mad when I see underscores used in namespaces, class names, and so forth. **This is DOT net, not UNDERSCORE net**.

## Implicit Declarations

A lot of people seem to think of this as a _personal preference_, but I want to point out that declarations utilising the `var` keyword _are_ better. Here's an example why:

```c#
public void Explicit()
{
  List<string> data = GetData();
  foreach (string item in data)
    Console.WriteLine(item);
}

public void Implicit()
{
  var data = GetData();
  foreach (var item in data)
    Console.WriteLine(item);
}
```

As we can see above, the `GetData` method appears to return a `List<string>` which is then enumerated, with each element printed to the console. Now imagine we wanted to make some changes to `GetData` so that it returns an `IEnumerable<int>`. In the case of the `Explicit` method, we would have to make two changes, but no change would be required in the `Implicit` version.

I appreciate that some people prefer to see the type of a variable, but IDE's make this information readily available, especially in the case of Visual Studio. Others might argue that being forced to make a change is a good thing because it forces you to check callers for breaking changes, but nowadays we all TDD (right!?) so it's no longer necessary. We should trust in our tests to identify breaking changes.

The implicit declaration simply saves time, especially when the object graph has a much deeper hierarchy.

## Comments

Don't use them. Your code should describe what your code is doing. If you think your code is hard enough to read to justify a comment, change your code.

I know sometimes we comment code out to test something in debug, when trying to diagnose an issue, or when trying to get the code to compile quickly, but **never commit commented out code**. The whole point of source control is to give us a history of changes, we simply do not need commented-out code polluting the code base.

_As a slight caveat, I will in rare cases add a comment to my code to acknowledge/explain the reason behind an unenviable design decision such as a third party API forcing me to use Exception Logic or call some seemingly unrelated method to make a later call work. I consider this an unavoidable case for using a comment_.

## Exceptions

The `try/catch` block is all too often misused, so here are a couple suggestions:

### Exception Logic

Exception logic is the act of checking whether an exception is thrown by a line of code as a type of `if` statement. For example:

```c#
public bool TryGetData(out int result)
{
  result = default(int);
  try
  {
    result = int.Parse(GetStringRepresentation());
    return true;
  }
  catch
  {
    return false;
  }
}
```

Instead, always try to make use of conditions available; generating exceptions is [no small task](http://blogs.msdn.com/b/ricom/archive/2006/09/25/771142.aspx). For example, the above could have been simply changed to:

```c#
public bool TryGetData(out int result)
{
  return int.TryParse(GetStringRepresentation(), out result);
}
```

There will be cases where exception logic cannot be avoided, but always **use it as a last resort**!

### On Error Resume Next

One of the most bizarre statements I recall learning back in VB days was _On Error Resume Next_, which simply means to ignore an exception and continue processing. **Exceptions are exceptional**. This is sometimes mixed with _exception logic_ but not in all cases. I appreciate developers will also often try/catch an exception because they don't want an exception to bubble, but oftentimes the higher level component is in a better position to handle and react to an exception.

### Validation

Validation comes in two forms. There is code validation which asserts that method arguments are valid, correct code flow is followed (opening a connection before using it for example), and so forth. These are perfectly suitable places to `throw` a **relevant and descriptive** exception, but should not be confused with _user input validation_.

There are those that argue that it is easier to throw exceptions for malformed user input and have high level catch blocks generate a friendly UI from the exception, but this is not a practice I follow or advocate. Validation issues should be returned as part of a model whilst exceptions are used for their intended purpose.

