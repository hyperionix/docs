---
layout: default
title: Blocking Operations Probe
nav_order: 2
parent: Packages Step By Step
permalink: package-2
---
# Blocking Operations Probe
Let's do something more interesting than printing the message. We want to write a probe which will be protect a file from being. Let's see what it takes and what features are available in package execution environment. Replace `File Delete.lua` file content with the following code.
```lua
-- Use sysapi library
setfenv(1, require "sysapi-ns")
local fs = require "fs.fs"

-- Define path to file we will protect
local PROTECTED_FILE = (fs.getTempDirectory() .. "protectedFile"):lower()

-- Use LuaJIT powerfull FFI to declare some C stuff we need
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
    -- PFILE_DISPOSITION_INFORMATION type we declared earlier
    local info = ffi.cast("PFILE_DISPOSITION_INFORMATION", fileInfo)
    if info.DeleteFile then
      deleteFile = true
    end
  elseif infoClass == ffi.C.FileDispositionInformationEx then
    -- we don't need to declare PFILE_DISPOSITION_INFORMATION_EX type as it is defined in sysapi library
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
  -- fileName is declared wthout `local` keyword so it could be accessed in onExit and onSkip handlers
  fileName = ffi.string(fileNameBuf):sub(5)

  if fileName:lower() == PROTECTED_FILE then
    -- block the operation so onSkip callback will be called instead original hooked function
    return {
      skip = true
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
        -- the function is called on onEntry handler returns skip = true
        -- here developer should do all to block the function execution
        context.r.rax = 0xC0000022
        context.p.IoStatusBlock.DUMMYUNIONNAME.Status = 0xC0000022
      end
    }
  }
}
```

Let's also change the test to this code that will try to delete two files. Deleting the first file should file due to our probe. Deleting the second file should succeed.

```lua
setfenv(1, require "sysapi-ns")
local File = require "file.File"
local fs = require "fs.fs"
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

Let's see what happened:
1. Test loaded package "File Delete". It means that the probe and all of its dependent hooks were loaded.
2. The test created and deleted two files in the temporary folder. One named `protectedFile` and one named `allowedFile`.
3. When the test tried to delete `protectedFile` our probe blocked the operation.
4. When the test tried to delete `allowedFile` out probe didn't react and the operation was permitted.

To block an operation the probe returned `skip = true` only when the correct file name was detected.

```lua
    return {
      skip = true
    }
```

We also had to tell the probe what value to return to the caller of `NtSetInformationFile` when we skipped it. This is done with the `onSkip` method. In this case we return `0xC0000022` which is the value of `STATUS_ACCESS_DENIED`. Alternatively, you can return `0` to pretend the operation was successful or any other error value you want.

```lua
      onSkip = function(context)
        -- the function is called on onEntry handler returns skip = true
        -- here developer should do all to block the function execution
        context.r.rax = 0xC0000022
        context.p.IoStatusBlock.DUMMYUNIONNAME.Status = 0xC0000022
      end
```

In the [next](package-3) step we will show how probe packages could generate [events](events) that can be analyzed in a central location.
