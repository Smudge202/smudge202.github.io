---
layout: post
title: Dependency Injection in MonoGame
hidden: true
wip: true
tags: [game-development, game-loop]
---

In the [previous article](http://blog.devbot.net/game-loop/) of this [mini-series](http://blog.devbot.net/tag/game-loop/) I talked about the [problems](http://blog.devbot.net/game-loop/#dependency-injection) inherent in [MonoGame](http://www.monogame.net/) (and other Game Engines) that effectively prevent Dependency Injection (DI). During the development of [Clean Space](http://blog.devbot.net/clean-space-introduction) I made several attempts to resolve this particular issue, and in this article I want to describe the first (pretty crude) mechanism I put together.

![yuck](../images/yuck.jpg)

I'll also explore (and *prove*) the performance considerations of making changes like this to MonoGame, whilst contrasting this *dependency injection* pattern to the many [*service locator*](http://blog.ploeh.dk/2010/02/03/ServiceLocatorisanAnti-Pattern/) anti-patterns I've seen used as workarounds.

## First World Problems

So the issue with MonoGame, and I'll keep reiterating that it's not limited to *just* MonoGame, is that there are no abstractions, and many classes have inter-dependencies. That is to say that class *A* depends on class *B*, and class *B* depends on *A*. This problem very quickly rears it's ugly head if you follow the application bootstrapping code, as demonstrated in the previous post.

Throughout this article I'll be working exclusively with the [`dev` branch](https://github.com/MonoGame/MonoGame/tree/develop) of MonoGame, but the [*Universal Windows Platform*](https://docs.microsoft.com/en-us/windows/uwp/get-started/universal-application-platform-guide) (or 'UWP' for short) XAML project is much the same as in the MonoGame 3.6 Stable version. As such, let's recap and explore the application startup of a UWP XAML project.

> *Note: ['XAML'](https://msdn.microsoft.com/en-us/library/cc295302.aspx) stands for 'e**X**tensible **A**pplication **M**arkup **L**anguage' and is an XML-based language designed by Microsoft to describe the visual components of your Application.*

## Startup (XAML)

Each platform MonoGame supports has a slightly different startup mechanism, primarily dictated by the Windows and .NET Framework calls that need to be made in order to get an application up and running. In the case of a UWP XAML application, the first bit of code to run (that we can see) is that contained in `App.xaml.cs`.

Whilst this class contains around 130 lines of code by default (from the project template) most of that isn't too important with regards to Dependency Injection. The only line that really matters to us right now is:

```csharp
rootFrame.Navigate(typeof(GamePage), e.Arguments);
```

During startup, because nothing has been navigated to or displayed yet, the application will transition to `GamePage`. At this point, the code in `GamePage.xaml.cs` will be executed:

```csharp
public sealed partial class GamePage : Page
{
  readonly Game1 _game;

  public GamePage()
  {
    this.InitializeComponent();

    // Create the game.
    var launchArguments = string.Empty;
    _game = MonoGame.Framework.XamlGame<Game1>
      .Create(launchArguments, Window.Current.CoreWindow, swapChainPanel);
  }
}
```

This is the first point at which MonoGame gets involved. Here we can see that it is trying to load the `Game1` class into a `SwapChainPanel` (which the project template has declared for you in `GamePage.xaml`).

Before we take a look at the `Game1` class, let's have a look at the code that executes within the [`XamlGame<Game1>.Create` method](https://github.com/MonoGame/MonoGame/blob/develop/MonoGame.Framework/WindowsUniversal/XamlGame.cs#L28-L56):

```csharp
static public T Create(string launchParameters, CoreWindow window, SwapChainPanel swapChainPanel)
{
  //argument validation...

  // Save any launch parameters to be parsed by the platform.
  UAPGamePlatform.LaunchParameters = launchParameters;

  // Setup the window class.
  UAPGameWindow.Instance.Initialize(window, swapChainPanel, UAPGamePlatform.TouchQueue);

  // Construct the game.
  var game = new T();

  // Set the swap chain panel on the graphics mananger.
  if (game.graphicsDeviceManager == null)
      throw new NullReferenceException("You must create the GraphicsDeviceManager in the Game constructor!");
  game.graphicsDeviceManager.SwapChainPanel = swapChainPanel;

  // Start running the game.
  game.Run(GameRunBehavior.Asynchronous);

  // Return the created game object.
  return game;
}
```

There's a lot more happening here than everything else so far. We can ignore the argument exceptions (excluded for brevity above) and jump straight to `UAPGamePlatform` and `UAPGamePlatform`. 'UAP' is short for [*Universal Apps*](https://msdn.microsoft.com/en-gb/magazine/dn973012.aspx) which is what we called 'UWP' a few years ago when the concept was first introduced. The classes in question actually belong to the `XNA` namespace (not MonoGame) which is a game framework Microsoft stopped developing several years ago before the MonoGame community picked up the reins.

> *Note: 'XNA' is not an acronym. [Greg Duncan explained](https://channel9.msdn.com/coding4fun/blog/ANXFramework-ANXs-not-XNA-but-kind-of) that if anything, 'XNA' stands for '**X**NA's **n**ot **a**cronymed'...*
>
>![not funny](../images/not-funny.jpg)
>
> ...but at least it's one less acronym from Microsoft to try and remember.

The `Initialize` method on the `UAPGameWindow` class is also very involving, not to mention `internal` so is otherwise not visible to us.

Once the XNA components are configured, MonoGame goes on to instantiate our `Game1` class (passed as a generic `T`), then complain (by throwing an exception) if you don't set the `graphicsDeviceManager` property (which is also `internal`...) before calling the public `Run` method on the inherited `Game` class.

The reason I keep highlighting these `internal` members is it means, unless we're writing code in the same assembly as MonoGame, or MonoGame for some reason exposes it's internal members to our code using an `InternalsVisibleTo` attribute, we can only *access* those classes via reflection, and doing so would not be type safe. This is a *huge* problem with regards to extensibility because it makes it either impossible or very *unsafe* to extend and change behaviours.

![not happy](../images/happiness-challenged.jpg)