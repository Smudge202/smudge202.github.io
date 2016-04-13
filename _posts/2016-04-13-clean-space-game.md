---
layout: post
title: Clean Space Game
tags: [clean-space, game-development]
---

In my [previous post](blog.devbot.net/clean-space-introduction/) I talked about the reasons for writing a game, but without going into the detail of what the game is actually about. I hope to clarify some of the high level detail of (code name) *Clean Space* in this article to give an idea of the scale involved, and to give an idea of what this series has to look forward to. I'll admit outright that I'm not following any kind of [guide](http://www.amazon.co.uk/Level-Up-Guide-Great-Design/dp/1118877160) on game design at this stage, so there's a level of professionalism in this regard that you should expect to be missing.

## Brief

_Clean Space_ is, or will be, a space game. Specifically, it's a 4X (e**x**plore, e**x**pand, e**x**ploit, and e**x**terminate). To quote the Rifleman's Creed:

> There are many like it, but this one is mine.

### Plausible

Whilst I'll inevitable take inspiration from the numerous 4X, strategy, and RPG games I've played over the years, for better or for worse, this game will differ. As will be explained in future posts about the game engine, [clean engine](https://github.com/clean-development/engine), everything from a technical aspect is most certainly different to other games. This, I hope, will afford me the ability to implement a level of complexity and realism simply not possibly in other games.

To ellaborate on the *'realism'* mention there, when talking about space games the best we can do is *plausible realism*. That is, whilst it may not be technically or even physically possible currently, the game will utilise theoretical physics considered *plausible*, or in some cases simply *possible*.

### Technicalities

From a technical perspective, it should go without saying that space is really big. Genuinely, unimaginably enormous. The numbers involved when talking about masses and distances, whilst can be read by humans, simply do not compute for us. It doesn't even compute for computers in many cases; the numbers involved can be massively beyond the scope of primitives such as a `long` which is a 64-bit integer, with a maximum value of 9,223,372,036,854,775,807 (9 quintiliion in short scale). Don't get me wrong, that is an enormous number, but even our **closest** star, *Alpha Centauri*, is around 4.1315e+16 metres away (41 quadrillion in short scale). So the distance from Earth to Alpha Centauri in metres is about 1/225 the capacity of the largest primitive available to developers, making it unlikely to be able to store information for anything but a small cluster of stars. 

Of course, their are numerous and obvious issues with that statement, such as the fact that the distance between these celestial bodies, especially if measured at the level of *metres*, is constantly changing, and ignoring obvious work arounds such as using a less granular unit of measurement such as an AU (astronomical unit).

### Perspective

Beyond the technical implications of working on such massive scales, consider for a moment taking into account the *speed of information* when designing a UI. The fastest speed to transfer data currently possible is the speed of light (~229,792km/s). To send data to or fro Mars can be anywhere between 182 seconds to 1,342 seconds depending on the distance between Eart and Mars at the time. Whilst I intend to allow FTL capacilities in the game, consider for a moment how these factors would play into trying to display the *position* of a space craft.

The game not only needs to track an *internal* state for which all things be held accountable, but it also needs to be able to correctly identify and simulate a *perspective*. Given a space craft approaching Mars orbit, when viewed from earth my perspective includes an information speed gap of potentially several minutes between _the **expected** position_ and _the **last known** position_. This same information, when viewed from Mars for example, or the space craft itself, would be quite different.
