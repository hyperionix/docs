---
layout: default
title: Blocking Operations Probe
nav_order: 2
parent: Packages Step By Step
permalink: package-2
---
# Blocking Operations Probe
Lets do something more interesting than printing the message. We want to write a probe which will be protect a file from removing. Let's see how it looks like and what features are available in packages execution environment.
```lua
-- Use sysapi library
setfenv(1, require "sysapi-ns")
local fs = require "fs"

-- Define path to file we will protect
local PROTECTED_FILE = (fs.GetTempPath() .. "protectedFile"):lower()

-- Use LuaJIT powerfull FFI to declare some C stuff we need
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

-- The handler is triggered everytime ntdll!NtSetInformationFile function will be called
local function NtSetInformationFile_onEntry(context)
  -- Determine if NtSetInformationFile was called for delete operation
  -- Access to all function arguments like they were defined in the corresponding hook
  local infoClass = context.p.FileInformationClass
  local fileInfo = context.p.FileInformation
  local deleteFile = false

  -- Easy use enum constants with FFI
  if infoClass == ffi.C.FileDispositionInformation then
    -- Type casts
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
    -- Leave the handler as it isn't file delete operation
    return
  end

  -- Fast memory buffer allocation 
  local fileNameBuf = ffi.new("CHAR[?]", 1024)
  -- Call external C functions 
  local success = ffi.C.GetFinalPathNameByHandleA(context.p.FileHandle, fileNameBuf, 1024, 0)
  if not success then
    return 
  end

  -- GetFinalPathNameByHandleA returns name with \\?\
  -- fileName is declared wthout `local` keyword so it could be accessed in onExit and onBlock handlers
  fileName = ffi.string(fileNameBuf):sub(5)

  if fileName:lower() == PROTECTED_FILE then
    -- block the operation so onBlock callback will be called instead original hooked function
    return {
      block = true
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
      onBlock = function(context)
        -- the function is called on onEntry handler returns block = true
        -- here developer should do all to block the function execution
        context.r.rax = 0xC0000022
        context.p.IoStatusBlock.u.Status = 0xC0000022
      end
    }
  }
}
```

Lets change the test also

```lua
setfenv(1, require "sysapi-ns")
local File = require "File"
local fs = require "fs"
local bor = bit.bor

Packages {
  "File Delete"
}

local function createAndDeleteFile(fileName)
  local f = File.create(fileName, CREATE_ALWAYS, bor(GENERIC_READWRITE, DELETE))
  if f then
    return f:delete()
  end
end

Case("Main") {
  case = function()
    loadPackage("File Delete")
    -- this operation will be blocked due to our probe
    assert(createAndDeleteFile(fs.getTempDirectory() .. [[\protectedFile]]) == false)
    assert(ffi.C.GetLastError() == 5)
    -- this operation will be completed successfully
    assert(createAndDeleteFile(fs.getTempDirectory() .. [[\allowedFile]]) == true)
    unloadPackage("File Delete")
  end
}

```

And run `hdk` tool

```bat
.\bin\hdk --run-test "File Delete"
```
```
dbg: Run case [Main]
dbg: Package [File Delete] has been loaded successfully, id = 127
dbg: Package [File Delete] has been unloaded successfully
Received events:
----------------------------------
```

In the [next](package-3) step we will show how probe packages could generate an [Events](events)