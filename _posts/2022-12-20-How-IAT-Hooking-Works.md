---
title: How IAT Hooking Works
date: 2022-12-20 15:15:0 +0100
categories: [Reverse Engineering, Hooking, Hacking]
tags: [hooking, hook, hacking, IAT, ILT, IDT]     # TAG names should always be lowercase
---


## Overview

This method of hooking is done by changing the address of the function you want to hook in the IAT so it jumps to your code rather than the function.

***

## What is IDT, IAT & ILT?

The IDT is an acronym for 'Import Directory Table', this contains an array of `IMAGE_IMPORT_DESCRIPTOR` structs. This is usually found at the beginning of the `.idata` section of a binary (sometimes you may find it in `.rdata`). These structs are used for each imported DLL to the process. The `IMAGE_IMPORT_DESCRIPTOR` structs are laid out like so:
```cpp
typedef struct _IMAGE_IMPORT_DESCRIPTOR {
    union {
        DWORD Characteristics; // 0 for terminating null import descriptor
        DWORD OriginalFirstThunk; // RVA to the ILT/INT
    } DUMMYUNIONNAME;
    
    DWORD TimeDateStamp; /* 0 if not bound,
       -1 if bound, and real date\time stamp
       in IMAGE_DIRECTORY_ENTRY_BOUND_IMPORT (new BIND)
       O.W. date/time stamp of DLL bound to (Old BIND) */
    DWORD ForwarderChain; // -1 if no forwarders
    DWORD Name;
    DWORD FirstThunk; // RVA to IAT (if bound this IAT has actual addresses).
} IMAGE_IMPORT_DESCRIPTOR;
typedef IMAGE_IMPORT_DESCRIPTOR UNALIGNED *PIMAGE_IMPORT_DESCRIPTOR;
```

The main things in the above struct that are mainly used for IAT Hooking at the `OriginalFirstThunk` (The relative virtual address to the ILT/INT), `Name` (name of the DLL) and `FirstThunk` (the relative virtual address to the IAT).

![Relationship of the Import Directory IAT, and ILT](/assets/IDT-ILT-IAT-diagram.png)

*General relationship of the Import Directory, IAT and ILT*

The ILT/INT is an acronym for either Import Lookup Table or Import Name Table, respectively. Both of which are names which are used interchangeably. The is a table of function names which have been imported into the process by the DLL. As just mentioned above the `IMAGE_IMPORT_DESCRIPTOR` contains the address of the ILT/INT.

![ILT/INT Example](/assets/ILT-INT-diagram.png)

*Example of the ILT/INT*

IAT stands for Import Address Table, so it pretty much does what it says on the tin; it's a table of addresses for each imported function. However, on disk the IAT is identical to the ILT (so it just holds the names of the functions), but at runtime the loader overwrites the IAT with the addresses of those functions.

![IAT & ILT on disk and on runtime](/assets/IAT-diagram.png)

*IAT & ILT on disk and at runtime*

I believe this is all you really need to know to understand how this hooking technique works.

***

## How does IAT Hooking work?

This is relatively simple and has been used in various bits of malware for various purposes, some of which are:
- Stop a function from running
- Extract information from the function
- Simply just running additional code whenever the original function is called

!!! How to find IAT and function

![Example of how the IAT looks after being hooked](/assets/Before-Hook-After-Hook.png)

*Example of how the IAT looks after it has been hooked to jump to your own code*

Once you have found the IAT and the function to hook, the process is the following:
1. Change the address of the function in the IAT to the address of your own code, by doing this whenever that function is called it will go to your own code rather than the actual function.
2. This will then run your own code and then you can jump to the original function to run it.

***

## Issues with IAT Hooking

1. It is quite easy to detect this type of hooking as each imported function comes from another binary (example: `kernel32.dll`) and these have bounds in memory, so if the function address is changed in IAT to an address outside of the bounds then you know it's not the original function and something funky is going on.
2. If the import is imported via ordinal then you won't be able to do this type of hooking.
