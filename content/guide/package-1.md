---
layout: default
title: Writing Simple Probe
nav_order: 1
parent: Packages Step By Step
permalink: package-1
---

# Writing Simple Probe

## Prepare development environment
We recommend using <a href="https://code.visualstudio.com/" target="_blank">vscode</a> editor but you any other text editor should work too.
* Complete [HDK installation and initialization steps](index#installing) if you haven't done it yet.
* Clone <a href="https://github.com/hyperionix/sysapi" target="_blank">sysapi</a> repository. It is a helper library to simplify low level Windows API access. Library documentation could be found <a href="/sysapi/index.html" target="_blank">here</a>.
```bat
git clone https://github.com/hyperionix/sysapi.git --branch develop sysapi/
```
* Open Hyperionix development workspace in vscode. In the right bottom corner you'll see vscode extensions recommendations. We recommend installing all of them for the easiest development experience.
```bat
code .\.vscode\hyperionix.code-workspace
```

## Writing hook package
Let's write the first [hook package](pages/hook-details). This type of packages is used to declare a place to setup pysical hook.

Create a file for the hook definition and edit it.
```bat
mkdir .\packages\my\hooks\win32\MyNtDeleteFile\ 
code .\packages\my\hooks\win32\MyNtDeleteFile\MyNtDeleteFile.lua
```
Paste and save the following snippet.
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
You will notice this hook defines:
* A name that can be referenced later.
* A target function symbol. Specifically this hook targets `NtDeleteFile` in `ntdll.dll`.
* Full function decleration so the hook knows which function arguments to get and what to return.

We have now declared a function we want to intercept and its arguments. Next we will verify the package with `hdk` utility:
```bat
.\bin\hdk --verify MyNtDeleteFile
```
```
Verify package MyNtDeleteFile... OK 
```
## Writing probe package
Our first [probe package](pages/probe-details) will define logic built on the top of the hook package we just defined. It will print a message for every file deletion function the hook catches.

Create a file for the probe definition and edit it.
```bat
mkdir ".\packages\my\probes\File Delete\"
code ".\packages\my\probes\File Delete\File Delete.lua"
```
Paste and save the following snippet.
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
Next we will verify the package with `hdk` utility:
```bat
.\bin\hdk --verify "File Delete"
```
```
Verify package File Delete... OK 
```
This is what probe package definition looks like. It depends on two hooks: `MyNtDeleteFile` we've just written and `NtSetInformationFile` which is implemented in cloned hyperionix packages repository. The clone hyperionix packages already has a `NtDeleteFile` hook definition, but we wrote our here for the sake of the example.

`onEntry` is a package callback which will be executed in the context of the hook on the function entry. Detailed information about package callbacks could be found [here](package-callbacks). Currently we just print a message inside the callback. So if the Hook package is loaded all calls of `NtDeleteFile` syscall and `NtSetInformationFile` will call corresponding `onEntry` callback.
## Package test
Package testing is not required but we strongly recommend writing some tests to verify your probes work as planned. Writing a test now will help you to understand better how packages work.
```bat
code ".\packages\my\probes\File Delete\test.lua"
```
Paste and save the following snippet.
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
    -- create and delete success
    assert(createAndDeleteFile(fs.getTempDirectory() .. [[\allowedFile]]) == true)
    unloadPackage("File Delete")
  end
}
```
To run test use `hdk` with `--run-test`.
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

Let's see what happened. 
1. `hdk` found a package named `"File Delete"` and executed its `test.lua`.
2. It built all packages from `Packages` list.
3. It executed all cases declared as `Case` (we have only single one).
4. Inside `Main` test case `hdk` loaded `File Delete` package. You can consider this as analog of injecting the package into a process. After loading the package hooks `ntdll!NtDeleteFile` and `ntdll!NtSetInformationFile` are set.
5. We trigger one of these function calls and just output `Hello` message.

You can use [sysapi](sysapi) both in packages and tests.

<details>
  <summary>Windows Internals Reference</summary>
In Windows a file can be deleted in multiple ways but all of them lead to one of the following functions: `ntdll!NtDeleteFile` or `ntdll!NtSetInformationFile` with `FILE_DISPOSITION_DELETE` flag (actually there are can be more ways but let's consider only these two for now). In most cases like removing a file from Windows Explorer the second option is used but we also intercept ntdll!NtDeleteFile to cover all cases. 
</details>

In the [next](package-2) step we will improve the Probe and add feature to block target function execution.
