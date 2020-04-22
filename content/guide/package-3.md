---
layout: default
title: Events Creation in Probe
nav_order: 3
parent: Packages Step By Step
permalink: package-3
---
# Events Creation in Probe
[Events](events) are objects that probes can create with developer defined information. These events can then be sent to a central location from multiple agents. Let's take our existing probe and generate an event event time file deletion is blocked. In this section we will become familiar with [entities](entity).

Let's change the probe as follows. We added a comments to mark what has changed.

```lua
setfenv(1, require "sysapi-ns")
local fs = require "fs.fs"
-- Use hp library for create Entities
local FileEntity = hp.FileEntity
local ProcessEntity = hp.ProcessEntity

local PROTECTED_FILE = (fs.getTempDirectory() .. "protectedFile"):lower()

ffi.cdef [[
  typedef struct _FILE_DISPOSITION_INFORMATION {
    BOOLEAN DeleteFile;
  } FILE_DISPOSITION_INFORMATION, *PFILE_DISPOSITION_INFORMATION;

  enum {
    FILE_DISPOSITION_DELETE = 1
  };

  DWORD GetFinalPathNameByHandleA(
    HANDLE hFile,
    LPSTR  lpszFilePath,
    DWORD  cchFilePath,
    DWORD  dwFlags
  );
]]

local function NtSetInformationFile_onEntry(context)
  local infoClass = context.p.FileInformationClass
  local fileInfo = context.p.FileInformation
  local deleteFile = false

  if infoClass == ffi.C.FileDispositionInformation then
    local info = ffi.cast("PFILE_DISPOSITION_INFORMATION", fileInfo)
    if info.DeleteFile then
      deleteFile = true
    end
  elseif infoClass == ffi.C.FileDispositionInformationEx then
    local info = ffi.cast("PFILE_DISPOSITION_INFORMATION_EX", fileInfo)
    if bit.band(info.Flags, ffi.C.FILE_DISPOSITION_DELETE) then
      deleteFile = true
    end
  end

  if not deleteFile then
    return
  end

  local fileNameBuf = ffi.new("CHAR[?]", 1024)
  local success = ffi.C.GetFinalPathNameByHandleA(context.p.FileHandle, fileNameBuf, 1024, 0)
  if not success then
    return
  end
  fileName = ffi.string(fileNameBuf):sub(5)
  if fileName:lower() == PROTECTED_FILE then
    return {
      skip = true,
      -- add list of events we want to emit
      events = {
        Event {
          name = "Attempt to delete protected file",
          -- create target file entity and process which attempt to delete file
          -- obviously it is current process. This will add detailed information
          -- about the file and the process to the event.
          fileName = FileEntity.fromPath(fileName),
          process = ProcessEntity.fromCurrent(),
          critical = true,
          foo = "bar" -- you can add any custom data to the event
        }:saveTo("file") -- save the event to file
      }
    }
  end
end

Probe {
  name = "File Delete",
  hooks = {
    {
      name = "MyNtDeleteFile",
      onEntry = function(context)
        print("Hello from NtDeleteFile")
      end
    },
    {
      name = "NtSetInformationFile",
      onEntry = NtSetInformationFile_onEntry,
      onSkip = function(context)
        context.r.rax = 0xC0000022
        context.p.IoStatusBlock.DUMMYUNIONNAME.Status = 0xC0000022
      end
    }
  }
}
```
Run the test with hdk and you will see these two events received by the tool. `hdk --run-test` will collect events and display them directly in the console. Once the probe is deployed to an agent, the agent will send events to a central location instead of printing to console.

```bat
.\bin\hdk --run-test "File Delete"
```
```
dbg: Run case [Main]
dbg: Package [File Delete] has been loaded successfully, id = 127
dbg: Package [File Delete] has been unloaded successfully
Received events:
Attempt to delete protected file: 1
----------------------------------
```

Open `events.jsonl` file and view the last event on the last line:
```bat
code events.jsonl
```
The file is a valid JSON so you can easy beatify it e.g. with vscode (`Ctrl + P -> Format Document`) but note that it could be very large. You can erase the file content when you need to make room.

In [the next](package-4) part we will show how to use `onExit` callback and how to inject the probe into real process for testing.

