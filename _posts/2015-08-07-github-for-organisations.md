---
layout: post
title: Contributing - GitHub for Organisations
hidden: true
---

This article is intended to guide you and your company through the process of setting up and making contributions to open source projects, as well as hosting your own repositories. The intent of this particular setup is to accredit and allow acknowledgement of a developers' work, but retaining a control of what is published into the wilderness.

It is a process I have used successfully in the past, and although might seem long-winded as presented here, once you are accustomed to it, it is very fluent and simple to follow. The guide itself will walk you through elements relating exclusively to [GitHub](http://github.com), but the concepts I'm sure can be carried over to alternatives so as [BitBucket](http://bitbucket.org).

_Please note, I natively type in **British** English but will throughout this article be referring to steps in GitHub which uses **American** English. I can but <del>apologise</del> apologize for the mix of spellings herein._

## Setup

These first few steps you'll want to be carried out by a development manager or lead developer, because this person will inherit administrative rights for your repositories. Once you've selected a good candidate to continue with this setup, hop on over to GitHub and either sign in, or [create a new account](https://github.com/join) if you don't have one. This account should be for the **individual** but could be a generic managerial account for which the details can be passed on in the future (there are ownership transferral mechanisms available if you choose to go with an individual account).

Once you're all signed up and logged in, find the organizations section in your account settings or [click here](https://github.com/account/organizations/new) to begin creating an account for your organisation. For most companies, the default "Open Source" free plan should be sufficient to get you going, though it's worth noting that other plans are available should you require them in the future.

The wizard is self-explanatory so I won't mollycoddle you through it. Once you have created your organisation, I recommend getting your teams set up. You'll need to ask your developers for their GitHub usernames. I recommend allowing your devs to use their **personal** usernames; enforcing some corporate based schema really doesn't make sense here and reduces the value of carrying over accounts should they move on from the company. Similarly, you may find that some developers have no interest in contributing to open source projects, which unless your actual company workflow requires, I would be reluctant to make mandatory.

## Contributing

Most companies very regularly use Open Source software in the shape of [NuGet](http://nuget.org) packages. Have you ever found a bug in your dependencies or missed a critical feature? Have you ever been forced to download and tinker the original source to fit your circumstance?

Well it is time at last to start giving back to those from which your company has built upon. Thanks to a [recent/upcoming change to Nuget packages](https://github.com/aspnet/Announcements/issues/46) it is becoming even easier to locate the source of your packages, but usually a quick google such as [`xunit github'](https://www.google.co.uk/#q=xunit%20github) will achieve the same result.

Once you've located the repository containing the source, hit the _"fork"_ button in the top right, and choose your organisation for the destination. Voila, your organisation now has a fork of the code.

Although the process of making changes may vary slightly for you, here is a step by step guide of how we achieved it:

1. Once you have come to an agreement that resources can be allocated to work on the open source project, developers should log in to GitHub and fork the project for themselves.
1. The developer can now clone their own fork and begin development.
1. Follow whatever your standard practice is for developing, committing changes to the developer's own fork.
1. When the developer believes they have completed the work, create a pull-request from their own fork, to the organisation's fork.
1. At this stage you can follow any standard code review practice you have in place, making sure the code meets your company's standard.
1. Many open source projects have their own guidelines and standards for contributions. Look for a `Contrib` file at the root of the repository or check the `ReadMe` if one is present.
1. Once it is agreed that the code is good enough, you can merge the pull request into the organisation fork, and then open a pull request from the company's fork, to the original repository.
 
All being well, your pull request will be accepted and the commit flow will clearly accredit both the company for the pull request, and also the developer(s) who worked on the code.

Going through these steps for your first few contributions may require you to refer back to this guide a few times, but it should quickly become quite fluent. You may decide that the process can be streamlined, some of the control and responsibility levied back upon the developers instead of line management. In any case, I hope this article helps you get the ball rolling!
