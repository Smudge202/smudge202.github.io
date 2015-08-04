---
layout: post
title: Comments - Modern C# Standards and Conventions
hidden: true
---

This article is part of a series describing [Modern C# Standards and Conventions](http://blog.devbot.net/standards). A list of the other articles in this series can be found below.

## Contents

* [Casing](http://blog.devbot.net/conventions-casing)
* [Underscores](http://blog.devbot.net/conventions-underscores)
* [Implicit vs Explicit](http://blog.devbot.net/conventions-implicit)
* Comments - **You are here**
* [Exceptions](http://blog.devbot.net/conventions-exceptions)
* [ref & out Keywords](http://blog.devbot.net/conventions-refs)
* [Scoping](http://blog.devbot.net/conventions-scoping)
* [Regions](http://blog.devbot.net/conventions-regions)

_If you feel there is something else that should be added here, please drop a comment below or better yet, [send me a PR](https://github.com/smudge202/smudge202.github.io)_

## Comments

Comments come in 4 forms, which I'll address individually. But, as a rule of thumb, **do not use comments in code**. 

With that sweeping statement in place, let's examine why.

### XML Comments

XML Comments are the comments you find above member and type declarations. You can add these easily by moving the caret to the line above the declaration and typing `///` (three forward-slashes) which will automatically generate an XML template. These can then be used by various generators to define _""documentation""_ for the code, and can also be used to generate intellisense for people using the code.

However, I always endeavour to make my API **obvious** to the consumer, and provide _actual_ documentation for anything that may not be obvious at first glance. 

I know some companies _enforce_ the policy of having a XML comments for any exposed API, but you'll often find developers add useless information to the XML comments such as `id argument: this is the ID argument`. The time taken to generate that is a constant deduction for something that provides **zero value**.

As always, there are probably many examples of when this isn't the case and an XML comment may be of use to consumers. However, it is not a practice I tend to follow or encourage in others.

### Code Comments

These are comments you find through-out code, typically describing what the code nearby does.

There are a plethora of issues with this. Not least of which is that the **code describes what the code is doing**. If you feel you need to add a comment to describe what the code is doing, change the code to make it more readable. Comments become stale and disconnected as the code to which the comments relate is maintained, leading to comments describing behaviour no longer present, or the associated code being moved whilst the comment sits in situ.

Code comments are far too often misleading, and entirely unnecessary if you write descriptive code.

### Commented-out Code

This is a pet hate of mine. Long gone are the days of restoring commented out behaviour at a later stage. It is one of the many issues that source control resolves.

I may comment out some code that I'm in the process of refactoring, as it forms a nice _to-do list_, however there should never be a case of committing that code to the repository.

### TODO

I'm not sure how many people know this, but in Visual Studio you can go to the `View` menu, and select `Task List` (Ctrl+\, T). This gives you a window that will display all cases of the line `// TODO` in your code, which you can expand upon to provide a poor-mans' task list. For example:

```c#
public void GetData()
{
  throw new NotImplementedException();
  // TODO : Get some tests in so we can flesh out this implementation
}
```

This gives me a nice little message in my Task List window. However, I would much rather have a failing test to mark incomplete behaviour, or a properly defined task (_work item_ in TFS or _issue_ in GitHub for example). I've found it's far too easy to ignore the Task List window in Visual Studio, and it forces a _Find in Files_ search on collaborators and peers who may be working in another IDE (not Visual Studio).

### Caveat

I have listed a few exceptions in the various categories above, but I will admit one further exception to the rule. In rare cases, I add a comment to code to acknowledge/explain the reason behind an unenviable design decision. For example, if a third party API forces me to use Exception Logic such as `OperationCanceledExceptions` on a Handle Wait (thanks MS), or perhaps a call to some seemingly unrelated method to make a later call work (saw a lot of this using an old Sage API). 

I consider these to not only be an occasion in which you _could_ add a comment, but in fact situations in which you **should** add a comment. Ideally provide links to supporting documentation or evidence, and abstract the call away - not to hide it, but to invite the opportunity to change or replace it.

## Next

The next article in this series is [Exceptions](http://blog.devbot.net/conventions-exceptions).
