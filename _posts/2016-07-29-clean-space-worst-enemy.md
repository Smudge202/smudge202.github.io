---
layout: post
title: Clean Space - Developer's Worst Enemy
hidden: true
wip: true
tags: [clean-space, game-development]
---

Whether you're a game developer, or write software for large corporate enterprises, there is one frustration we all share. The end user always finds a way to use the software we've laboured over to make it behave _differently_ to how we intended. It's never the user's fault of course, our failings in UI/UX design, our missing restraints and misunderstanding of user intent all point to the developer being at fault. Of course, this only makes the situation all the more frustrating, and more often than it perhaps should, leads us, as software developers, to venting at the perceived _stupidity_ of the user base.

![funny drinking water](http://images-cdn.9gag.com/photo/aM1gyO6_460s_v1.jpg)

_Image from [9gag](http://9gag.com/)_
{: style="font-size: 12px; text-align: center;"}

In this article, I'm going to examine a [user story](https://en.wikipedia.org/wiki/User_story) for _Clean Space_, which I hope will demonstrate just how much thought should be put into these scenarios.

## User Story

The example we're going to look at today is pretty simply, on the face at least. 

> As a player of Clean Space<br />
> I want to be able to **name** my spacecraft<br />
> So that other players can recognise my vehicle

Easy, right? Just store some names in a database somewhere, track it against an entity ID of some kind, display it in the UI. Job done.

### Wrong

Certainly, I could take the above approach. In fact, many might argue I _should_ take that approach and iteratively improve upon that design as further requirements crop up. However, for the sake of pragmatism, a software developer will often find themselves trying to walk the thin line of balancing naviety versus future proofing. Both are equally bad for a product.

Fortunately, I have a fair amount of knowledge to draw upon from my commercial experience here, so the first thing is start _fleshing out_ the requirements. I have to constantly swap between my _Product Owner_ and _Developer_ hats, but let's step through each of the requirements which after hours of heated debate infront of a mirror, I've managed to agree with myself.

### Requirements

So let's get the obvious ones out of the way. Being technically minded, the most immediate constraint that come to mind are that the vehicle names should be unique. Which begs the question, what should we do when a name is not unique? I could of course tell the user _"tough luck, that name has been used"_, but I can think of several cases where that would be a shoddy experience, at best. What if the name was used on a vehicle that belongs to another _faction_/_race_? What if the name was used on a vehicle that has since been de-commissioned or destroyed? What if the vehicle that originally had the name, has since been given a new name?

As tough as it can be, we need to examine these cases. Product Owners I've worked with can attest to the fact that I often play devil's advocate during sprint planning sessions. I don't _dislike_ a given story, nor the product owners themselves; they've come to understand (I hope!) that I don't question the stories so pendantically as any attempt of malice, simply as a prudent matter of professionalism. I'm not trying to complicate things, I'm trying to offer considerations and side effects that may have been overlooked. Whether they are addressed or not immediately is a question for the product owners, but as someone that takes pride in my code, I need to consider all the states my software might be exposed to to ensure it doesn't collapse upon itself the second it's exposed to an end user. We'll never foresee all possible eventualities, but that doesn't mean we shouldn't given them some thought!

With that in mind, let's take a look at the requirements I derived. 

_The following uses [cucumber](https://cucumber.io/)/[gherkin](https://github.com/cucumber/cucumber/wiki/Gherkin)/[specflow](http://www.specflow.org/) syntax which is a great way to communicate between product owners and development team_.  

```cucumber
Scenario: Name is unique
Given the name "HMS Smudge" has not been used before
When a user tries to name their vehicle "HMS Smudge"
Then the vehicle is registered with the name "HMS Smudge"

Scenario: Name has been used by same faction
Given the name "HMS Smudge" has been registered to the "Earth" authorities
When a user tries to name their vehicle "HMS Smudge" in the "Earth" authority
Then the vehcile registration for "HMS Smudge" is declined

Scenario: Name has been used by another faction
Given the name "HMS Smudge" has been registered to the "Martian" authorities
When a user tries to name their vehicle "HMS Smudge" in the "Earth" authority
Then the vehcile is registered with the name "HMS Smudge" 

Scenario: Name has been used by a destroyed vehicle without transfer authority
Given the name "HMS Smudge" has been registered to the "Earth" authorities by the player "Incurus"
And the "HMS Smudge" was destroyed
And "Incurus" has not given authority to use the name "HMS Smudge" to "Smudge"
When the player "Smudge" tries to name their vehicle "HMS Smudge" in the "Earth" authority
Then the player "Incurus" is presented with a request to transfer the title "HMS Smudge"
# Note: Intentionally omitting requirements for title transfer negotiations
```