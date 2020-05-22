---
layout: default
title: Hyperionix Development Kit
permalink: hdk
nav_order: 5
---

# Hyperionix Development Kit
HDK is toolset for helping to develop, test and debug packages locally before push it into wildlife. The main part of the toolset is `hdk` utility which allows to verify packages syntax, run its tests inside itself and even inject packages to the other processes. You can check it in action in [first package guide](guide)

## HDK cheatsheet
Make load-unload of a package. 
```powershell
hdk.exe --verify <package-name>
```

Run test.lua of a package
```powershell
hdk.exe --run-test <package-name>
```

Inject a package into one or multiple processes with same name
```powershell
hdk.exe --inject <package-name> --process <process-name>
```

HDK initialization with custom agent name and Splunk address
```powershell
hdk.exe --hdk-init --agent-name myagent --splunk-addr tcp://mysplunk.com:9997
```

Use custom source based [sysapi](api#sysapi) version instead default one.
```powershell
hdk.exe ... --sysapi-path <path-to-sysapi-sources>
```
> NOTE: If you fork `sysapi` library and modified it you can't install it on your agents. You can only test it in HDK and create PR.

Add a custom packages search path
```powershell
hdk.exe ... --packages-path <packages-path>
```