---
layout: default
title: Writing Simple Probe
nav_order: 1
parent: Packages Step By Step
permalink: package-1
---

# Writing Simple Probe

## Prepare development environment
We recommend to use <a href="https://code.visualstudio.com/" target="_blank">vscode</a> editor but you are still free to use any editor you like.
* Complete HDK installation and initialization steps if you haven't done it yet ([instructions](index#installing))
* Clone <a href="https://github.com/hyperionix/sysapi" target="_blank">sysapi</a> repository. It is a helper library to simplify low level Windows API access. Library documentation could be found <a href="/sysapi/index.html" target="_blank">here</a>.
```bat
git clone git@github.com:hyperionix/sysapi.git --branch master sysapi/
```
* Open Hyperionix development workspace in vscode. In the right bottom corner you'll see vscode extensions recommendations. Its better to install it.
```bat
code .\.vscode\hyperionix.code-workspace
```

## Writing Hook package
Lets write the first [hook package](pages/hook-details). This type of packages is used to declare a place to setup pysical hook.
```bat
mkdir .\packages\my\hooks\win32\MyNtDeleteFile\ 
code .\packages\my\hooks\win32\MyNtDeleteFile\MyNtDeleteFile.lua
```
Paste the following snippet.
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
So here we are. We have declared a function we want to intercept and its arguments. Verify the package with `hdk` utility
```bat
.\bin\hdk --verify MyNtDeleteFile
```
```
Verify package MyNtDeleteFile... OK 
```
## Writing Probe package
[Probe packages](pages/probe-details) are describe logic built on the top of Hook packages. 
```bat
mkdir ".\packages\my\probes\File Delete\"
code ".\packages\my\probes\File Delete\File Delete.lua"
```
Paste the following snippet.
```lua
Probe {
  name = "File Delete",
  hooks = {
    {
      name = "MyNtDeleteFile",
      onEntry = function(context)
        print("Hello from MyNtDeleteFile")
      end
    },
    {
      name = "NtSetInformationFile",
      onEntry = function(context)
        print("Hello from NtSetInformationFile")
      end
    }
  }
}
```
```bat
.\bin\hdk --verify "File Delete"
```
```
Verify package File Delete... OK 
```
This is how Probe package definition looks like. It depends from two hooks: `MyNtDeleteFile` we've just written and `NtSetInformationFile` which implemented in cloned hyperionix packages repository. Actually we also have `NtDeleteFile` hook definition there but lets assume we don't so we've written our own. 

`onEntry` is a package callback which will be executed in the context of the hook on the function entry. Detailed information about packages callbacks could be found [here](package-callbacks). Currently we just print a message inside the callback. So if the Hook package is loaded all calls of `NtDeleteFile` syscall and `NtSetInformationFile` will call corresponding `onEntry` callback.
## Package Test
Package testing is unnecessary but we strongly recommend doing it on the first steps. It will help you to understand better how packages work.
```bat
code ".\packages\my\probes\File Delete\test.lua"
```
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
    assert(createAndDeleteFile(fs.getTempDirectory() .. [[\allowedFile]]) == 1)
    unloadPackage("File Delete")
  end
}
```
```bat
.\bin\hdk --run-test "File Delete"
```
```
dbg: Run case [Main]
dbg: Package [File Delete] has been loaded successfully, id = 117
Hello from NtSetInformationFile
dbg: Package [File Delete] has been unloaded successfully
Received events:
----------------------------------
```

Lets see what happened. 
1. The `hdk` find a package with name `"File Delete"` and executed its `test.lua`. 
2. It built all packages from `Packages` list.
3. It executed all cases declared as Case (we have only single one).
4. Inside `Main` test case `hdk` loaded `File Delete` package. You can consider this as analog of injecting the package into a process. After loading the packages hooks on `NtDeleteFile` and `NtSetInformationFile` were set.
5. We trigger one of these function calls and just output `Hello` message.

You can use [sysapi](sysapi) both in packages and tests.

<details>
  <summary>Windows Internals</summary>
  
On Windows a file could be deleted in a multiple ways but all of them are leads to the one of the following functions: ntdll!NtDeleteFile or ntdll!NtSetInformationFile with FILE_DISPOSITION_DELETE flag (actually there are can be more ways but lets consider only these two). In most cases like removing file from Windows Explorer the second one is used but we also intercept ntdll!NtDeleteFile to cover all cases. 
</details>

In the [next](package-2) step we will improve the Probe and add feature to block target function execution.

