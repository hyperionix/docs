---
layout: default
title: Quick Start
nav_order: 1
permalink: /
---

# Quick Start

## General information
* Hyperionix is a platform for development and deployment of modular OS monitoring and behavior modification packages.
* The platform gives you an ability to intercept almost all functions on target OS and write custom logic for the hook processing.
* Packages are small code snippets written on modified and improved version of <a href="https://luajit.org/" target="_blank">LuaJIT 2.1.0-beta3</a> and the packages could depend on each other.
* You can develop test and debug the packages locally and after easily tune, deploy and control it on your machines with installed Hyperionix Agent using <a href="https://admin.hyperionix.com/" target="_blank">Hyperionix Management Console</a>.
* You are free to use packages from <a href="https://github.com/topics/hyperionix-packages" target="_blank">community driven repositories</a> or create and support your own public or private repository.

## Hyperionix Development Kit

Hyperionix Development Kit (HDK) is a toolset for local development and testing of package. More information could be found [here](hdk).

NOTE: currently you can't have HDK and installed Hyperionix Agent on the same machine.

### Installing
1. Register at <a href="https://admin.hyperionix.com/" target="_blank">Hyperionix Management Console</a>.
2. Go to *Download* on the left menu and download HDK installer.
3. Install it on your machine.

### Initialization
* Install [HDK](hdk).
* Open cmd or powershell and go to the HDK installation path.
* Clone packages repository.
* Verify HDK installation.

```bat
cd /d %ALLUSERSPROFILE%\Hyperionix\hdk\
git clone git@github.com:hyperionix/packages.git packages/hyperionix/
.\bin\hdk --hdk-init
.\bin\hdk --verify all
.\bin\hdk --run-test all
```

So you are ready to [write your first package](quick-package)

