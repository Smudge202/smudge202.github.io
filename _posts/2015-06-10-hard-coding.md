---
layout: post
title: Make hard coding common sense
published: false
---

Our organisation has a great band of developers that know one another very well. The team is all too aware that I can get a little _stroppy_ (read: filled with rage) when I come across bad blog posts, and are all too keen to point out such posts to me. And thus I was introduced to a [blog article](http://enterprisecraftsmanship.com/2015/06/08/make-hard-coding-your-default-choice/) by Vladimir Khorikov which endorses hard coding configuration values. My team will not have anticipated my reaction to the article; I quite agree with Vladimir, albeit would like to refine his thoughts and provide some structure.

## Configuration Hell

Have you ever been there? It's when you have more configuration settings than you know what to do with; endless combinations, each of which changing behaviours slightly or entirely. The ability to understand, let alone test such monstrosities are often quickly diminished as more settings are added and team members move on to greener pastures.

I have been there. It is an awful place to be.

## Hardcoded Configuration

On itself, hardcoding configuration into your application is _not_ a bad thing to do. So long as you do so cleanly. No one enjoys scatterings of magic numbers across the code base. But having a code file that contains configuration values, is not any different to having those values in a configuration file. Yes, it requires recompilation, but if you have no requirement for that configuration to change, there's no need for it to exist outside of the code. 

**So long as the code that consumes configuration is unaware and unbound from the configuration source**.

