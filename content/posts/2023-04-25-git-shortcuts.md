---
title: Git Shortcuts
description: ""
date: 2023-04-25T08:59:47.770Z
preview: ""
draft: false
tags:
  - terminal
  - git
categories:
  - level up your terminal
lastmod: 2023-04-25T15:07:31.888Z
slug: git-shortcuts
---
In this article, I'm going to cover handy shortcuts that can make your life in using git in the terminal faster and more convenient.

I really love using [Windows Terminal](https://github.com/microsoft/terminal), it allows you to use any type of shell that you want, [customize](https://ohmyposh.dev/) it, and [extend it](https://github.com/PowerShell/PSReadLine) to the limit of your own imagination. I cannot recommend enough the great article from [Scott Hanselman](https://www.hanselman.com/blog/my-ultimate-powershell-prompt-with-oh-my-posh-and-the-windows-terminal) that teaches you how to unlock most of the potential within Windows Terminal, really making it your own.

Once you get used to working with the terminal is pretty hard to go back, the speed at which you are used to writing the commands and your own shortcuts make it a flow-like experience to daily work.

A few of the things that can help you with your daily work are git shortcuts

#git shortcuts
Git shortcuts are just another name for the [git configuration file](https://git-scm.com/docs/git-config), this being a global git config setup that essentially dictates how git should behave on your dev box.

the only thing you need to get started is to open your terminal and type in a command like this one 
```shell
git config --global alias.co checkout
```

this will append a configuration setting in your .git config file and will allow you to replace the ol' 

```shell
git checkout BRANCH_NAME
```

for a simpler 

```shell
git co BRANCH_NAME
```

that might seem small, but you are saving 6 characters multiple times a day. Measure that in a year, or even your professional life.

likewise, other commands can be simplified. Here is a list of the ones I use:

- to create a branch
```shell
git config --global alias.br branch
```
- to commit
```shell
git config --global alias.ci commit
```
- to review your current status 
```shell
git config --global alias.st status
```
- to check the last commit
```shell
git config --global alias.last 'log -1 HEAD'
```
to cherry-pick a commit
```shell
git config --global alias.cp cherry-pick
```
  
if you also unlock the power of [WSL](https://learn.microsoft.com/en-us/windows/wsl/) you can extend git (and Windows for that matter) with the help of Linux commands and applications in one seamless shortcut:
- to prune and delete old **local** branches
```shell
git config --global alias.purge '!git fetch -p && git branch -vv | awk "/: gone]/{print \$1}" | xargs git branch -d'
```
with the help of this shortcut
```shell
git prune
```

you automatically fetch and prune remote branches, list them, send it to Linux's [awk](https://www.gnu.org/software/gawk/manual/gawk.html) to filter out only the pruned ones, and then feed those to the delete command.

just make sure that you have git, wsl, and a distro with awk installed in your box to make use of this shortcut.

for today that's it. thanks for reading.