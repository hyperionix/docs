---
layout: default
title: Writing Simple Probe
nav_order: 1
parent: Packages Step By Step
permalink: package-1
---

# Writing Simple Probe

## Prepare development environment
We recommend using <a href="https://code.visualstudio.com/" target="_blank">vscode</a> editor but any other text editor should work too.
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
Let's write the first [hook package](hook-details). This type of packages is used to declare a place to setup physical hook.

Create a file for the hook definition and edit it.
```bat
mkdir .\packages\my\hooks\win32\MyNtCreateUserProcess\ 
code .\packages\my\hooks\win32\MyNtCreateUserProcess\MyNtCreateUserProcess.lua
```
Paste and save the following snippet.
```lua
Hook {
  name = "MyNtCreateUserProcess",
  target = "ntdll!NtCreateUserProcess",
  decl = [[
    NTSTATUS NtCreateUserProcess(
        _Out_ PHANDLE ProcessHandle,
        _Out_ PHANDLE ThreadHandle,
        _In_ ACCESS_MASK ProcessDesiredAccess,
        _In_ ACCESS_MASK ThreadDesiredAccess,
        _In_opt_ POBJECT_ATTRIBUTES ProcessObjectAttributes,
        _In_opt_ POBJECT_ATTRIBUTES ThreadObjectAttributes,
        _In_ ULONG ProcessFlags,
        _In_ ULONG ThreadFlags,
        _In_opt_ PRTL_USER_PROCESS_PARAMETERS ProcessParameters,
        _Inout_ PPS_CREATE_INFO CreateInfo,
        _In_opt_ PVOID AttributeList
        );
  ]]
}
```
You will notice this hook defines:
* A name that can be referenced later.
* A target function symbol. Specifically this hook targets `NtCreateUserProcess` in `ntdll.dll`.
* Full function declaration so the hook knows which function arguments to get and what to return.

We have now declared a function we want to intercept and its arguments. Next we will verify the package with `hdk` utility:
```bat
.\bin\hdk --verify MyNtCreateUserProcess
```
```
Verify package MyNtCreateUserProcess... OK 
```
## Writing probe package
Our first [probe package](probe-details) will define logic built on the top of the hook package we just defined. It will print a message for every process creation function the hook catches.

Create a file for the probe definition and edit it.
```bat
mkdir ".\packages\my\probes\My Process Created\"
code ".\packages\my\probes\My Process Created\My Process Created.lua"
```
Paste and save the following snippet.
```lua
Probe {
  name = "My Process Created",
  hooks = {
    {
      name = "MyNtCreateUserProcess",
      ---@param context EntryExecutionContext
      onEntry = function(context)
        print("Hello from MyNtCreateUserProcess")
      end
    }
  }
}
```
Next we will verify the package with `hdk` utility:
```bat
.\bin\hdk --verify "My Process Created"
```
```
Verify package My Process Created... OK 
```
This is what probe package definition looks like. It depends on `MyNtCreateUserProcess` hook. The cloned hyperionix packages already has a `NtCreateUserProcess` hook definition and `Process Created` probe but we wrote our here for the sake of the example.

`onEntry` is a package callback which will be executed in the context of the hook on the function entry. Detailed information about package callbacks could be found [here](package-callbacks). Currently we just print a message inside the callback. So if the Hook package is loaded all calls of `NtCreateUserProcess` syscall will call corresponding `onEntry` callback.
## Package test
Package testing is not required but we strongly recommend writing some tests to verify your probes work as planned. Writing a test now will help you to understand better how packages work.
```bat
code ".\packages\my\probes\My Process Created\test.lua"
```
Paste and save the following snippet.
```lua
setfenv(1, require "sysapi-ns")
local Process = require "process.Process"

-- Package we want to test
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
    -- Load tested package
    package:load()
    -- Run a process
    assert(createAndTerminateProcess("notepad.exe") == true)
    -- Unload tested package 
    package:unload()
  end
}
```
To run test use `hdk` with `--run-test`.
```bat
.\bin\hdk --run-test "My Process Created"
```
```
dbg: Run case [Main]
dbg: Package [My Process Created] has been loaded successfully, id = 117
Hello from MyNtCreateUserProcess
dbg: Package [My Process Created] has been unloaded successfully
Received events:
----------------------------------
```

Let's see what happened. 
1. `hdk` found a package named `"My Process Created"` and executed its `test.lua`.
2. It built all packages created with `Package` function.
3. It executed all cases declared as `Case` (we have only single one).
4. Inside `Main` test case `hdk` loaded `My Process Created` package. You can consider this as analog of injecting the package into a process. After loading the package hooks `ntdll!NtCreateUserProcess` is set.
5. We trigger one of these function calls and just output `Hello` message.

You can use [sysapi](sysapi) both in packages and tests.

<details>
  <summary>Windows Internals Reference</summary>
In Windows a process can be created using multiple syscalls: `NtCreateProcess`, `NtCreateProcessEx` and `NtCreateUserProcess` but the first two of them are used only internally so all processes are created with `NtCreateUserProcess`. We meaningly don't handle `NtCreateProcess` and `NtCreateProcessEx` to simplify the example.
</details>

In the [next](package-2) step we will improve the Probe and add feature to block target function execution.
