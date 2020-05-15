---
layout: default
title: Hooks
nav_order: 1
parent: Packages Details
permalink: hook-details
---

# Hook package
Such packages describe one or multiple (not implemented yet) places for hook setting. Hooks could be placed on (almost) all places of code but the most typically used case is a function prolog hooking. Hook declaration looks like:
```lua
Hook {
 name = "string",
 target = "string",
 decl = "string",
 convention = "string",
 onEntry = function(context)
 end,
 onExit = function(context)
 end
}
```
* `name` (**string**) - name of the hook, should be unique across all used packages repositories (this limitation will be fixed in future).
* `target` (**string** or **integer**) - hook target. Could have one of the following formats:
    - `"ModuleName!SymbolName"` - DLL or EXE name and its public or private (see [symbols support](symbols-support)) symbol name.
    - `"ModuleName!Offset"` - DLL or EXE name and offset relative to the module base address.
    - `Absolute address` - absolute virtual address value.
* `decl` (**string**, *optional*) - if the hook target is function it is useful to declare its prototype. It provides proper access to function arguments in `onEntry` and `onExit` callbacks.
* `convention` (**string**, *optional*) - if the hook target is a function you can specify its calling convention if it differs from default. `"stdcall"` (default for 32 bit apps), `"fastcall"` (default for 64 bit apps), `"cdecl"` and `"thiscall"` are valid values. 
* `onEntry` (**function**, *optional*) - callback which is called before the hooked function is executed. See [Package callbacks](probe-details#onEntry).
* `onExit` (**function**, *optional*) - callback which is called after the hooked function. [Package callbacks](probe-details#onExit).

Developers must typically use a probe package which depends on a hook for defining `onEntry` and `onExit` callbacks instead of writing it directly in hook packages. It provides better package scalability, allows to reuse hook packages for different probes and install multiple probes on top of a single hook. Use the callbacks inside a hook package only if you are sure that the hooks couldn’t be used by someone else or in multiple probes.

## Hook performance overhead
Results for Intel Core i7-7700HQ 2.80 GHz:
* Empty onEntry callback execution overhead: `~500 ns/call`
* Empty onEntry and onExit callbacks execution overhead: `~1100 ns/call`