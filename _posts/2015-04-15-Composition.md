---
layout: post
title: Composition
---

I see a _lot_ of people get application composition _very_ wrong, _very_ often. The problem isn't so much that the subject lacks resources, on contrary, [dependency injection] for example is probably the most talked about and blogged pattern of all time! However, the information presented is all too often incorrect or conflicting, spawned from the enlightened minds of mid level developers keen to share their light bulb moment in which DI and some of it's benefits are suddenly realised.

There seems to be little point in dripping more content into the ocean that is DI resources, so I've decided to take matters a step further and provide a framework. In this article I'll address what I consider to be some of the misconceptions surrounding composition techniques and clarify some terminology. In future articles I'll highlight how the use of [Compose] can assist in rectifying these common flaws, whilst simplifying the definition of manageable service components and resulting integration within your application.

##Terminology

First thing's first, lets add my take on the terminology involved in hope of sparking some dim light into those lightbulbs, and if nothing else, to clarify the rest of this article.

### Inversion of Control (IoC)

Sometimes referred to as the _Hollywood Principle - "Don't call me, I'll call you"_, IoC spans back more than 25 years. Although it made appearances in papers beforehand, it was possibly made most famous in the book; _[Gang of Four]_.

I like to think of it as the broadest description of dependency flow,, to which patterns that can be used to achieve the [Dependency Inversion Principle] all fit in. It describes how reusable, generic components are used by applications and in which direction the flow of control should occur.

For more information in IoC, I recommend checking out [Martin Fowler's article].

### Dependency Inversion Principle (DIP)

DIP is a much more targetted principle when compared to IoC, concerning the decoupling of dependencies so that _high-level_ components can be made independent of _lower-level_ modules. It is a combination of abstracting your concrete implementations, whilst ensuring consumers remain isolated from dependencies.

It is important to note, despite Dependency **Inversion** sharing an acronym with Dependency **Injection**, the two are **not the same**. Dependencies can be _inverted_ simply by using abstractions, and in no way _requires_ dependency _injection_.

This is possibly made more clear in [this post by Derick Bailey] which tackles differentiating the two terms.

### Dependency Injection (DI)

For all the confusion DI brings about at time, I consider it one of the most valuable patterns available. It takes very little time to teach a junior developer to use dependency injection, and is a pretty easy sell if said junior follows TDD (if not, why not!?).

Dependency Injection is simply a design pattern intended to assist in achieving the Dependency Inversion Principle, which in turn can assist with inverting control (IoC). That's it, really. I quite often see people stating it's usefulness with TDD as it's greatest asset, and although I believe that to be a very handy _side-effect_, the key benefit to me is the component isolation it affords us. Not just for the sake of testing, but for the sake of a cleaner structural design.

Notice how each of the terms listed so far have been a smaller, more direct description of something designed to assist in achieving a broader goal.  

Now I hope you'll understand when I describe the key misconception I come across... utilising dependency injection does not mean you have achieved DIP, let alone IoC!

### 

  [dependency injection]: https://www.google.co.uk/#safe=active&q=dependency+injection
  [Compose]: http://www.github.com/smudge202/compose
  [Gang of Four]: http://www.amazon.co.uk/Design-patterns-elements-reusable-object-oriented/dp/0201633612
  [Dependency Inversion Principle]: http://
  [Martin Fowler's article]: http://martinfowler.com/bliki/InversionOfControl.html
  [this post by Derick Bailey]: https://lostechies.com/derickbailey/2011/09/22/dependency-injection-is-not-the-same-as-the-dependency-inversion-principle/
