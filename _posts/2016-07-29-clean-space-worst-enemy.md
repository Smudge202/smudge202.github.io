---
layout: post
title: Clean Space - Developer's Worst Enemy
hidden: true
wip: true
tags: [clean-space, game-development]
---

Whether you're a game developer, or write software for large corporate enterprises, there is one frustration we all share. The end user always finds a way to use the software in such a way that it breaks or doesn't follow the behaviour we intended. It's never the user's fault of course, our failings in UI/UX design, our missing restraints and misunderstanding of user intent all point to the developer being at fault. Of course, this only makes the situation all the more frustrating, and more often than it perhaps should, leads us to venting at the perceived _stupidity_ of our user base.

![funny drinking water](http://images-cdn.9gag.com/photo/aM1gyO6_460s_v1.jpg)

_Image from [9gag](http://9gag.com/)_
{: style="font-size: 12px; text-align: center;"}

In this article, I'm going to examine a [user story](https://en.wikipedia.org/wiki/User_story) for _Clean Space_, which I hope will demonstrate just how much thought should be put into these scenarios, whilst providing amples hints towards game design aspects.

## User Story

The example we're going to look at today is pretty simply, on the face at least. 

> As a player of Clean Space<br />
> I want to be able to **name** my spacecraft<br />
> So that other players can recognise my vehicle

Easy, right? Just store some names in a database somewhere, track it against an entity ID of some kind, display it in the UI. Job done.

### Wrong

Certainly, I could take the above approach. In fact, many might argue I _should_ take that approach and iteratively improve upon that design as further requirements crop up. However, for the sake of pragmatism, a software developer will often find themselves trying to walk the thin line of balancing _naviety_ versus _future proofing_. Both are equally bad for a product.

Fortunately, I have a fair amount of knowledge to draw upon from my commercial experience here, so the first thing is start _fleshing out_ the requirements. I have to constantly swap between my _Product Owner_ and _Developer_ hats given this is for the most parts, a one-man-team for now. However, let's step through each of the requirements which, after hours of heated debate infront of a mirror, I've managed to agree with myself.

### Requirements

Let's start by getting the seemingly obvious ones out of the way. Being technically minded, the most immediate constraint that comes to mind are that the vehicle names should be _unique_. Which begs the question, what should we do when a name is not unique? I could of course tell the user _"tough luck, that name has been used"_, but I can think of several cases where that would be a shoddy experience, at best, for the end user. What if the name was used on a vehicle that belongs to another _faction_/_race_? What if the name was used on a vehicle that has since been de-commissioned or destroyed? What if the vehicle that originally had the name, has since been given a new name?

Examining these cases can be laborious, but in doing so we should discover features we hadn't previously considered, and can decide whether or not that feature should be included or not. Product Owners I've worked with can attest to the fact that I often play devil's advocate during sprint planning sessions. I don't _dislike_ a given story, nor the product owners themselves; they've come to understand (I hope!) that I don't question the stories so pendantically as any attempt of malice, simply as a prudent matter of professionalism. I'm not trying to complicate things, I'm trying to offer considerations and side effects that may have been overlooked. Whether they are addressed or not immediately is a question for the product owners, but as someone that takes pride in my code, I need to consider all the states my software might be exposed to, to ensure it doesn't collapse upon itself the second it's exposed to an end user. We'll never foresee all possible eventualities, but that doesn't mean we shouldn't give them some thought!

With that in mind, let's take a look at the requirements, features, and game content we can derive from that _simple_ unique constraint. 

_The following sections use [cucumber](https://cucumber.io/)/[gherkin](https://github.com/cucumber/cucumber/wiki/Gherkin)/[specflow](http://www.specflow.org/) syntax which is a great way to communicate between product owners and development teams_.  

#### Golden Path

```cucumber
Scenario: Name is unique
Given the name "HMSC Enterprise" has not been used before
When a user tries to name their vehicle "HMSC Enterprise"
Then the vehicle name "HMSC Enterprise" is allowed to be registered
```
_NB: As I've mentioned previously, I've tried very hard to distinguish space flight from the traditional naval perspectives that Sci-Fi generally depicts. Therefore,_ "HMSC" _stands for "Her/His Majesty's Space Craft", and hopefully no_ Clean Space _article uses the word "ship"_. 

I like to always start with a _golden path_ requirement, using the simplest possible case in [TDD](https://en.wikipedia.org/wiki/Test-driven_development) to define some code structure. That code will inevitably change as further requirements are introduced, but I'd rather have something wrong _on paper_ than have nothing at all written down.

It's important to note that my assertation (the _'Then'_ section of the scenario) does not state that being Unique sets the vehicle's name; this is on purpose. I'm not trying to complicate matters, simply trying to leave the scenario _open_ to further requirements so that when I start, for example, adding in-game _costs_ to name registrations, I don't need to come back and edit all previous scenarios. This scenario will still hold true, and I can _spec_ the cost requirements in isolation.

With the _win_ scenario out of the way, let's start examining _hiccups_ in our design.

#### Title Transfers

The next scenario inevitably needs to look at how to handle the case of a vehicle name not being unique. As mentioned earlier, I've decided that the name only needs to be unique within the vehicle owner's faction; it would be daft if I couldn't register a name because a faction I'm at war with has already used it. Let's assume however that the name has been used by someone in our own faction. How should we deal with that?

At this point, I could of course display the typical _"Sorry, that name is already used"_ message, but I've decided to introduce a game mechanic to make resolving these conflicts more interesting. These name's can instead be considered [_intangible assets_](https://en.wikipedia.org/wiki/Intangible_asset) held by a player, which can be traded and negotiated over. With this in mind, let's look at the _non-unique_ scenarios.

```cucumber
Scenario: Name has been used by same faction without transfer authority
Given the name "HMSC Enterprise" has been registered to the "Earth" authorities by the player "Incurus"
And "Incurus" has not given authority to use the name "HMSC Enterprise" to "Smudge"
When the player "Smudge" tries to name their vehicle "HMSC Enterprise" in the "Earth" authority
Then the vehicle name "HMSC Enterprise" is not allowed to be registered
And "Smudge" is presented with the option to negotiate with "Incurus" for the name "HMSC Enterprise"
```

_NB: In case you wonder why names are repeated across all the steps, it's because each line of a scenario is a method call by the test runner. I prefer to have the underlying code work in an almost [RESTful](https://en.wikipedia.org/wiki/Representational_state_transfer) manner._

As you can see, there's a lot more moving parts here. Despite the introduction of additional mechanics/features and subsequent complexity, our original scenario still holds true. We've managed to introduce these mechanics within the scope of this scenario without digging into further complications regarding having multiple "_authorities_" (or _registrars_ if you prefer), nor defining any of the requirements for negotiating. We still have a couple more areas to cover in this article so I'll refrain from adding the dozens of additional scenarios we still need to cover regarding the _uniqueness_ of a vehicle name.

All of this might seem a lot of work for little reward but, in my opinion, this is what is required in order to _defeat_ those _nefarious end users_! These practices and thought processes are by no means restricted to game development, they are part of the software development profession across all industries. And these considerations are by no means the whole story...

### Malicious

I'm old enough, just about, to remember a world without the internet. For all the benefits the _age of information_ has brought us, it's hard not to reflect upon just how poorly people have _changed_ (or _revealed_?) in their behaviour. Given the premises of _free speech_ and the physical disconnect of communications, not to mention the seeming anonymity, _bullying_, _trolling_ and all manner of malice and malevolence seems to have taken hold. This hostility is painted with a pretty broad brush, so software developers not only have to consider the security of their products, but also the impact users may have upon each other.

As is common practice, I won't go into any detail regarding _Clean Space_ security practices except to assure you that they are both present, and at the forefront of development considerations. However, we can discuss game design intended to _protect_ users from one another.

#### Community Moderation

It's commonly understood nowadays that some people just enjoy annoying, angering, and _abusing_ others. Personally, I think it's a despicable _hobby_, but each to their own. Software developers often find themselves adding content filtering to their products, preventing swear words being used as usernames for example, and an increasing number of games seem to be incorporating _report_ mechanics that allow players to notify the developers when something is awry. Every software developer has come across, if not used extensively, the services of [Stack Overflow](http://stackoverflow.com/). This is **the** epitome of _community moderation_.

For those that aren't aware of the _Stack Exchange_ mechanics, they are in essence:

* Each User is assigned a _reputation_.
* User's can increase their _reputation_ by moderating the content of others.
* User's can lose _reputation_ if their content is flagged.
* Increased _reputation_ unlocks further functionality and tooling designed to assist with moderation.

There's a bit more to it than that, but that covers the basics. It's an incredibly useful tool because, especially given the site's [traffic](http://stackexchange.com/sites?view=list#traffic), there is no need to maintain (or pay for) teams of moderators. The users manage themselves and the content of the site.

Whilst _Clean Space_ could never in my wildest dreams draw more than a tiny fraction of that traffic, every aspect of user input is going to be subject to _community moderation_. The time spent on providing these features is not insignificant, but compared to the ever increasing amount of time that would be required to moderate content, it's a price I'm more than willing to pay. Hopefully you can see where I'm going with this, but let's get back onto the subject of naming spacecraft.

#### Name Moderation

I mentioned before that registering a vehicle name would incur an _in-game cost_. I'm no more an expert on _economy_ than I am on _celestial mechanics_, but my limited understanding and observations of [_Eve Online_](https://www.eveonline.com/) which is the premiere example of a virtual economy, means I spend plenty of time considering the ramifications of in-game transactions. The subject will require several articles unto itself in the future, so for now I'll glaze over the details.

In this case, the _cost_ of registering a name is not simply intended as a [_gold sink_](https://en.wikipedia.org/wiki/Gold_sink). The best way to encourage players to perform tasks such as _content moderation_ is to offer an incentive. In _Clean Space_, there can be considerable amounts of time spent doing _nothing_ whilst a spacecraft travels vast distances, which to make the game interesting, means I need to provide ample _distractions_. I could of course use some form of _mini-game_ but, it makes a great deal more sense to  incentivise these _maintenance_ tasks and keep the game running smoothly.

Most interactions that require server-side computing in _Clean Space_ entail an in-game cost, and these costs can be increased so that portions of the amount can be offered to moderators. Whilst such mechanics are atypical in game design, they are common place in the real world; consider for example getting a new Passport - we pay for someone to process our application. In addition, by controlling and monitoring the influx of game currency, I can protect and predict server-side resource requirements, whilst providing an _immersive_ and _self-moderating_ gaming experience.

Therefore, there are at least 3-steps to not only registering a vehicle name, but a great many user interactions in _Clean Space_:

1. The name must pass **system validation** (be unique or have negotiated a _title transfer_).
1. The player must have **sufficient funds** to place into [escrow](https://en.wikipedia.org/wiki/Escrow).
1. The name must have passed **community moderation**.

These 3 high-level requirements are helping to bring about uniformity and consistent UI designs. Due to the vast distances involved in space, most interactions entail _delays_ (as information is transmitted across space). Whilst it can be a nuisance at times, it means introducing [_eventual consistency_](https://en.wikipedia.org/wiki/Eventual_consistency) mechanics to other areas of the game don't _stick out_, and offer all the technical benefits of _distributed computing_.

## Summary

I started this article by explaining how much of a _nuisance_ users can be. Whilst that may still hold true in the case of _trolls_ and _hackers_, look at all the game content we've derived from these considerations. I don't think for a second that people won't find ways around these mechanics and constraints, but we'll have headed off a great number of them at the pass. More importantly, we've introduced dozens of features to improve the experience of users who just want to get on and play the game.

This thought process and subsequent practices don't just apply to game design. In my experience, developers as a whole would do well to think more closely about the user experience. 