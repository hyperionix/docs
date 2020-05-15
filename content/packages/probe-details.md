---
layout: default
title: Probes
nav_order: 2
parent: Packages Details
permalink: probe-details
---

# Probe package
Probe is a package built on top of one or multiple hooks and which describes a logic of the hook processing.

Probes declaration looks like:
```lua
Probe {
 name = "string",
 hooks = {
   {
     name = "string",
     onEntry = function(context)
     end,
     onExit = function(context)
     end
   },
   ...
 }
}
```

* `name` (**string**) - name of the probe, should be unique across all used packages repositories (this limitation will be fixed in future).
* `hooks` (**array**) - list of hooks the probe depends on. All hooks should be accessible for user (e.g. repositories with the hooks should be added and synchronized in HMC). 

The main part of probes is probes callbacks.

## Probe callbacks
Probe callbacks are `onEntry` and `onExit` functions in hook or probe declaration. The callbacks are executed before and after hooked function execution. They are main places where developers describe logic of hook processing. Both callbacks shares the same environment. It means that if a non-local variable will be defined in `onEntry` it is also possible to reference it in `onExit`. 

In probe callbacks you can do the following: 
* get and modify CPU registers state;
* get and modify hooked function formal parameters;
* create [entities](entities) and [events](events);
* block hooked function execution;
* use [sysapi](api#sysapi) library or [LuaJIT FFI](https://luajit.org/ext_ffi.html) to do almost all you can imagine.


### onEntry
The callback is used typically for analyzing function formal parameters and making a decision if onExit or target function should be skipped. Control is accomplished using the `context` callback parameter which has an `EntryExecutionContext` type. This object has the following methods:
* `skipExitHook` - is called if you don't need to call `onExit` callback. The main idea here is to call `onExit` callback only when it is really necessary. Otherwise this call will have performance impact as calling of the callback isn't free.
* `skipFunction` - is called if you don't want the original hooked function to be executed at all. The main scenario here is blocking hooked function execution. Usually it is important to change the function parameters and return value to get correct behaviour from a caller. Note that you don't need to care about stack in this case. All will be done correctly if the hooked function calling convention and number of parameters were defined correctly.

Also the `context` object has the following fields:

* `r` - access to CPU registers state;
* `p` - access to hooked function formal parameters by name as they were defined in hook package.

### onExit
The callback is executed after the hooked function if `skipExitHook` wasn't called in `onEntry`. Call `onExit` if it is necessary. Inside the callback you have access to function output parameters and return value so you are able to handle it and event change it. Note that you also have access to all function parameters like in `onEntry` callback. The callback parameter `context` has type `ExitExecutionContext` which obviously doesn't have `skipExitHook` and `skipFunction` methods. The object has the following fields:
* `r` - access to CPU registers state;
* `pr` - `onEntry` CPU registers state;
* `p` - access to hooked function formal parameters by name as they were defined in hook package;
* `retval` - alias to `eax` register.

> NOTE: You must always keep in mind that all you’ll write in probe callbacks will be executed in a hooked function context and potentially could lead to undefined behaviour, deadlock or even process crash if you write something improperly. Be sure you understand what you are doing and write tests for your probe. 

### Examples

```lua
local function onEntry(context)
  -- ...
  -- block a hooked function execution if it tries to work with a `forbidden-file`
  if context.p.name == "forbidden-file" then
    -- return error status to the caller
    context.r.eax = -1
    context:skipFunction()
  end
  -- ...
end
```
```lua
local function onEntry(context)
  -- ...
  -- call onExit callback only if FILE_WRITE_ACCESS access flag is set
  if band(context.p.access, FILE_WRITE_ACCESS) == 0 then
    context:skipExitHook()
  end
  -- ...
end
```
For more detailed example check our [first probe tutorial](package-1).

For more hooks and probes check public repositories on [github](https://github.com/topics/hyperionix-packages).