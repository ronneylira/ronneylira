---
title: 2024 03 06 Build Performance With Roslyn
date: 2024-03-06T10:21:39+01:00
draft: true
tags:
  - roslyn
  - dontet-framework
categories:
  - performance
slug: build-performance-roslyn
lastmod: 2024-03-06T10:12:10.756Z
---
[Roslyn](https://learn.microsoft.com/en-us/dotnet/csharp/roslyn-sdk/) has been kicking around for a couple of years now, and it's got some seriously cool tricks up its sleeve for how you build and keep your code in line.

I'm blown away by how these new goodies with .NET aren't just for the new stack (you know, dotnet core and all that jazz), but they're also giving a boost to the old classics we still rely on.

In my current gig, we've got this huge monolithic app that's a real time hog. It takes a solid 15s to clean, a painful 80s to build, and a whopping 5 minutes to load up for the first time. And that's not just because the solution's a beast – it's all the dependencies, caching, and ASP.NET doing its thing too. Talk about putting a damper on the whole code-test-run cycle 😰.

So a few months back, I decided to take matters into my own hands and give our project a makeover with the latest version of the DotnetCompilerPlatform. After diving into the docs, the plan was simple: slap on the packages, give it a whirl around the office, and see what shakes out.

We had to do a bit of tinkering in Visual Studio (grabbed the .NET Compiler Platform SDK & C# and Visual Basic Roslyn compilers), and then got to work installing the [DotnetCompilerPlatform](https://github.com/aspnet/RoslynCodeDomProvider) NuGet package in our projects.

The initial results on my machine were pretty promising: build time went from 80s to 74s (not earth-shattering, but hey, progress!), and that agonizing first page load time went from 5 minutes to a much more manageable 2 minutes.

I roped in some of my buddies to test out the new setup on their rigs, and the numbers were eye-opening:

On average, build time is about 1.04x faster, and that dreaded first page load? A whopping 2.6x speed boost.

Talk about a game-changer. My only regret? Not jumping on this bandwagon sooner.