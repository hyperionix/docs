---
layout: default
title: onExit callback in Probe
nav_order: 4
parent: Packages Step By Step
permalink: package-4
---
# Using onExit callback
Besides `onEntry` callback probes could have `onExit` callback which is called if it is defined and `onEntry` doesn't call `skipExitHook` or `skipFunction` context methods. Let's add to our probe generating an event for all successful process creation operations.

```lua
setfenv(1, require "sysapi-ns")
local FilePath = require "file.Path"
local ProcessEntity = hp.ProcessEntity
local EventChannel = hp.EventChannel

Probe {
  name = "My Process Created",
  hooks = {
    {
      name = "MyNtCreateUserProcess",
      ---@param context EntryExecutionContext
      onEntry = function(context)
        local imagePath = FilePath.fromUS(context.p.ProcessParameters.ImagePathName)
        if imagePath.basename:lower() == "notepad" then
          context.r.eax = STATUS_ACCESS_DENIED
          context:skipFunction()
          Event(
            "Attempt to start forbidden process",
            {
              processPath = imagePath.full,
              parentProcess = ProcessEntity.fromCurrent()
            }
          ):send(EventChannel.file)
        end
      end,
      ---@param context ExitExecutionContext
      onExit = function(context)
        -- Create one more event if a process was created successfully
        if context.retval == STATUS_SUCCESS then
          Event(
            "Process Created",
            {
              newProcess = ProcessEntity.fromHandle(context.p.ProcessHandle[0]),
              parentProcess = ProcessEntity.fromCurrent()
            }
          ):send(EventChannel.file, EventChannel.splunk)
        end        
      end
    }
  }
}
```
We have defined an `onExit` handler that generates an event only when the hooked function was actually successful. This way we can collect events only when a process was actually created. 

Finally, run the test:
```bat
.\bin\hdk --run-test "My Process Created"
```

You will see something like:
```
Received events:
Attempt to start forbidden process: 1
My Process Created: 1
```
Here we generated an event for blocked process creation and another event for successful process creation.
