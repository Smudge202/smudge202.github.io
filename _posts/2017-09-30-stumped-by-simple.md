---
layout: post
title: Stumped by Simple Technical Test
hidden: true
wip: true
---

I recently went back out to market in search of a new role and was quickly put on to a really exciting company. The outcome to my application to this company is not yet determined, but part of the process is to complete what appears at first glance to be a simple technical test. However, having given the test a bit of thought throughout the course of the day, it's actually a *lot* more complicated than it first appears. In fact, I spent quite a while completely stumped as to a satisfactory design and implementation. I'm not sure at this stage whether this was intentional, some sneaky derivative of some known difficult problem that, having never completed college or university I'm unaware of, or if I was just over-thinking matters.

Anyway. I'm going to put to paper, so to speak, my thought process as I walk through this test. I'm hoping it will help me derive, refine and justify a solution, whilst also proving of interest to you. Unfortunately, I won't be able to publish this article without permission from the company that issued the test, but hopefully this article makes the light of day at some point in the future.

![funny stuck cow](../images/funny-stuck.jpg)

## Requirements

In essence, from a pool of 10 resources, the intent is to select 2 resources at random to complete half day shifts, per day, whilst complying with the following rules:

1. A resource can do at most one half day shift in a day.
1. A resource cannot have half day shifts on consecutive days.
1. Each resource should have completed two half day shifts in any 2 week period.

Those were the business requirements (re-worded slightly), as I received them. Immediately I note that the second requirement supercedes and makes superfluous the first requirement. By not allowing the same resource to be allocated to more than one shift in any 2 day period, a resource won't be able to have 2 shifts in a single day. Assuming I have correctly understood the intent of those rules (which in a real world scenario I would seek to clarify), we can safely ignore the first requirement so long as the second is correctly enforced.

## Evolving Design

Let's start with the most obvious, albeit naive approach of simply using a random sequence. Be it `Random.Next` or one of the cryptographic sequences available in .Net, we can easily get a random sequence, and let's assume for now we have a function that can select a resource from our pool of resources given part of this sequence. What are the problems with such a design?

Let's label our resources `A` to `J` to give us 10 unique resources. Let's also assume the random sequence we generate provides integers, which allows as to easily zero-index our resources (`0` = `A`, `1` = `B`, `2` = `C`, ... `9` = `J`).

In order to better communicate intent, let's call the first day of a 2 week period *`d`*, the second day of the week being *`d`*`+1`, the third day *`d`*`+2` and so forth. We also have two shifts per day, so let's call the AM shift *`a`* and the PM shift *`b`* (note casing difference to our resources so as not to confuse them). Therefore, the aftenoon shift of the 5th day in a 2 week period we'll describe as *`d`*`+4`*`b`*. If that doesn't make sense to you yet, hopefully it'll become clear as we work our way through this.

The first obvious problem in my mind is that the sequence `0, 0, 0, 0, 0, 0, 0, ...` is essentially just as likely as any other random sequence you can think of. I recall some years ago watching a documentary that showed that a subset of people have been choosing consecutive sequences (`1, 2, 3, 4...`) in the national lottery for as long as it's been around; it's just as likely as any other seemingly random sequence.

We can easily side-skirt this issue by running the business rules when a random number is selected. If the random number will cause one of our rules to break, simply ignore it and take the next number in the sequence. However, in order to test the second business rule (because we can ignore the first) we need to know who has been allocated to the shifts yesterday and/or tomorrow, and in order to achieve the third business rule, we need to consider an entire 2 week period.

If we assume our implementation iterated through the 2 week period, starting at *`d`*`+0`*`a`* (or *`da`* for short), moving on to *`db`*, then *`d`*`+1`*`a`* and so forth, we can easily track the days allocated so far, and enforce rule 2 by simply skipping random numbers if the resource derived from the number has appeared in the last 2 days. In doing so however, it's perfectly plausible that we paint ourselves into a corner:

