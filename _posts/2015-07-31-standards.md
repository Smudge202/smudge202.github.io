---
layout: post
title: C# Coding Standards and Conventions
published: false
---

Unlike my typical rants and rages, I've decided to put together a _quick_ article on some of the more basic coding standards and conventions I follow, as well as the reason(s) I do so. A lot of the battle of conventions is to offer some form of standardisation, making code easier to read and understand by other developers (and your future-self!), but there are also some legacy practices that have unfortunately lived through the ages that need to be weeded out. On the opposing front, many new practices and language features have been slow on the uptake, perhaps for consistency's sake, but possibly for lack of understanding.

So, without further ado, and in no particular order, this is how to _Smudgify_ your code.

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

As we can see above, the `GetData` method appears to return a `List<string>` which is then enumerated and each element printed to the console. Now imagine we wanted to make some changes to `GetData` so that it returns an `IEnumerable<int>`. In the case of the `Explicit` method, we would have to make changes in that method, but no change would be required in the `Implicit` version.

Some might argue that this is a good thing because it forces you to check callers for breaking changes, but nowadays we all TDD (right!?) so it's no longer necessary. The implicit declaration simply saves time, especially when the object graph has a much deeper hierarchy.
