---
layout: post
title: Comprehensive, Practical Guide to Model-View-Presenter in C# Winforms
hidden: true
wip: true
---

Several months ago, having scoured search engines for hours upon hours for guides, examples, repositories, and general information relating to the Model View Presenter (MVP) pattern, applicable specifically to .Net Windows Forms Applications (WinForms), I came up short. The company I had just taken a role with used WinForms extensively, and any roadmap for moving the company to much more familiar Web and Cloud technologies would have no doubt failed. Sans resource material, I soldiered on [from mud, through blood to the green fields beyond](http://www.royaltankregiment.com/en-GB/regcolours.aspx), placing me in the seemingly unique position of being able to provide advice and practical information for those wishing to achieve the same.

It seems prudent to, at an early stage in this article, provide the list of requirements to which I was determined to fulfill. If you are not interested in similar requirements, I suspect you'll find the remainder of this article of much less use. I will also mention that several assumptions are made throughout the article as to the skill level of the reader (that's you). I will try to provide links to related resources throughout, but please use the comments section at the bottom of the page for any specific questions.

## Tests

Although business requirements and their respective user stories ultimately drove my work on (what I will affectionately refer to as) the _legacy code base_, there were some key aspects and infrastructure I needed in order to complete my objectives in a professional manner.

The _list_ of **technical** requirements, as will be explained, are much more a _chain_ of requirements. The first link of that chain, and I would hope of any chain of work, were the business requirements. The first step from a development point of view however, or the second link in aforementioned chain, were the tests. My team and I were able to translate the user stories into _specs_, or more specifically, a [feature file in specflow](http://www.specflow.org/getting-started/).

Despite a couple members of the company having experience with specflow specifically, let alone similar tools, this was the first time my company had embarked upon such a venture. Credit where credit's due, developers had at various stages prior to my arrival attested the value of unit tests, and several of the soluitions had numerous tests. However, despite comprehensive build and deployment automation through use of Team Foundation Server (TFS2013), none of the tests were run. I know this, because when I first attempted to run the unit tests in Visual Studio, not only did huge portions of them fail, but many attempted to access databases, files, network shares, third part interop, etc. They were beyond useless; if you do not run your tests, you do not trust your tests. If you do not trust your tests, there is no point in having them.
