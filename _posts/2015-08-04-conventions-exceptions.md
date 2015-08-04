---
layout: post
title: Exceptions - Modern C# Standards and Conventions
hidden: true
---

This article is part of a series describing [Modern C# Standards and Conventions](http://blog.devbot.net/standards). A list of the other articles in this series can be found below.

## Contents

* [Casing](http://blog.devbot.net/conventions-casing)
* [Underscores](http://blog.devbot.net/conventions-underscores)
* [Implicit vs Explicit](http://blog.devbot.net/conventions-implicit)
* [Comments](http://blog.devbot.net/conventions-comments)
* Exceptions - **You are here**
* [ref & out Keywords](http://blog.devbot.net/conventions-refs)
* [Scoping](http://blog.devbot.net/conventions-scoping)
* [Regions](http://blog.devbot.net/conventions-regions)

_If you feel there is something else that should be included here, please drop a comment below, or better yet [send me a PR](https://github.com/smudge202/smudge202.github.io)._

## Exceptions

An exception is the .Net Framework representation for an error condition or unexpected behaviour. That is a pretty broad description but it's basically what [MSDN says](https://msdn.microsoft.com/en-us/library/5b2yeyab(v=vs.110).aspx) on the subject.

There are two sides to Exceptions which will help us break down this article. They are _Handling_ exceptions, and _Throwing_ exceptions. As you will see by the length of this article, both are commonly misused and misunderstood, but I will try keep each section brief and to the point.

## (Mis)Handling Exceptions

The essence of exception handling in C# (and many other languages) is the `try/catch` block, whereby you _try_ to execute one or more lines of code, and can _catch_ a model representation of an exception should one occur. We'll primarily focus on the `catch` block and statement, however it's worth noting that you should try to limit the size and extent of your `try` blocks. You should pay attention to the types of Exception that can be thrown, and only place the code that _may_ throw an exception in said block.

It may be that you have a broader _catch-all_ statement towards the application entry in order to log the condition and state of the application and perhaps provide an **apology** to the end-user. However, these broad catch blocks should not be littered throughout your code!

### Exception Logic

Exceptions are _exceptional_, and we can but aim to do our best in the case one is thrown. What we should *not* do is make Exceptions an expectation. 

There are two common examples of such behaviour, the first is _Validation_ which we will examine later, but there is also something commonly referred to as _Exception Logic_. That is, we use an exception to represent conditional flow in the same way an `if` statement is used. For example:

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

You see here that we attempt to parse some data. If we are unable to parse the data, an exception is thrown which we _swallow_, and return false. The golden path is of course that we are able to parse the data and return true. This is a pretty simple and obvious example, but is worth bearing in mind next time you write a `try/catch`.

Exceptions should not be used in place of an `if` statement. i.e.

```c#
public bool TryGetData(out int result)
{
  if (int.TryParse(GetStringRepresentation(), out result))
    return true;
  else
    return false;
}
```

I'm simply using this _middle-step_ to demonstrate that there is an implicit `if` statement in play, but of course the code example can be slimmed down to:

```c#
public bool TryGetData(out int result)
{
  return int.TryParse(GetStringRepresentation(), out result);
}
```

There _will_ be cases in which exception logic cannot be avoided, but always **use it as a last resort**!

### On Error Resume Next

A somewhat bizarre statement I recall coming across back in my VB days, `On Error Resume Next` means to ignore an exception and continue processing. Oftentimes mixed with aforementioned _Exception Logic_, but also occasionally used in solitude. This is a big no-no, and includes `try/catch` blocks that have nothing but a `TODO` comment in the `catch` block.

```c#
try
{
  DoSomething();
}
catch
{
  // TODO : Should probably write this to a log or something
}
```

This is not **handling** an exception, it is attempting to **ignore** an exception. This can lead to all kinds of atrocities further down the line, which I'll try not to run through my _what-if-generator_. Don't swallow exceptions unless you really, **really** have to.

### Catch Scope

The root most object in the exception hierarchy, is the `Exception` object itself, from which _all_ other exceptions derive. The `catch` block allows us to filter our _handling_ logic based upon inheritance, so that if we try to catch an `ArgumentException` for example, we would also catch instances of `ArgumentNullException` (unless it has a separate catch block), because `ArgumentNullException` inherits from `ArgumentException`.

_As of C#6 there are additional filters we can apply to our catch blocks, but I'll omit these for sake of brevity. The same concepts apply for C#6, if not more so due to the additional filtering the features allow._

So by _catch scope_, I mean _"How many exceptions are within scope of your catch block"_. This should be the **smallest** number your code can possibly get away with. For example, if I'm working with sockets, I should only catch `SocketException` or derivatives, never `Exception`.

This is made easier if you follow the advice above about keeping your `try` blocks as small as possible.

## Throwing Exceptions

On the flip side of catching exceptions, developers also have the ability to `throw` exceptions. There are some commonly made mistakes and misunderstanding regarding the implications of throwing an exception, so let's start with some basics.

### Exception Stack

As you've probably seen, when an exception is thrown the runtime will collect information regarding the call stack at the point the exception is thrown, and add that information to the Exception. It can be very useful seeing which method and class throws an exception when diagnosing an issue, but equally vital can be seeing which methods were invoked in order to reach that location in code.

Unfortunately, it is possible for un-informed developers to accidentally drop that stack information, as shown here:

```c#
try
{
  DoSomething();
}
catch (SpecificException ex)
{
  _log.Error(ex);
  throw ex;
}
```

You see here that we are trying to log some information before bubbling the details up. However, in doing so, the stack information in the bubbled exception no longer points to the originating method, it points to your catch block! Please don't do this unless you truly intend to - you may do this for security reasons to prevent some sensitive information being leaked.

The correct behaviour in 99.999% of cases is to use one of the following two techniques:

```c#
try
{
  DoSomething();
}
catch (SpecificException ex)
{
  _log.Error(ex);
  // by removing the `ex` on the throw, we preserve the stack.
  throw; 
}
catch (AnotherException innerEx)
{
  // we can preserve the stack of the original exception
  // by wrapping it up safely in another exception which can 
  // then be used to provide additional information.
  throw new OuterException("Some useful information", innerEx);
}

```

### Validation

I've seen this hotly debated in the past, so feel like I should probably explain this as quickly and concisely as possible. _Validation_ falls into two main categories. There is **Code Validation** which involves making sure your method arguments are within expected ranges for example, and there is **User Input Validation**. The former can and should use exceptions, the latter I do not believe should.

Ignoring that _user input validation_ probably falls into the _Exception Logic_ section above, Exceptions are not cheap. Whether thrown by the runtime or in user code, generating exceptions is [no small task](http://blogs.msdn.com/b/ricom/archive/2006/09/25/771142.aspx). If you're ignoring the mis-use of _exception logic_, and the performance cost of doing so, consider this: We actively differentiate and segregate domain from view; why would we tie ourselves to the limitations, conditions and restrictions of an Exception, when user-input validation is clearly a case for View logic.

I know many will still argue in favour of Exceptions for validation, but this is an age-old argument to which I've already chosen sides because the argument "but it's easier" is to me, **unprofessional**.

## Next

The next article in this series is [ref & out Keywords](http://blog.devbot.net/conventions-refs).
