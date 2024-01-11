---
title: "NU5026: The file to be packed was not found on disk"
description: ""
date: 2023-10-30T10:50:15+01:00
draft: false
tags:
  - dotnet
  - devops
  - cd/ci
categories:
  - devops
  - pipelines
slug: NU5025-error-the-file-to-be-packed-was-not-found-on-disk
lastmod: 2023-10-30T10:48:17.259Z
---
For a long time, our company relied on MyGet as our private package repository. However, a recent outage caused a 24-hour delay in our work, so we've decided to stop using MyGet and switch to GitHub Artifacts as our new provider.

This situation presented us with an excellent chance to enhance the quality of our pipelines by implementing some standard tools and protecting us from the risks associated with broad access to these resources.

One of the steps we took was transitioning from Azure DevOps classic pipelines to reusable YAML pipelines. However, a challenge arose when these new pipelines, which were responsible for building NuGet packages, began to encounter an error that we couldn't replicate either locally or in the old pipelines.

```powershell
Build FAILED.
"/home/vsts/work/1/s/NewPackage/NewPackage.csproj" (pack target) (1:7) ->
(GenerateNuspec target) -> 
  /opt/hostedtoolcache/dotnet/sdk/6.0.416/Sdks/NuGet.Build.Tasks.Pack/build/NuGet.Build.Tasks.Pack.targets(221,5): error NU5026: The file '/home/vsts/work/1/s/NewPackage/bin/Debug/netstandard2.0/NewPackage.dll' to be packed was not found on disk. [/home/vsts/work/1/s/NewPackage/NewPackage.csproj]

0 Warning(s)
1 Error(s)
```


After several rounds of testing and investigation, we successfully identified the root cause. In the .csproj file, we had previously set a useful flag that generated the package during the build process:
```xml
<GeneratePackageOnBuild>true</GeneratePackageOnBuild>
```
this is being tracked in an [issue](https://github.com/dotnet/sdk/issues/10335) in the dotnet sdk repo.

To avoid package generation in the classic pipelines, we had decided to perform it during the build process. However, with a dedicated pack task, this approach was not functioning as anticipated. So, we removed the tag, and voilà, it's now packing correctly.

As we adapt and grow, we continually discover new solutions and uncover unexpected challenges. Remember, every obstacle presents an opportunity for growth and improvement. I hope this post helps you.