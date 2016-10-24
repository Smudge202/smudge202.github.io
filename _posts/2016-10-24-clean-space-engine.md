---
layout: post
title: Clean Space - The Engine
hidden: true
wip: true
---

Last year I spent some time [studying Game Development](http://blog.devbot.net/industry/), trying to understand where differences exist in practices, and why. I was less than impressed with what I found, but decided I should walk a mile in a game developer's shoes by not only [writing a game](http://blog.devbot.net/tag/clean-space/), but possibly creating the most complex game there has ever been (I know, _citation needed_, but you'll see in due course...).

Considering all the amazing advancements in graphics over the years, I'm amazed at how poor tooling, frameworks, and the entire game development eco-system is right now. Whilst there has probably never been so many indie developers, and access to game engines is possibly the best it's ever been, these resources feel about a decade behind that which is available to other software industries.

Which is why, at least in part, I decided to fix it.

## Limitations

_Clean Space_ intends to not only provide me with a great learning resource and enjoyable gameplay, but to make a few statements. For one, I want to show that _clean_ coding is not only possible in game development, but should be _preferable_. In addition, I want to show that you don't _need_ to write your games in C++, nor do you need to make silly concessions in higher level languages like C# by substituting properties for fields, all in the name of [_performance_](http://web.ageofascent.com/asp-net-core-exeeds-1-15-million-requests-12-6-gbps/).

To that end, _Clean Space_ has the self-imposed _limitation_ of being entirely written in C# (plus [Gherkin](https://github.com/cucumber/cucumber/wiki/Gherkin) if you include the tests, `xaml` if you include the UWP forms, etc). So to, is [Clean Engine](https://github.com/clean-development/engine) _(the game engine used)_. The _rendering_ engine used, be it [Unity](https://unity3d.com/), [MonoGame](http://www.monogame.net/), or similar, of course may use other languages, but they're not my concern.

# Engines

Some of you may have done a double-take of that last paragraph. Most people consider _Unity_ and _MonoGame_ to be _Game Engines_, certainly if you navigate to the [Unity3D Home Page](https://unity3d.com/), it says "Unity - Game Engine" in the tab title. So why do I need _Clean Engine_ ontop of a _game engine_? 

The problem with _traditional_ game engines is the [game loop](http://gameprogrammingpatterns.com/game-loop.html). If you follow the tutorials for any of these engines, you'll see that they try to push you down the route of placing your _game logic_ within the _game loop_. You need to react to user input, amend your state, and render accordingly, within this loop. Fair enough, but what if my game uses I/O? I need to read/write a file or send/receive data over a network? I can't just block the thread within the game loop (any game where the UI freezes while your saving a game decided otherwise).

So, I need to consider _asynchrony_. But as the .Net `async/await` keywords prove, _asynchrony_ does not simply mean _multi-threaded_. There's more to it than that.

# Async/Await

Contrary to seemingly popular misunderstanding, these keywords are not about creating new threads to run tasks. Certainly, [`Task.Run`](https://msdn.microsoft.com/en-us/library/system.threading.tasks.task.run(v=vs.110).aspx) can queue work on the ThreadPool, but that's not where the power of these keywords exist. Low level Windows API's for I/O use [completion callback mechanisms](https://msdn.microsoft.com/en-us/library/windows/desktop/ms684124(v=vs.85).aspx), which allows us to _yield_ threads instead of waiting for said I/O. The `async/await` keywords can therefore intelligently utilise threads instead of exhausting the ThreadPool waiting around for I/O to complete.

This model is widely adopted across the .Net ecosystem, even the ancient WinForms technology allows async UI event handlers! But, _Game Loops_ do not.