**Week 1:** | *`da`* | *`db`* | *`d`*`+1`*`a`* | *`d`*`+1`*`b`* | *`d`*`+2`*`a`* | *`d`*`+2`*`b`* | *`d`*`+3`*`a`* | *`d`*`+3`*`b`* | *`d`*`+4`*`a`* | *`d`*`+4`*`b`* | *`d`*`+5`*`a`* | *`d`*`+5`*`b`* | *`d`*`+6`*`a`* | *`d`*`+6`*`b`* |
:-- | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: |
 | A | B | C | D | A | B | C | D | A | B | C | D | A | B
**Week 2:** | *`d`*`+7`*`a`* | *`d`*`+7`*`b`* | *`d`*`+8`*`a`* | *`d`*`+8`*`b`* | *`d`*`+9`*`a`* | *`d`*`+9`*`b`* | *`d`*`+10`*`a`* | *`d`*`+10`*`b`* | *`d`*`+11`*`a`* | *`d`*`+11`*`b`* | *`d`*`+12`*`a`* | *`d`*`+12`*`b`* | *`d`*`+13`*`a`* | *`d`*`+13`*`b`*
 | C | D | A |

As you can hopefully see here, I've allocated the first 4 resources (`A` to `D`) sequentially for the first 8 and a half days, leaving 11 slots as yet unallocated. At this stage though, no matter what comes next, it is no longer possible to achieve rule 3. That is because I have 6 resources (`E` to `J`) that must be allocated to 2 slots each within this period, which would require 12 of my 11 remaining slots.

You could argue that the likelihood of having the `A` to `D` sequencing is very unlikely, and I agree with you. However, I've simply used this sequencing to emphasise the issue. You could randomly sequence 9 of the 10 resources for 24 of the total 28 slots in a 2 week period, and still fall foul of the business rules because you would not be able to allocate the remaining resource into the 2 slots that rule 3 mandates without breaking rule 2.

This is not the only problem either, what about the slots *`d`*`-1`*`a`* and *`d`*`-1`*`b`* (that is, the day *before* this 2 week period started). It's not even visible to our chosen scope, but if those two slots contained resources `A` or `B`, we'd have broken rule 2 without even being able to see it!

![funny concentrating cat](../images/funny-concentrationcat.jpg)

## Changing Tack

The final paragraph in the previous section highlights a pretty significant issue with regards to design of this system. One could argue that we simple add *`d`*`-1`*`a`* and *`d`*`-1`*`b`* to our scopes in order to circumvent the possibility of accidentally breaking rule 2 on *`da`* and *`db`*, but what if *`d`*`-1`*`a`* or *`d`*`-1`*`b`* were forced to skip one or more random numbers due to similar constraints placed on them by either rule 3, or rule 2 due to allocations in *`d`*`-2`*`a`* and *`d`*`-2`*`b`*? If you were to follow that line of reasoning, you would need **every** day in history to know for sure that you weren't going to break rules.

The technical test did mention that I could use any *ORM* I wanted, so perhaps that is the intent? When the system starts, I could let the application sit and churn for hours calculating rotas for all of time. The thought of doing such a thing actually repulses me somewhat, and fortunately I know of a mechanism that, combined with a little creativity, should completely circumvent that necessity.

For anyone that plays computer games, you'll have come across seeded randoms, typically in world generation algorithms like those used by *Minecraft*. The games are capable of generating (either upfront or procedurally, as is the case for *Minecraft*) a random environment based on a seed value. The world is for all intents and purposes completely random, but anyone using the same seed value will also generate the exact same random world.

