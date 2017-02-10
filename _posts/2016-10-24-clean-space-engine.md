---
layout: post
title: Clean Space - The Engine
hidden: true
wip: true
---

Last year I spent some time [studying Game Development](http://blog.devbot.net/industry/), trying to understand where differences exist in practices, and why. I was less than impressed with what I found, but decided I should walk a mile in a game developer's shoes by not only [writing a game](http://blog.devbot.net/tag/clean-space/), but by creating one of the most complex games in terms of content.

Considering all the amazing advancements in graphics over the years, I'm amazed at how poor tooling, frameworks, and the entire game development eco-system appears to be right now. Whilst there has probably never been so many indie developers, and access to graphics engines is possibly the best it's ever been, these resources feel about a decade behind that which is available to other software industries. The emphasis on these technologies is forever pushing towards the frontiers of improvements to hardware and graphics, with usability, user experience, and user support taking a back seat.

This article will discuss some of the problems I've found with regards to traditional game development using [Unity](https://unity3d.com/) and [MonoGame](http://www.monogame.net/), and how _Clean Engine_ goes about rectifying issues such as Dependency Injection and inherent `async` support. 

## Limitations

_Clean Space_ intends to not only provide me with a great learning resource and enjoyable gameplay, but to make a few statements. For one, I want to show that _clean_ coding is not only possible in game development, but should be _preferable_. In addition, I want to show that you don't _need_ to write your games in C++, nor do you need to make concessions in higher level languages like C# by, for example, substituting properties for fields, all in the name of [_performance_](http://web.ageofascent.com/asp-net-core-exeeds-1-15-million-requests-12-6-gbps/).

To that end, _Clean Space_ has the self-imposed _limitation_ of being entirely written in C# (plus [Gherkin](https://github.com/cucumber/cucumber/wiki/Gherkin) if you include the tests, `xaml` if you include the UWP forms, etc). So to, is [Clean Engine](https://github.com/clean-development/engine) _(the game engine used)_. The _graphics_ engine used, be it _Unity_, _MonoGame_, or similar, of course may use other languages, but they're not my concern.

## Engines

Some of you may have done a double-take of that last paragraph. Most people consider _Unity_ and _MonoGame_ to be _Game Engines_, certainly if you navigate to the [Unity3D Home Page](https://unity3d.com/), it says "Unity - Game Engine" in the page title. So why do I need _Clean Engine_ ontop of a _game engine_? 

The problem with _traditional_ game engines is the [game loop](http://gameprogrammingpatterns.com/game-loop.html). If you follow the tutorials for any of these engines, you'll see that they try to push you down the route of placing your _game logic_ **within** the _game loop_. You need to react to user input, amend your state, load or locate content, and render accordingly, all within this loop. Fair enough, but what if my game uses I/O? I need to read/write a file or send/receive data over a network? I can't just block the thread within the game loop (any game where the UI freezes while your saving a game decided otherwise).

So, I need to consider _asynchrony_. But as the .Net `async/await` keywords prove, _asynchrony_ does not simply mean _multi-threaded_. There's more to it than that.

## Async/Await

Contrary to what seems to be popular misunderstanding, the `async` and `await` keywords are not about creating new threads to run tasks. Whilst [`Task.Run`](https://msdn.microsoft.com/en-us/library/system.threading.tasks.task.run(v=vs.110).aspx) can queue work on the ThreadPool, that's not where the power of these keywords exist. Lower level Windows API's for I/O use [completion callback mechanisms](https://msdn.microsoft.com/en-us/library/windows/desktop/ms684124(v=vs.85).aspx), which allow us to _yield_ threads instead of wasting them waiting for I/O. So long as you understand the potential pitfalls, it can be immensely powerful, and simple.

The model is so widely adopted across the .Net ecosystem, even technologies as old as WinForms now allow async UI event handlers. But, _Game Loops_ in _Unity_ and _MonoGame_ do not. It's somewhat baffling as to why such widely adopted frameworks don't support `async/await`. Certainly, these models may not have been around or widely adopted when the frameworks first hit the market, but the same can be said for WinForms. As will be explained in this and future articles, the mechanisms used by _Clean Engine_ and _Clean Space_ rely heavily, where possible, on eventual consistency. That the code is designed to facilitate and encourage event sourcing and CQRS _(Command Query Repsonsisiblity Segregation)_, makes having access to asynchronous API's all but crucial.

## Dependency Injection

//TODO: link to ioc shizzle, brief desc, etc.

## Layering

For a long time I was unable to decide which of the traditional C# graphics engines I wanted to use for the rendering of _Clean Space_. I still find myself switching between _Unity_ and _Monogame_ as one or t'other drives me into a rage with a lack of documentation, poor tooling, broken and confusing APIs... The practice of switching between these frameworks however has been very beneficial. Where possible, _Clean Engine_ has evolved to provide abstractions to which _Clean Space_ can be written against, with pluggable implementations available for the underlying rendering engines. This has had the added bonus of allowing me to improve upon those APIs, hiding away noise and workarounds.

It is upon this layer that _Clean Engine_ provides it's improved game design model. 

However, I have not abstracted the underlying frameworks completely, just for the sake of having done so. Almost all of the modelling, textures, lighting, cameras, and general rendering concepts are left untouched as these tend to be the most volatile components. I don't believe there would be significant benefit to having abstracted both frameworks, especially given the ongoing maintenance cost of having done so, not to mention the _lowest common denominator_ issues such abstraction would cause. The time spent switching between these frameworks, trying to realign and reimplement the UI, I believe, would be greatly exceeded by maintaining said abstractions, plus suffer from typical disadvantages of LoD (law of demeter, resulting from Interface Segregation - SOLID).

## Structure

