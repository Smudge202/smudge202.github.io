---
layout: post
title: Logging - Good Logging Practices to locate Undisposed Objects
hidden: true
wip: true
---

I tweeted a thank you to the guys over at [Seq](https://getseq.net/) recently because _Seq_ was instrumental in finding and eliminating a very difficult issue I came across. But technology choice alone isn't enough, so I've decided to blog about some of the logging practices I tend to follow as a rule, and some specific practices that I used in a particularly complex application I've been developing over the last couple of months.

<blockquote class="twitter-tweet" data-partner="tweetdeck"><p lang="en" dir="ltr">Logging best practices and <a href="https://twitter.com/getseq_net">@getseq_net</a> have been an absolute god-send for tracking down tricky problems. Much love. <a href="https://t.co/6fubnhSnOo">pic.twitter.com/6fubnhSnOo</a></p>&mdash; Tommy Long (@Smudge202) <a href="https://twitter.com/Smudge202/status/816749326642847749">January 4, 2017</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

## Foreword

This is going to be a lengthy post, and several of the practices shown are circumstantial, subjective, situational, and so forth. Add these practices to your developer toolbelt, and decide for yourself when they best fit your needs.

The system I've been working on is a series of commercial applications subject to standard IP clauses, so I've included as much code as I can in this post, wherever possible sans abstraction and obfuscation, but I won't be able to provide the applications themselves as samples.

The system has 4 microservice styled applications which communicate over a service bus. The system is far from being complete, but one of the founding requirements was to replace a legacy application. As such, several of the legacy practices were brought forth into the new system with the intent of iteratively improving, and where necessary, replacing existing mechanics. As such, some of the code may not be entirely pleasant but, trust me when I say the requirements made it necessity.

The code is primarily contained within 15 .Net Core libraries targeting the full .Net Framework (4.5.2) due to dependency requirements. 4 of those libraries are what we call _application hosts_, containing no business logic, designed to host a selection of the libraries as a .Net Core Console Application which can be deployed inside [Docker](https://www.docker.com/) containers, as a Windows Service, or in Azure. (This is a practice our company tends to follow and has a collection of composition helpers for, so that the applications can be ran anywhere, leaving deployment considerations to our IT department.)

In addition to the above, we have a considerable amount of heritage code in other solutions which is NuGet packaged and deployed by TFS/Octopus, which is then pulled down by one of our microservices at runtime and installed/executed within separate `AppDomain`s. As you might imagine, with so many moving parts, logging is essential.

