---
layout: default
title: Events Creation in Probe
nav_order: 3
parent: Packages Step By Step
permalink: package-3
---
# Events Creation in Probe
[Events](events) are objects that probes can create with developer defined information. These events can then be sent to a central location from multiple agents. Let's take our existing probe and generate an event event time process creation is blocked. In this section we will become familiar with [entities](entity) and [event channels](event-channels).

Let's change the probe as follows. We added a comments to mark what has changed.

```lua
setfenv(1, require "sysapi-ns")
local FilePath = require "file.Path"
-- Use hp library for create Entities
local ProcessEntity = hp.ProcessEntity
-- EventChannel is used to control where events will be stored
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

          -- Create an event
          Event(
            -- the event name
            "Attempt to start forbidden process",
            -- list of the event attributes
            {
              processPath = imagePath.full,
              parentProcess = ProcessEntity.fromCurrent()
            }
            -- where to save the event
          ):send(EventChannel.file)
        end
      end
    }
  }
}
```
Notice we added `Event` function call in case process creation is blocked. All generated events must be created using this function. An event can have any number of attributes that help describe it. We can control where each event goes with `:send(EventChannel.<channel>)` event method. More information about event channels could be found [here](events#EventChannel). For now we'll use `:send(EventChannel.file)` to demonstrate events being logged into a file on the system.

```lua
  Event(
    "Event1",
    {
      foo = 1,
      bar = "data"
    }
  ):send(EventChannel.file)

  Event(
    "Event2",
    {
      foo = 2,
      bar = "data"
    }
  ):send(EventChannel.splunk)
```

Events can also be combined to create bigger events. For example, you may want to combine an event for a certain file being downloaded from the Internet with an event for a process being created from that same file. You can then create a composite event noting a suspicious process. We will cover this later when we discuss ESM.

Next, let's run the test with hdk and you will see this event received by the tool. `hdk --run-test` will collect events and display them directly in the console. Once the probe is deployed to an agent, the agent will send events to a central location instead of printing to console.

```bat
.\bin\hdk --run-test "My Process Created"
```
```
dbg: Run case [Main]
dbg: Package [My Process Created] has been loaded successfully, id = 127
dbg: Package [My Process Created] has been unloaded successfully
Received events:
Attempt to start forbidden process: 1
----------------------------------
```

Open `events.jsonl` file and view the last event on the last line:
```bat
code events.jsonl
```
The file is a valid JSON so you can easy beatify it e.g. with vscode (`Ctrl + P -> Format Document`) but note that it could be very large. You can erase the file content when you need to make room.

In [the next](package-4) part we will show how to use `onExit` callback and how to inject the probe into real process for testing.

