---
layout: default
title: Quick Package
nav_order: 2
permalink: quick-package
---

# Quick Package
In this guide we implement a package for monitoring and blocking removing of some files. Detailed information about packages could be found [here](packages) but you can read it later. We recommend to use [vscode](https://code.visualstudio.com/) editor as we gonna to implement plugins for it in future. But you are still free to use any editor you like. In this case you have to replace `code` in the snippet.

```bat
set EDITOR=code
```

## Hook
```bat
mkdir .\packages\my\hooks\win32\MyNtDeleteFile\ 
%EDITOR% .\packages\my\hooks\win32\MyNtDeleteFile\MyNtDeleteFile.lua
```
```lua
Hook {
  name = "MyNtDeleteFile",
  target = "ntdll!NtDeleteFile",
  decl = [[
    NTSYSAPI NTSTATUS NtDeleteFile(
      POBJECT_ATTRIBUTES ObjectAttributes
    );
  ]]
}
```
So we have written a [hook package](pages/hook-details) where we have declared a function we want to intercept and its arguments. Verify the package with `package-verifier` utility
```bat
.\bin\package-verifier --verify MyNtDeleteFile
```
```
Verify package MyNtDeleteFile... OK 
```
## Probe
On the top of hook packages you can create [probe packages](pages/probe-details) where you can describe a logic and perform some manipulations
```bat
mkdir ".\packages\my\probes\File Delete\"
%EDITOR% ".\packages\my\probes\File Delete\File Delete.lua"
```
```lua
Probe {
  name = "File Delete",
  hooks = {
    {
      name = "MyNtDeleteFile",
      onEntry = function(args)
        print("Hello from MyNtDeleteFile")
      end
    },
    {
      name = "NtSetInformationFile",
      onEntry = function(args)
        print("Hello from NtSetInformationFile")
      end
    }
  }
}
```
```bat
.\bin\package-verifier --verify "File Delete"
```
```
Verify package File Delete... OK 
```
This how probe package definition looks like. It depends from two hooks: `MyNtDeleteFile` we've just written and `NtSetInformationFile` which isn't exist. But if you complete [HDK intialization](index) steps you should have hyperionix packages repository cloned and the hook is defined there. Actually, we also have `NtDeleteFile` hook definition there but lets assume we don't so we've written our own. `onEntry` is a function which will be executed in the context of the hook on the function entry. Currently we just print a message.
## Package Test
Package testing is unnecessary but we recommend doing it on the first steps. It will help you to understand better how packages work.
```bat
%EDITOR% ".\packages\my\probes\File Delete\test.lua"
```
```lua
local ffi = require"ffi"
local sysapi = hp.sysapi

hp.pt.Packages {
  "File Delete"
}

local function createAndDeleteFile(fileName)
  sysapi.CreateFile(fileName, sysapi.GENERIC_WRITE, sysapi.FILE_SHARE_DELETE, nil, sysapi.CREATE_ALWAYS, 0, nil)
  return ffi.C.DeleteFileA(fileName)
end

hp.pt.Case("Main") {
  case = function()
    load_package("File Delete")
    assert(createAndDeleteFile(sysapi.GetTempPath() .. "\\allowedFile") == 1)
    unload_package("File Delete")
  end
}
```
```bat
.\bin\package-verifier --run-test "File Delete"
```
```
dbg [hppt:0] Run case [Main]
dbg [hppt:0] Package [File Delete] has been loaded successfully, id = 117
Hello from NtSetInformationFile
dbg [hppt:0] Package [File Delete] has been unloaded successfully
Received events:
----------------------------------
```

Lets see what happened. 
1. The `package-verifier` find a package with name `"File Delete"` and executed its `test.lua`. 
2. It built all packages from `Packages` list.
3. It executed all cases declared as Case (we have only single one)
4. Inside `Main` test case `package-verifier` loaded `File Delete` package. You can consider this as analog of injecting the package into a process. After loading the packages hooks on `NtDeleteFile` and `NtSetInformationFile` were set.
5. We trigger one of these function calls and just output `Hello` message.

<details>
  <summary>Windows Internals</summary>
  
On Windows a file could be deleted in a multiple ways but all of them are leads to the one of the following functions: `ntdll!NtDeleteFile` or `ntdll!NtSetInformationFile` with `FILE_DISPOSITION_DELETE` flag (actually there are can be more ways but lets consider only these two). In most cases like removing file from Windows Explorer the second one is used but we also intercept `ntdll!NtDeleteFile` to cover all cases. 
</details>

## Improve The Probe
Lets do something more interesting than printing the message. We want to write a probe which will be protect a file from removing and generate critical event about such attempts. On other cases the probe should generate and usual event with deleted file information. Under the cut the full source code with detailed comments. Replace the old probe source code with this and test it. You can also start from getting how it works.

```lua
-- TODO We will get rid if these 
local sysapi = hp.sysapi
local ffi = require "ffi"
local ProcessEntity = hp.entity.process

-- Define path to file we will protect
local PROTECTED_FILE = (sysapi.GetTempPath() .. "protectedFile"):lower()

-- Use LuaJIT powerfull FFI to declare some C stuff we need
ffi.cdef[[
  DWORD GetFinalPathNameByHandleA(
    HANDLE hFile,
    LPSTR  lpszFilePath,
    DWORD  cchFilePath,
    DWORD  dwFlags
  );

  typedef struct _FILE_DISPOSITION_INFORMATION {
    BOOLEAN DeleteFile;
  } FILE_DISPOSITION_INFORMATION, *PFILE_DISPOSITION_INFORMATION;

  typedef struct _FILE_DISPOSITION_INFORMATION_EX {
    ULONG Flags;
  } FILE_DISPOSITION_INFORMATION_EX, *PFILE_DISPOSITION_INFORMATION_EX;

  enum {
    FILE_DISPOSITION_DELETE = 1
  };
]]

local function NtSetInformationFile_onEntry(args)
  -- The handler is triggered everytime ntdll!NtSetInformationFile function will be called
  -- Determing if NtSetInformationFile was called for delete operation
  -- Access to all function arguments like they were defined in the correpsonding hook
  local infoClass = tonumber(args.p.FileInformationClass)
  local fileInfo = args.p.FileInformation
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

  -- Leave the handler as it isn't file delete operation
  if not deleteFile then
    return
  end

  -- Fast memory buffer allocation 
  local fileNameBuf = ffi.new("CHAR[?]", 1024)
  -- Call external C functions 
  local success = ffi.C.GetFinalPathNameByHandleA(args.p.FileHandle, fileNameBuf, 1024, 0)
  if not success then
    return 
  end

  -- GetFinalPathNameByHandleA returns name with \\?\
  -- fileName is declared wthout `local` keyword so it could be accessed in onExit and onBlock handlers
  fileName = ffi.string(fileNameBuf):sub(5)

  if fileName:lower() == PROTECTED_FILE then
    -- use Entities library to create special objects with detailed information about system objects
    -- here we want to know wich process trying to delete the protected file and obviously it is current process
    local process = ProcessEntity {
      handle = ffi.C.GetCurrentProcess()
    }

    -- return a table from the hander with information
    return {
      -- easy block operation (onBlock callback will be called)
      block = true,
      events = {
        -- generate critical event
        Event {
          name = "Attempt to delete protected file",
          fileName = fileName,
          process = process,
          critical = true,
          -- you can add any custom data to the event
          foo = "bar",
          -- this part of the event controls where the event will be saved
          control = {
            file = true
          }
        }
      }
    }
  else
    -- if file isn't protected than we want to get onExit handler executed
    return true
  end
end

local function NtSetInformationFile_onExit(args)
  -- the handler is executed when NtSetInformationFile_onEntry returned `true`
  -- you can access to CPU registers state 
  if args.r.rax == 0 then
    return {
      events = {
        -- create ordinary event that file deleted
        Event {
          name = "File Deleted",
          -- note that `fileName` isn't defined in the current handler but it was declared
          -- without `local` keyword in onEntry handler. All such variables are shared between 
          -- onEntry, onExit and onBlock handlers
          fileName = fileName,
          control = {
            file = true
          }
        }
      }
    }
  end
end

Probe {
  name = "File Delete",
  hooks = {
    {
      name = "MyNtDeleteFile",
      onEntry = function(args)
        print("Hello from NtDeleteFile")
      end
    },
    {
      name = "NtSetInformationFile",
      onEntry = NtSetInformationFile_onEntry,
      onExit = NtSetInformationFile_onExit,
      onBlock = function(args)
        -- the function is called on onEntry handler returns `block = true`
        -- here developer should do all to block the function execution
        args.r.rax = 0xC0000022
        args.p.IoStatusBlock.u.Status = 0xC0000022
      end
    }
  }
}
```

Lets change the test also

```lua
local ffi = require"ffi"
local sysapi = hp.sysapi

hp.pt.Packages {
  "File Delete"
}

local function createAndDeleteFile(fileName)
  sysapi.CreateFile(fileName, sysapi.GENERIC_WRITE, sysapi.FILE_SHARE_DELETE, nil, sysapi.CREATE_ALWAYS, 0, nil)
  return ffi.C.DeleteFileA(fileName)
end

hp.pt.Case("Main") {
  case = function()
    load_package("File Delete")
    assert(createAndDeleteFile(sysapi.GetTempPath() .. "\\protectedFile") == 0)
    assert(ffi.C.GetLastError() == 5)
    assert(createAndDeleteFile(sysapi.GetTempPath() .. "\\allowedFile") == 1)
    unload_package("File Delete")
  end
}
```

And run package-verifier

```bat
.\bin\package-verifier --run-test "File Delete"
```
```
dbg [hppt:0] Run case [Main]
dbg [hppt:0] Package [File Delete] has been loaded successfully, id = 127
dbg [hppt:0] Package [File Delete] has been unloaded successfully
Received events:
Attempt to delete protected file: 2
File Deleted: 1
----------------------------------
```
