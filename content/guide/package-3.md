---
layout: default
title: Events Creation in Probe
nav_order: 3
parent: Packages Step By Step
permalink: package-3
---
# Events Creation in Probe
[Events](events) are some data that probes could created with developers defined information. Lets add an event generation everytime our probe will block the file removing operation. Also we will become familiar with [Entities](entity).

Lets change the probe as follows. We added a comments to mark what has changed.

```lua
setfenv(1, require "sysapi-ns")
local fs = require "fs"
-- Use hp library for create Entities
local FileEntity = hp.FileEntity
local ProcessEntity = hp.ProcessEntity

local PROTECTED_FILE = (fs.GetTempPath() .. "protectedFile"):lower()

ffi.cdef [[
  typedef struct _FILE_DISPOSITION_INFORMATION {
    BOOLEAN DeleteFile;
  } FILE_DISPOSITION_INFORMATION, *PFILE_DISPOSITION_INFORMATION;

  typedef struct _FILE_DISPOSITION_INFORMATION_EX {
    ULONG Flags;
  } FILE_DISPOSITION_INFORMATION_EX, *PFILE_DISPOSITION_INFORMATION_EX;

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
        context.p.IoStatusBlock.u.Status = 0xC0000022
      end
    }
  }
}
```
Run the test with hdk and you will see these two events received by the tool.

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

Open events.jsonl file and view the events:
```bat
code events.jsonl
```

In [the next](package-4) part we will show how to use `onExit` callback and how to inject the probe into real process for testing.
