---
layout: post
title: Managing Action Conflicts in ASP.Net 5 with Swashbuckle
hidden: true
wip: true
---

For several weeks now we've been using [Swashbuckle](github.com/domaindrivendev/swashbuckle) in our [ASP.Net 5](https://get.asp.net/) Web API to assist document and even test our API endpoints. This is a brand new API, and we've been doing everything we can to follow best practices with regards to Web API's. Our discussions and decisions regarding best practices are always focused upon those practices, and not necessarily upon the technical implications of an outcome. Thanks to the superb extensibility of *ASP.NET 5*, this hasn't been an issue until today, when we came across an issue that [many others](https://github.com/domaindrivendev/Swashbuckle/issues/142) also seem to have had problems with.

> Request matched multiple actions resulting in ambiguity for actions with different parameters

