---
layout: post
title: Clean Space - The Game
tags: [clean-space, game-development]
---

In my [previous post](http://blog.devbot.net/clean-space-introduction/) I talked about the reasons for writing a game, but without going into the detail of what the game is actually about. In this article I hope to clarify some of the high level detail, concepts, and goals of (code name) *Clean Space*, to give an idea of the scale involved and an idea of what this series has to look forward to.

## Brief

*Clean Space* is, or will be, a space game. Specifically, it's a 4X (e**x**plore, e**x**pand, e**x**ploit, and e**x**terminate). To quote the Rifleman's Creed:

> There are many like it, but this one is mine.

I've played several space games, not least my top 5 which are (in no discernible order):

* [X3: Terran Conflict](http://www.egosoft.com/games/x3tc/info_en.php)
* [Aurora 4X](http://aurora2.pentarch.org/)
* [Eve Online](http://www.eveonline.com/)
* [Homeworld 2](http://www.homeworldremastered.com/)
* [Kerbal Space Program](https://kerbalspaceprogram.com/en/)
 
If I haven't listed your favourite space game, don't take it to heart, we all have our preferences. I own [Elite: Dangerous](https://www.elitedangerous.com/) and found it a tedious grind, despite playing during the beta with a 1/10 economy. [Star Citizen](https://robertsspaceindustries.com/) looks amazing but that's it, there's simply no content and, whilst I'd love to be proven wrong, I don't have faith in the guys to deliver. There are of course great space games from the past such as [Freelancer](https://en.wikipedia.org/wiki/Freelancer_(video_game)), but I wouldn't want to play it today (nor would I expect it to run on Windows 10).

As with almost every game genre, there are a lot of great (and even more bad) games out there. I'm not aiming for *x% market share* or *y number of users*. Given how vile gaming/internet communities can be, it wouldn't be all that bad if I was the only person to ever play *Clean Space*. So long as I enjoy it.

## Content

So, what is the game about? What type of game is it?? Interesting question... First and foremost, it's a 3D 4X strategy akin *Aurora 4X*. It is *not* a pretty game, in fact, as you'll see in the coming articles, beyond getting some poorly textured or wireframed models onto the screen so I can prove and test aspects of gameplay, I'm not too concerned with the UI right now (though, I do care for the UX). My hope is that parts, if not all of the game, can be open sourced, which maybe one day will inspire a designer to come along and spruce things up.

The game will be set in near to distant future (depending on how long you play of course), and will play out much more like a simulation than an arcade game. To give you an idea of how detailed the simulation will be, here are a couple of points:

* Every single game feature is written for the AI to use (preferably, *correctly*), before being made available to the player.
* The Goal is to incorporate a fully functional, fully featured mode whereby the player could be a Captain.
 
If you've ever played any of the later *SimCity* games, you'll probably have come across a feature whereby you could attach a camera to one of your citizens and follow them around, perhaps even control their movement. The intent with *Clean Space* is to provide a full game, at that level (think *X3: Terran Conflict* or *Star Trek: Bridge Commander*). The player can choose (maybe even switch) between these modes. When Governing an empire, the AI will control ships in the same way a player would, with all the features and functionality available to the player, and vice-versa when the player is a Captain.

With an impossible goal now firmly established, let's talk technicalities.

## Problems

This blog series is not a *how to* (though I hope many learn from it!). In fact, I'll often post misconstrued *facts* and make mistakes in the code I show, which I'll try to remember to correct when I realise my mistakes. I want this series to be about *the journey*. I'll talk about small tasks that I tackle on a day-to-day basis, how I go about researching subjects, and what I learn. I am not a *Game Developer*. **I've never written a game**. But, I am a pretty good developer, and I'd like to think I'm not a total idiot.

Whilst I'll inevitable take inspiration from the 4X, strategy, and RPG games I've played over the years, for better or for worse, *Clean Space* will differ. As will be explained in future posts about the game engine, [clean engine](https://github.com/clean-development/engine), everything from a technical perspective is most certainly different to other games. This, I hope, will afford me the ability to implement a level of complexity and *realism* simply not technically possibly in other games.

To ellaborate on the *'realism'* mention there, when talking about space games, the best we can do is *plausible realism*. That is, whilst something may not be possible given our current understanding, the game will utilise theoretical physics considered *plausible*, or in some cases, simply *possible*. Some form of *Warp Drive* certainly isn't possible given the human race's current understanding of physics, let alone engineering capabilities, but there's actually people at [NASA contemplating the physics involved](http://www.space.com/22430-star-trek-warp-drive-quantum-thrusters.html) making it a (albeit remote) possibility in the future, and therefore a candidate for integration into the game.

Throughout this series, there will be articles discussing the tasks I set out to achieve, and problems I face. Expect there to be a lot of problems. The following are 2 example problems we'll tackle at a later stage, to warm your brain up...

### Technicalities

From a technical perspective, it should go without saying that space is really big. Genuinely, unimaginably enormous. The numbers involved when talking about masses and distances, whilst can be *read* by humans, simply do not compute for us. It doesn't even compute for computers in many cases; the numbers involved can be massively beyond the scope of primitives such as a `long` which is a 64-bit integer, with a maximum value of 9,223,372,036,854,775,807 (9 quintillion in short scale). Don't get me wrong, that is a gigantic number, but even our **closest** star, *Alpha Centauri*, is around 4.1315e+16 metres away (41 quadrillion in short scale). So the distance from Earth to Alpha Centauri in metres is about 1/225 the capacity of the largest primitive available to developers, making it unlikely to be able to store information for anything but a small cluster of nearby stars. 

Of course, their are numerous and obvious issues with that statement, such as the fact that the distance between these celestial bodies, especially if measured at the level of *metres*, is constantly changing, and ignores obvious work arounds such as using a less granular unit of measurement such as an AU (astronomical unit).

### Perspective

Beyond the technical implications of working on such massive scales, consider for a moment taking into account the *speed of information* when designing a UI. The fastest speed to transfer data currently possible is the speed of light (~229,792km/s). To send data to or fro Mars can be anywhere between 182 seconds to 1,342 seconds depending on the distance between Earth and Mars at the time. Whilst I intend to allow FTL capacilities in the game, consider for a moment how these factors would play into trying to display the *position* of a space craft.

The game not only needs to track an *internal* state for which all things be held accountable, but it also needs to be able to correctly identify and simulate a *perspective*. Given a space craft approaching Mars orbit, when viewed from earth my perspective includes an information speed gap of potentially several minutes between _the **expected** position_ and _the **last known** position_. This same information, when viewed from Mars for example, or the space craft itself, would be quite different.

## Enough Talk Already!

Ok, I'm done. Keep an eye out for the [next article](http://blog.devbot.net/tag/clean-space/) or [subscribe](http://blog.devbot.net/feed.xml) for the next article, which will be our first foray into code.
