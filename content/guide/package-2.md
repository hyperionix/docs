---
layout: default
title: Blocking Operations Probe
nav_order: 2
parent: Packages Step By Step
permalink: package-2
---
# Blocking Operations Probe
Let's do something more interesting than printing the message. We want to write a probe which will forbid `notepad.exe` process creation. Let's see what it takes and what features are available in package execution environment. Replace `My Process Created.lua` file content with the following code.
```lua
-- Use sysapi library
setfenv(1, require "sysapi-ns")
-- FilePath class
local FilePath = require "file.Path"

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
        end
      end
    }
  }
}
```

Let's also change the test to this code that will try to create two different processes. Creating the first should succeed. Deleting the second process should be blocked due to our probe.

```lua
setfenv(1, require "sysapi-ns")
local Process = require "process.Process"

local package = Package "My Process Created"

local function createAndTerminateProcess(path)
  local p = Process.run(path)
  if p then
    p:terminate()
    return true
  end
  return false
end

Case("Main") {
  case = function()
    package:load()
    -- Here we are able to run calc.exe process.
    assert(createAndTerminateProcess("calc.exe") == true)
    -- Running notepad.exe should be failed
    assert(createAndTerminateProcess("notepad.exe") == false)
    assert(ffi.C.GetLastError(), ERROR_ACCESS_DENIED)
    package:unload()
  end
}
```

And run `hdk` tool

```bat
.\bin\hdk --run-test "My Process Created"
```
```
dbg: Run case [Main]
dbg: Package [My Process Created] has been loaded successfully, id = 127
dbg: Package [My Process Created] has been unloaded successfully
Received events:
----------------------------------
```

Let's see what happened:
1. Test loaded package "My Process Created". It means that the probe and all of its dependent hooks were loaded.
2. The test tried to start two processes: `calc.exe` and `notepad.exe`
3. The first attempt was successful because `calc.exe` is allowed to run.
4. The second attempt failed because it tried starting notepad.exe and our probe blocked it as it's a forbidden process.

To block an operation the probe calls skipFunction when it detects a forbidden process name. This context method tells the hook to not call the actual hooked function. In this case this means the process will not be started if skipFunction is used.


```lua
  context:skipFunction()
```

We also had to tell the probe what value to return to the caller of `NtCreateUserProcess` when we skipped it. In this case we return `STATUS_ACCESS_DENIED` [NT Status](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-erref/596a1078-e883-4972-9bbc-49e60bebca55). Alternatively, you can return `STATUS_SUCCESS` to pretend the operation was successful or any other error value you want.

```lua 
  context.r.eax = STATUS_ACCESS_DENIED
```

In the [next](package-3) step we will show how probe packages could generate [events](events) that can be analyzed in a central location.
