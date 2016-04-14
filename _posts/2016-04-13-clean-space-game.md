---
layout: post
title: Clean Space Game
wip: true
hidden: true
tags: [clean-space, game-development]
---

In my [previous post](blog.devbot.net/clean-space-introduction/) I talked about the reasons for writing a game, but without going into the detail of what the game is actually about. In this article I hope to clarify some of the high level detail, concepts, and goals of (code name) *Clean Space*, to give an idea of the scale involved and an idea of what this series has to look forward to. I'll admit outright that I'm not following any kind of [guide](http://www.amazon.co.uk/Level-Up-Guide-Great-Design/dp/1118877160) on game design at this stage, so there's a level of professionalism in this regard that you should expect to be missing.

## Brief

_Clean Space_ is, or will be, a space game. Specifically, it's a 4X (e**x**plore, e**x**pand, e**x**ploit, and e**x**terminate). To quote the Rifleman's Creed:

> There are many like it, but this one is mine.

I've played several space games, not least my top 5 which are (in no discernible order):
* [X3: Terran Conflict]()
* [Aurora 4X]()
* [Eve Online]()
* [Homeworld 2]()

### Plausible

Whilst I'll inevitable take inspiration from the 4X, strategy, and RPG games I've played over the years, for better or for worse, this game will differ. As will be explained in future posts about the game engine, [clean engine](https://github.com/clean-development/engine), everything from a technical aspect is most certainly different to other games. This, I hope, will afford me the ability to implement a level of complexity and realism simply not technically possibly in other games.

To ellaborate on the *'realism'* mention there, when talking about space games the best we can do is *plausible realism*. That is, whilst it may not be technically or even physically possible given our current understanding, the game will utilise theoretical physics considered *plausible*, or in some cases, simply *possible*. Some form of *Warp Drive* certainly isn't technically or even physically possible given the human race's current understanding of physics, let alone engineering capabilities, but there's actually people at [NASA contemplating the physics involved](http://www.space.com/22430-star-trek-warp-drive-quantum-thrusters.html) making it a (albeit remote) possibility in the future, and therefore a candidate for integration into the game.

### Technicalities

From a technical perspective, it should go without saying that space is really big. Genuinely, unimaginably enormous. The numbers involved when talking about masses and distances, whilst can be *read* by humans, simply do not compute for us. It doesn't even compute for computers in many cases; the numbers involved can be massively beyond the scope of primitives such as a `long` which is a 64-bit integer, with a maximum value of 9,223,372,036,854,775,807 (9 quintillion in short scale). Don't get me wrong, that is a gigantic number, but even our **closest** star, *Alpha Centauri*, is around 4.1315e+16 metres away (41 quadrillion in short scale). So the distance from Earth to Alpha Centauri in metres is about 1/225 the capacity of the largest primitive available to developers, making it unlikely to be able to store information for anything but a small cluster of nearby stars. 

Of course, their are numerous and obvious issues with that statement, such as the fact that the distance between these celestial bodies, especially if measured at the level of *metres*, is constantly changing, and ignores obvious work arounds such as using a less granular unit of measurement such as an AU (astronomical unit).

### Perspective

Beyond the technical implications of working on such massive scales, consider for a moment taking into account the *speed of information* when designing a UI. The fastest speed to transfer data currently possible is the speed of light (~229,792km/s). To send data to or fro Mars can be anywhere between 182 seconds to 1,342 seconds depending on the distance between Earth and Mars at the time. Whilst I intend to allow FTL capacilities in the game, consider for a moment how these factors would play into trying to display the *position* of a space craft.

The game not only needs to track an *internal* state for which all things be held accountable, but it also needs to be able to correctly identify and simulate a *perspective*. Given a space craft approaching Mars orbit, when viewed from earth my perspective includes an information speed gap of potentially several minutes between _the **expected** position_ and _the **last known** position_. This same information, when viewed from Mars for example, or the space craft itself, would be quite different.

### Myth Debunking

Thanks mostly to Hollywood, there are a huge number of ludicrous inaccuracies that we have come to *accept* with regards to space travel. Most of these can be attributed to our apparent preference to consider vessels in space, naval. Throughout this post and all others, I will go out of my way to describe these space faring craft as anything but a _space**ship**_.

There's no point picking on films, games or books for the propagation of these myths. We set aside the laws of physics (and common sense) for better narration. I certainly don't want the tutorial for *Clean Space* to take 3 real life days to travel between Earth and the Moon, as is the case given our current tech levels. But, at the same time I want to stop treating space craft as *ships*.

#### Space Battles

If the human race ever does make it into space, and two space craft were to battle one another, that battle would take place at incredible speeds over vast distances. The craft would not be firing lasers and phasers and railguns at one another. Lasers can *only* move at the speed of light, so even if it was possible to accurately track your opponent over such a large distance and such great speeds, there would be several seconds or even minutes between firing and *impact*. Your opponent would have to make but the tiniest of course change to *dodge* the fire.

Much more likely would be missiles for medium to long range engagements, any other turretted weapon only useful as point defense for said missiles. Without giving away any spoilers, there is a space battle within the first few episodes of the TV Series, [The Expanse](https://en.wikipedia.org/wiki/The_Expanse_(TV_series)), which I personally think depicts one of the more likely scenarios of space warfare, assuming all the technology involved is possible.
