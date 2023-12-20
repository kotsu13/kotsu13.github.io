---
layout: post
title:  "Pokemon: Is my Pokemon legit?"
description: "How to tell if my pokemon is hacked"
date: 2023-01-30
img: # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [pokemon]
---

Two of the things my friends and I do on Pokemon S/V pretty often is tera raids and trading of pokemon. Sometimes we trade via the random trade feature, and other times we join public raids hosted on discord servers. As a result of that, I started looking more into Pokemon Automation to find out more about some of the projects. Some more well known ones I looked into are <a href="https://github.com/LegoFigure11/RaidCrawler">RaidCrawler</a> and <a href="https://github.com/kwsch/PKHeX">PkHex</a>. 

A brief summary of each project: 
- RaidCrawler is a brute-force program that helps you to find specific raids by helping you to automate date skipping
- PkHex is a pokemon save file editor. 

To use both projects, you need to be playing on a modded switch (you can also be playing on a switch emulator like Yuzu for PkHex). At the time of writing, the most common way to mod your switch is by using a HwFly hardware mod, which is a Chinese clone of the SxCore project, now abandoned. The most common bootloader that people usually install is hekate and for the firmware, AtmosphereOS, which runs on top of the switch's HorizonOS. People typically don't attempt to connect to Nintendo's servers while running a custom firmwire as there's a high chance of the switch getting banned, but there are people who still do so. 

One of the things I've been wondering about in recent days is, what is the likelihood of the Pokemon I received over random trade being "hacked"? 

Sometimes it's easier to make a guess (eg. trainer ID is 31337, which is very unlikely to be a very nice number when it's randomly generated), while other times it's not. Some assumptions are more subtle. For example, as of the time of writing of this post, it's not possible to get any legendary pokemon from S/V in its shiny forms legally, but this hinges on the fact that you have to play the game to know this, or at least be up to date about what people are saying about it. In other cases, the game also complicates things. The existence of tera raids makes it possible to get many clones of the same pokemon legally (ie, pokemon has the same encryption constant and pid) as one can reset and re-play the same tera raid many times. In this case, it's possible for someone to find a shiny 6* tera raid and farm it many times with friends to get lots of the same pokemon with perfect IVs. 

Disclaimer: I didn't do any very hardcore end-game things in the other pokemon games I played, so I'm not sure if getting pokemon with perfect stats or qualities is as easy or harder.

In order to try to figure out more about Pokemon Internals, I decided to dig into the PkHex codebase to try to learn what has been datamined and is available publicly.

<a href="https://github.com/kwsch/PKHeX/blob/master/PKHeX.Core/PKM/HOME/GameDataCore.cs">GameDataCore.cs</a> describes all the known properties used by a single pokemon. There are also verifiers that can be found in the <a href="https://github.com/kwsch/PKHeX/blob/master/PKHeX.Core/Legality/Verifiers/">Verifiers folder</a>, which is quite useful as a Pokemon can be represented by so many fields and it's not immediately clear how some fields interact with each other. However, some values cannot be easily checked unless you can load your save file into PkHex. An example of this is checking the exact EV values of a Pokemon, as the game only allows you to see a pie-chart of the EV spread without showing you the actual values. Any stat value more than 252, and a total more than 510 are considered invalid. There's also a lot of other subtle checks that can be found just from looking through this folder. While the checks are quite comprehensive, one should still keep in mind that all these is based on data-mining the game and hence, there may be checks that are not covered yet. 

I think in the end I cannot say with definitive proof whether the pokemon I get from random trades are legit or not, but I can look at the Pokemon's stats and decide based on a rough approximation. 