---
title: Basics of Hooking Functions
date: 2022-12-10 23:00:0 +0100
categories: [Reverse Engineering, Hooking, Hacking]
tags: [hooking, hook, hacking]     # TAG names should always be lowercase
---

## Overview

This blog post is to cover the absolute basics of hooking (on 32-bit architecture). These are used to ensure some of your code is run when a certain process of a function is run. For this blog post we are going to be using the scenario of your player character being damaged in game. So, we want to run our own code every time the player gets attacked, this allows us to do many fun things within a process.

Hooking is used quite a lot in certain malware (for instance to steal login information from a binary despite the password being hidden via asterisks), game hacking and also just adding functionality to standard processes.

***

## The general process of hooking a function

1. Find the function you want to hook into (whenever the function will be run, your own code will run).
2. Change certain instruction(s) to jump to your own code.
3. Run your own code and any collateral instruction(s) which have been overwriten for the jump instruction in step 2.
4. Make sure your own code jumps back to the instruction after your jump in the hooked function.

Here is an image example of a very basic hook into part of a function which subtracts damage from our player's health. In order to place our own code into a process we can inject a DLL into the process by using an injector ([Guided Hacking's Injector](https://guidedhacking.com/resources/guided-hacking-dll-injector.4/) is pretty good) and then using a function like [VirtualProtect()](https://learn.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualprotect) to change the protection of the function you want to hook to then to overwrite bytes for the jump to your own code.

![Simple Hook Example](/assets/simple-hook-example.png)

***

## Additional Resources

If you're interesting in learning how to do this rather than the theory behind it then I recommend watching/reading the following resources:

- [Video: C++ How to Detour / Hook Functions Tutorial](https://www.youtube.com/watch?v=jTl3MFVKSUM)
- [Video: How Function Hooking / Detouring Works](https://www.youtube.com/watch?v=b1ahj347pDc)
- [Guide: How to Hook Functions - Code Detouring Guide](https://guidedhacking.com/threads/how-to-hook-functions-code-detouring-guide.14185/)
