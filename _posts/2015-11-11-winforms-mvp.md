---
layout: post
title: Comprehensive, Practical Guide to Model-View-Presenter in C# Winforms
hidden: true
wip: true
---

Several months ago, having scoured search engines for several hours for guides, examples, repositories, and general information relating to the Model-View-Presenter (MVP) pattern, applicable specifically to .Net Windows Forms Applications (WinForms), I came up short. The company I had just taken a role with used WinForms extensively, and any roadmap for moving the company to much more familiar Web and Cloud technologies was going to be a long road. Sans resource material, I soldiered on [from mud, through blood to the green fields beyond](http://www.royaltankregiment.com/en-GB/regcolours.aspx), placing me in the seemingly unique position of being able to provide advice and practical information for those wishing to achieve the same.

It seems prudent to, at an early stage in this article, provide the list of requirements to which I was determined to fulfill. If you are not interested in similar requirements, I you may find the remainder of this article of much less use. I will also mention that several assumptions are made throughout the article as to the skill level of the reader (that's you). I will try to provide links to related resources throughout, but please use the comments section at the bottom of the page for any specific questions.

## Tests

Although business requirements and their respective user stories ultimately drove my work on (what I will affectionately refer to as) the _legacy code base_, there were some key aspects and infrastructure I needed in order to complete my objectives in a professional manner.

### Specs

The _list_ of **technical** requirements, as will be explained, are much more a _chain_ of requirements. The first link of that chain, and I would hope of any development work, are the user stories. The first step from a development point of view however, or the second link in aforementioned chain, were the tests. My team and I were able to translate the user stories into _specs_, or more specifically, a [feature file in specflow](http://www.specflow.org/getting-started/).

This is a great practice that the whole team should be involved in, as you will very quickly flesh out both the design and the requirements, making sure you do in fact have all the information you need before starting development; a great practice to prevent starting on some work and then having to shelve it whilst further information is sought.

I don't want to bog this article down with usage info regarding SpecFlow because, for many, this may be an unnecessary first step. We simply use these specs to drive the rest of our work, usually developing with one failing spec driving the current development. By describing the user interactions through familiar _Given/When/Then_ syntax, you start to get a feel for the UI components that will be necessary, and a feel for the UX. You may of course flip this on it's head and create a mock UI or wireframe for your end users to sign off, which would then lead to the creation of your specs. Whichever works for you.

### Unit Tests

With a failing end-to-end test in place, we can start with our unit testing. Again, I'm not going to go into uniting testing each layer in a typical n-tier system; I only want to look at the UI in this article. The first test I go for is that my application is actually going to be able to load my view (whereby the view is my Form). Now I suspect this will sound odd to some, why on earth wouldn'tmy application be able to load the form? Simply because my form will almost always be required to fetch something from my persistence layer, and/or process some logic in my domain. In order to achieve that, and conform with the [Dependency Inversion Principle](https://en.wikipedia.org/wiki/Dependency_inversion_principle), I opt to use [Dependency Injection](http://blog.devbot.net/composition/#dependency-injection-di).

By always having a test that checks the anticipated composition graph of the respective host is valid for my view, I protect myself against changes to composition. I appreciate there's a couple long words there, so time to show some examples.

#### Windows Forms Composition Example

The following example uses the [Compose](https://www.nuget.org/packages/compose) package, which we've extended as follows for WinForms compatibility _(note, we intend to push this change back up to the [repository](https://github.com/compose-net/compose) so with a little luck you won't need to do this)_:

```c#
public class WindowsFormsApplication<Form> : Executable where Form : System.Windows.Forms.Form
{
  public WindowsFormsApplication()
  {
    this.OnExecute<Form>(form => Windows.Forms.Application.Run(form));
  }

  protected override void PreServiceConfiguration(IServiceCollection services)
	{
		services.TryAddTransient<Form, Form>();
	}
}
```

There's quite a lot going on in the above snippet so I'll walk through it briefly. _Compose_ provides these `Execution` classes that can be extended to inherit all the fancy composition features such as dependency injection. By having `WindowsFormsApplication` inherit from `Execution`, we get access to everything _Compose_ has to offer.

In the constructor, we set up our _execution_; the action that will be run when we execture our application. Hopefully the content of the lambda is familiar to you because it's essentially the same code that is included in new Windows Forms Application templates. The difference, and perhaps a source of confusion, is the generic.  Simply, if you specify a generic on the `OnExecute` method, the _Compose_ library will use your composition graph to dependency inject the specified service.

The final detail of note in the snippet is the `TryAddTransient` call in the `PreServiceConfiguration` method override. Here I'm utilising Microsoft's [Dependency Injection](https://www.nuget.org/packages/microsoft.extensions.dependencyinjection) library to create a _self binding_ for the form. Whilst some DI containers assume a self binding for unregistered types, Microsoft's container requires explicit bindings.