It's perhaps not very well known, but even .Net's [`Random` class](https://msdn.microsoft.com/en-us/library/ctssatww(v=vs.110).aspx) can take an `int` during construction to act as a seed. This is useful, because if employed correctly, it would allow our random rota code to examine any days outside of our 2 week scope, such as the aforementioned *`d`*`-1`*`a`* and *`d`*`-1`*`b`* slots.

You could still face issues like those described earlier if numbers in the random sequence had to be skipped due to constraints coming from other days outside of our scope, but not if we use multiple phases and introduce *anchor* days.

Imagine the algorithm works as follows:

1. Instantiate the seeded random using the epoch of our anchor day within the 2 week scope (whether that anchor day is *`d`* or *`d13`* is up to you, but the start and end of the 2 week scope make the best candidates in my opinion).
1. Populate the anchor day in accordance with rule 2. That is, the first slot is guaranteed to be filled by the first number in the random sequence because all other slots are empty, then iterate that sequence until a *different* resource is selected for the second slot in that day.
1. Instantiate a second seeded random using the epoch of the period either before or after our scope (depending on which anchor you selected), and calculate the slot allocations that would have been used for that period. We know they follow the sames rules as our anchor day and because they are allocated first and foremost, they only need to account for rule 2, as ours did.
1. Next, focus on rule 3 by iterating the resources (taking into account the 2 slots already allocated to our anchor day). Randomly assign each resource to a random slot within the period, taking into account what we know of the anchor day in the adjacent period, and skipping the sequence if rule 2 would be broken.
1. Once all resources have a 2 slots each within scope ensuring our compliance with rule 3, continue allocating randomly however you want, taking into account rule 2 and the adjacent anchor.

This has the benefit of never needing to store anything. We can generate the rota for any period in the past, present, or future, and be confident that we'll meet our requirements. This part of the overall implementation appears to be immune to requirement changes, such as needing to change the number of resources, length of period, minimum shift count, and minimum allowed adjacency to shifts. Sure, we'll need to make these aspects extensible/configurable, but the algorithm should continue to function as intended if those changes are requested at a later date.

## Tidying up

The final piece of the puzzle is to align our periods to ensure we don't accidentally have 2 week periods overlapping with one another. This, again, is tad trickier than it might seem on first glance. The number of weeks in a year vary, and even the way we number those weeks varies. The accepted format is typically *ISO 8601*, however, the .Net implementation doesn't quite agree with that ISO, as shown [here](https://blogs.msdn.microsoft.com/shawnste/2006/01/24/iso-8601-week-of-year-format-in-microsoft-net/).

We can massively simplify this though. As mentioned above, we have to consider it possible for the business requirements to change the length of the period we're working with. Right now it is 2 weeks, but that doesn't necessarily tie us to operating in weeks (what if the business wants to change the period to 8 and a half days?).

To alleviate this issue, let's take the epoch (which as of .Net 4.6 and .NetStandard 1.3, we can easily pull from [`DateTimeOffset.ToUnixTimeSeconds()`](https://msdn.microsoft.com/en-us/library/system.datetimeoffset.tounixtimeseconds(v=vs.110).aspx)) and perform a modulo operation with the epoch as *dividend* and period length in seconds as *divisor*. This will give us the remainder, which can be deducted from the epoch used to identify an anchor. Adding the period length to that anchor gives us a second anchor, and doing so again can get a third and so forth if required.

Regardless of what period a user might ask for, we can populate rotas aligned to those anchors in order to provide the period requested. For example, if the user asks to see the rotas for a 3 week period (continuing to use 2 week rota periods), we can confidently generate 2 week periods behind the scenes and then limit our results to the period the user requested.

As with the random allocation algorithm, this should also be immune to the requirement changes we identified above.

## Known Issues

Whilst our algorithm will continue to function as intended regardless of the requirement changes identified, if any of our parameters are modified, that algorithm will no longer align periods as it did before, and/or no longer populate the periods in the same manner. Therefore, the application will be unable to fulfill it's requirements because it is unable to predict the location and/or allocations of our anchors, opening us up to the same problems we identified before having introduced those anchors.

I think, for now, that would be an acceptable caveat for the design (given I don't have an actual product owner available to verify that decision in the context of a technical test). We could alleviate the issue by storing any periods we allocate, but I'll leave that as an optional extra I'll skip for the sake of brevity in this test.