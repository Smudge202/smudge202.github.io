---
layout: post
title: Clean Space - The Moon does NOT orbit the Earth
hidden: true
wip: true
math: true
tags: [clean-space, game-development]
---

In this article I'm going to delve into some of the science behind the exploration aspects of _Clean Space_, and perhaps tackle a couple misconceptions that you and I may share, such as the title of this article. Nothing I'm presenting here is _ground-breaking_, but I hope you find some of the facts interesting as we recap ourselves on those awful trigonometry lessons or physics classes of your past! Don't be put off, I literally had no formal education beyond Secondary School in the UK (10th grade - or _sophomore_ I believe it's called - in the US), so there should be nothing here most layman can't grasp. I'll apologise now, because that same education (or lack thereof) will no doubt account for some level of ignorance and subsequent mistakes in this article; if the actual smart people fancy lending me some pointers in the comments section, I'd be very grateful!

![Moon orbiting the Earth](https://upload.wikimedia.org/wikipedia/commons/thumb/1/1d/Moon_trajectory1.svg/800px-Moon_trajectory1.svg.png)

_Moon orbiting the Sun, locked to the Earth. [Image from Wiki](https://commons.wikimedia.org/wiki/File:Moon_trajectory1.svg)_

As is hopefully evident from the image above, the [Earth-Moon system](https://en.wikipedia.org/wiki/Orbit_of_the_Moon#Path_of_Earth_and_Moon_around_Sun) is better considered as a [_binary planet_](https://en.wikipedia.org/wiki/Double_planet) orbiting the sun than the more typical images depicting Moon's orbit of Earth. The acceleration of gravity applied to the Moon by the Sun(~0.00596m/s$^2$) is actually twice that of the force applied to the moon from Earth (~0.0026967m/s$^2$).

## Clean Space

For those of you new here or whom glazed over the [_Clean Space Introduction_](http://blog.devbot.net/clean-space-introduction/), _Clean Space_ is the game I'm working on. I've been defining the game in day dreams and thought for at least a decade, but finally decided I had the capacity to make it a reality about 6 months ago. I've barely scratched the surface for game content so far, but the planning is well underway and there are some substantial amounts of code tucked away in private repositories as I work through the early stages. 

I have built a great support group so far to assist with design decisions, discussions over theoretical physics, and have several people keen to help out with (or at least peek at) the codebase. Despite a great deal of advice to the contrary (and apparent lack of sane judgement), my _first_ game is set to be a **massive** undertaking. When asked, I tend to repeat that I don't believe the game will even be in demo stages for at least another 24 months. However, I plan to share as much of the journey as I can through blogs like these, and the odd technical demonstration.

## Goal

I'll begin by describing what it is I set out to achieve here, and as best I can, describe the journey I took. I knew when I first decided on the scale and plausibility of _Clean Space_ that I would be spending an awful lot of time learning and not enough time coding. This particular subject was probably one of the most difficult to overcome so far, but puts me in great stead for future challenges by providing a much improved foundation of orbital mechanics (that's rather important in a space game), and also ~~adjusted~~ corrected my expectations for the future.

In short, I wanted to flesh out the technical and scientific realities of exploring a _stellar system_. So yeh, there's the first fun fact: 

  > Our **solar system** is the ONLY **solar system**; everything else is a **stellar** or **star** system.

_NB: I actually refer to other systems as_ **star** systems _normally (since learning the above), but calling them_ **stellar** systems _hopefully caught your attention. Our_ star system _is named for our star, whose Latin name is "Sol" (Solar)_. 

Of course, we tend to _confuse_ ourselves with a naming quite a lot (the "moon", the "sun" and so forth), but _correcting_ those neural pathways of our childhood (and the adult life of a layman) can be trickier than you might think. If you continue to read these articles, prepare yourself for that particular aspect. Whilst we can _intellectually_ accept these corrections and are often already aware of them, it takes a long time before it becomes _natural_. The number and size of differences between the over-simiplified lessons of our youth and subsequent assumptions are truly staggering. Don't be at all surprised though if I misrepresent something because I've fallen pray to the more natural tendencies myself!

## Science 

While we've made incredible progress regarding exploration of our own _Star System_ here on Earth over the last few millenia, we haven't actually explored all that much of the _Solar System_. Being a space game and the limited progress of our own spacefaring capabilities (though a massive thanks and high-five to efforts such as [_SpaceX_](http://www.spacex.com/)!), one of the hardest aspects of developing _Clean Space_ is balancing between real world physics, and the theoretical science that the game lends itself to. I've played a **lot** of games, and a great deal of those have been [4X games](https://en.wikipedia.org/wiki/List_of_4X_video_games). In that time, I've seen nothing come as close to real world science as _Clean Space_, except perhaps [Kerbal Space Program](https://kerbalspaceprogram.com/en/) (though orders of magnitude differences in _scale_).

There are of course various [space simulators](http://mashable.com/2012/08/08/space-simulators/#aJUYqf0WHOqK) that better resemble the science we're trying to achieve here, though through very different approaches in that they are exactly that, _space simulators_, not so much _games_.

I'm going to be intentionally ambiguous in some areas regarding the game design (and plans) for _Clean Space_. I'll give away plenty of hints throughout this article, but I want to hold some details back for some fun surprises down the line, plus retain freedom to go back and change any decisions made thus far. However, to get the ball rolling, let's postulate that through whatever technological miracle, that a spacecraft of Human (or _Terran_) design finds itself in an unexplored star system with sufficient capacity to move about said system in a timely manner. It is on a scientific mission to ascertain as much detail as possible about the celestial bodies it can discover within the system to improve upon observations that may have been made across the vast distance between this unexplored system, and the craft's originating star system.

Let's also assume, that the _unexplored_ system in question (through the perspective of the vehicle's owners), is the _solar system_ (i.e. the _star system_ you and I are in right now). That isn't my giving away lore for _Clean Space_, it's simply so that we can compare assertations and use [real world examples](https://en.wikipedia.org/wiki/Specification_by_example) to confirm the findings of this exploration vehicle.

### Gravity

I'm not going to try to teach people to suck eggs, but to ensure my readers (you) have the basics under wraps, I'll very briefly mention gravity.

> The force that attracts a body towards any other physical body having mass.

The greater the mass, the greater the gravity. We actually tend to refer to that resulting force as an _acceleration_, so that the force of gravity on the surface of Earth, for example, is 9.78 to 9.83 m/s$^2$ (meters per second per second). The force of gravity is **incredibly weak** despite the profound effect it has on our day to day lives, which might explain why it's so difficult to measure the [_gravitational constant_](https://en.wikipedia.org/wiki/Gravitational_constant).

This is perhaps an interesting subject; when I began working on _Clean Space_ I simply picked the first value for the gravitational constant I could find on wiki, and got on with my life. I recently revisited the fundamental physics of the game engine ([Clean Engine](https://github.com/clean-development/engine)) and was dismayed by the huge number of supposed _constants_ I was coming across, leading to a very interesting [article by Lisa Zyga](http://phys.org/news/2015-04-gravitational-constant-vary.html). For the sake of the remainder of this article however, as interesting as toying with the idea of a variable gravitational _constant_ might be (that's a hint towards a future article), we'll use the value `6.673889E-11`. 

_NB: For those that have forgotten how exponents work, the_ `E` _can be replaced with $$\times 10^x$$, or simply move the decimal point the given number of digits either to the right for positive, or left for negative exponents._

### Law of Cosines

So, our exploration ship has attained orbit around Earth. It's sensors have located a satellite body (the Moon), and it's time to get our geek on. So, what information can we get from observing these two objects?

![spacecraft orbiting earth](https://s3-eu-west-1.amazonaws.com/pub.sketchboard.io/images/56764aa8-6a9e-4e0b-b4bf-873558e164f0.png?updated=1469416250650)
_<sup>NB: Clearly not to scale</sup>_

Well, first and foremost, the spacecraft is able to get the distance from itself to the Earth and the Moon (even modern day technology can achieve this so I won't go into detail). From it's position, it should also be able to extrapolate the radius of both bodies which is required to get the distance from the vehicle to the _centre_ of both the Earth and Moon, not simply the distance from itself to the surface of each body as would be measured (the calculation could be done using the surfaces if required, but it becomes slightly more complicated). 

Given these little facts, we can calculate the distance between the Earth and Moon easily enough, using the [law of cosines](https://en.wikipedia.org/wiki/Law_of_cosines): $c^2 = a^2 + b^2 - 2ab \cdot cos(C)$ where _C_ denotes the angle between _a_ (the length between Earth and the vehicle) and _b_ (the length between the Moon and the vehicle), opposite from _c_ (the distance between Earth and Moon).

_Note that the case of_ a, A, _etc. is important. Lowercase for a length, uppercase for an angle._ 

You may also notice that the _Law of Cosines_ shares a striking resemblance with [Pythagorean theorem](https://en.wikipedia.org/wiki/Pythagorean_theorem). While Pythagoras' theorem is intended for right triangles, the _Law of Cosines_ works with any triangles, which is made more evident when you realise that $cos(90) = 0$, which means for right triangles, the $2ab \cdot cos(C)$ can essentially be ignored, bringing us back to Pyhtagoras' theorem.

We have at this point the distance between each body, allowing us to use the [law of sines](https://en.wikipedia.org/wiki/Law_of_sines) to get the 2 remaining angles if required (${a \over sin A} = {b \over sin B} = {c \over sin C} = d$ where _d_ is the diameter of the triangle's circumcircle). With the 3 lengths and 3 angles of the triangle between our spacecraft, the Earth, and the Moon, we can begin delving into orbital mechanics.

### Kepler

[Johannes Kepler](https://en.wikipedia.org/wiki/Johannes_Kepler) was a German mathematician, astronomer, and astrologer. He is well known for a number of published works, not least that which contained [_Kepler's laws of planetary motion_](https://en.wikipedia.org/wiki/Kepler%27s_laws_of_planetary_motion). The wiki article linked there is a bit mind blowing at first glance, fortunately there are much [simpler articles](http://www.physicsclassroom.com/class/circles/Lesson-4/Mathematics-of-Satellite-Motion) to reference should you want to follow along. In essence though, the laws provide mechanisms to ascertain:

#### Orbital Speed

The speed at which the satellite (Moon) moves:

{% raw %} $$v = \sqrt{{G \cdot M} \over R}$$ {% endraw %} 

where _G_ is the gravitational constant we discussed earlier, _M_ is the mass of the body being orbited (Earth), and _R_ is the radius of orbit for the satellite body (the distance between the centre of the Earth and the centre of the Moon that we discovered above). See how this is all coming together yet?

_NB: The above only works for a circular orbit, but let's keep this first lesson simple._

#### Acceleration

The acceleration of gravity on the satellite (Moon):

{% raw %} $$a = {{G \cdot M} \over R^2}$$ {% endraw %}

#### Orbital Period

And finally, the length of time it takes for the satellite to complete an orbit:

{% raw %}$${T^2 \over R^3} = {{4 \cdot π^2} \over {G \cdot M} }$$ {% endraw %}

_`π` is [pi](https://en.wikipedia.org/wiki/Pi) for those that may have forgotten._

_NB: It's worth noting that there are a great number of assumptions necessary for the rudimentary calculations above to work accurately, such as the observed bodies being spherical (which isn't always the case in the real world). However, given this game universe is one of my own making, I do have the luxury of enforcing these assumptions as contraints, should I choose._

### Celestial Mechanics

For anyone that's managed to get a vehicle into [Kerbin](http://wiki.kerbalspaceprogram.com/wiki/Kerbin) orbit in the game _Kerbal Space Program_, the basic concepts of orbital manoeuvres have, I'm sure, made themselves aware. If a vehicle is [orbiting a body](https://en.wikipedia.org/wiki/Orbit#Understanding_orbits) (be it a planet, a star, anything), to attain a lower (closer) orbit, you don't _burn_ towards the body, like you might (and I once did) think. Instead, you thurst in the opposite direction to your orbital trajectory which is known as [retrograde](https://en.wikipedia.org/wiki/Retrograde_and_prograde_motion), and similarly thrust _prograde_ to attain a higher (further) orbit. What's more interesting is that, assuming you started in a circular orbit, applying thrust will most effect the opposite side of your orbit, pushing it away or towards the body being orbited (depending on whether your thrust is prograde or retrograde respectively), changing the _eccentricity_ of the orbit either towards a collision (suborbital), or a _parabolic_/_hyperbolic_ escape trajectory.

![prograde thrust](https://s3-eu-west-1.amazonaws.com/pub.sketchboard.io/images/5529bcf1-89d5-4019-a792-063215648814.png?updated=1469707434554)

Therefore, to go from a circular orbit, to another circular orbit, you would typically require 2 periods of thrust. One that changes the opposite side of your orbit, which you then _coast_ around to, before thrusting again to circularise the orbit.

![circularised orbit](https://s3-eu-west-1.amazonaws.com/pub.sketchboard.io/images/cc1b3110-20dd-4496-8cf4-6eb0fc9495e0.png?updated=1469707753596)

## Putting the pieces together

If you combine all of the above, there's a great number of things our exploration spacecraft can observe, measure, extrapolate, and assume. By plotting the orbital trajectory of bodies, we can ascertain the [_gravitational parameters_](https://en.wikipedia.org/wiki/Standard_gravitational_parameter) (μ) of the bodies involved. If the _gravitational constant_ is known, we can extrapolate the mass of bodies. We might even begin to make assumptions of density from observed mass (versus body volume) to take a guess at heavy metal likelihoods.

There are a great number of things we can learn from vehicles such as this. Which leads us to how this applies to _Clean Space_.

## Independent Observation

Whilst _Clean Space_ is based largely on real world physics, I don't intend to force all of the above down the throats of a player. For all the scientific simulation that goes on, I know that this _game_ needs to be much more _game_ than _second job_. At the same time, I want to reward those that do take the time to understand the physics.

So why does all of the above science and laborious equations matter? 

_Actually, the above is a very simple basis to get your toes wet..._

There are 2 reasons. The first, as I'm sure you could guess, is that the game engine for _Clean Living_ uses all this science to simulate the game universe (no pun intended). Using mechanisms like the above (typically using [_functions over time_](https://en.wikipedia.org/wiki/Kepler%27s_laws_of_planetary_motion#Position_as_a_function_of_time) to achieve the performance required), the engine intermingles real world physics with vasts amount of theoretical physics. This is a **difficult** task, and at least in part explains why the development time of the game will go on for years.

However, the second and much more _fun_ reason, is that unlike any game I've **ever** played, _Clean Space_ does not simply _reveal_ these facts about the universe to the player. Moreover, it doesn't reveal these facts to the _AI_!

Think about that for a moment. No, really. Consider the implications.

I mentioned in a [previous article](http://127.0.0.1:4000/clean-space-game/#technicalities) the limitations that would be enforced upon the world with regards to the _speed of information_. That in itself makes for an interesting mechanic that I don't believe I've seen in other games. But this little revelation brings upon a whole new scale.

### Research

The game design behind _research_ in _Clean Living_ is so unique that it'll require at least one article unto itself to fully describe. However, consider for a moment that your typical _technology tree_ isn't rail-roaded. Not only you, but all the AI of your _faction/race_ has but a foundation knowledge when the game begins. It understands all the real world science we know of today, and a limited amount of the aforementioned _theoretical intermingling_.

However, to _learn_ more about the universe, and to increase one's understanding and technology, you can't simply sink some resource into a hypothetical project and after an allotted time have something revealed to you by the game engine. You actually have to explore, make observations that challenge your current [_mis_]understanding of the universe about you. You have to design, fabricate, and trial experimental equipment, and only through subsequent observations and refining of techniques can you hope to learn more about the theoretical physics the sneaky programmer has put into the world.

### Exploration

The only scientific consequence I've come across when exploring in other games is perhaps stumbling across some artifact that provides some form of _boost_ to the explorer. In _Clean Space_, the role of exploration will be like nothing I've seen simulated before.

Forming adequate information to accurately and efficiently perform astrogation is still crucial, more so than you might expect given how forcibly the physics apply themselves, and just how complicated that is even when you have all the facts. Any time a spacecraft finds itself veering off of a projected course, it isn't a mistake in the system; it's an observation of the undiscovered.

## Summary

I truly wish I had the time to put my money where my mouth is, but the real world comes first and we've all got bills to pay. Hopefully, it's evident just how magnificent a task I've set for myself. Whilst I hope the prospect excites you, you're just going to have to hang in there for the evidence of all this posturing to come to fruition.

For the same reason it's much too early to reveal any gameplay footage, it's also the perfect time to submit your ideas. They don't need to be scientifically sound - trust me, mine are dubious at best. But if there's something you'd like to add, something you'd like to see in _Clean Space_, the sooner the better. Drop a comment below or use one of the icons at the bottom of the page to contact me!  
