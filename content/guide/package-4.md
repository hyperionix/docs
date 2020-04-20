---
layout: default
title: onExit callback in Probe
nav_order: 4
parent: Packages Step By Step
permalink: package-4
---
# Using onExit callback
Besides `onEntry` and `onBlock` callbacks probes could have `onExit` callback which is called only if `onEntry` returns boolean `true` or a table with field `callExit = true`. Let's add to our probe generating an event for all successful file removing operations.

```lua
setfenv(1, require "sysapi-ns")
local fs = require "fs"
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
      block = true,
      events = {
        Event {
          name = "Attempt to delete protected file",
          fileName = FileEntity.fromPath(fileName),
          process = ProcessEntity.fromCurrent(),
          critical = true,
          foo = "bar"
        }:saveTo("file")
      }
    }
  end
end

-- Define onExit callback which checks status of the operation and generate an event in case of success.
local function NtSetInformationFile_onExit(context)
  if context.r.eax == 0 then
    return {
      events = {
        Event {
          name = "File Delete",
          fileName = FileEntity.fromPath(fileName),
          process = ProcessEntity.fromCurrent()
        }:saveTo("file")
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
      -- add onExit callback to the probe definition
      onExit = NtSetInformationFile_onExit,
      onBlock = function(context)
        context.r.rax = 0xC0000022
        context.p.IoStatusBlock.u.Status = 0xC0000022
      end
    }
  }
}
```
