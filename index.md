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
* You can develop test and debug the packages locally. Once ready you can easily tune, deploy and control packages to multiple machines running Hyperionix Agent using <a href="https://admin.hyperionix.com/" target="_blank">Hyperionix Management Console</a>.
* You can use packages from <a href="https://github.com/topics/hyperionix-packages" target="_blank">community driven repositories</a> or create and support your own public or private package repositories.

[Go To HMC](https://admin.hyperionix.com/){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 } [HMC Guide](hmc){: .btn .fs-5 .mb-4 .mb-md-0 }

## Hyperionix Development Kit

Hyperionix Development Kit (HDK) is a toolset for local development and packages testing. More information could be found in [the full guide](hdk).

NOTE: currently you can't have HDK and Hyperionix Agent installed on the same machine.

### Requirements
Before you begin make sure you have:
1. Windows 7 and above
2. A working e-mail address
3. [git](https://git-scm.com/downloads) installed

### Installing
1. Register at <a href="https://admin.hyperionix.com/" target="_blank">Hyperionix Management Console</a>.
2. Go to *Download* on the left menu and download HDK installer.
3. Install it on your machine.

### Initialization
* Install [HDK](hdk).
* Open cmd or powershell and go to the HDK installation path.
```bat
cd /d %ALLUSERSPROFILE%\Hyperionix\hdk\
```
* Clone packages repository.
```bat
git clone https://github.com/hyperionix/packages.git packages/hyperionix/
```
* Verify HDK installation.
```bat
.\bin\hdk --hdk-init
.\bin\hdk --verify all
.\bin\hdk --run-test all
```
Detailed information for hdk utility could be found in [the full guide](hdk).

You are now ready to [write your first package](guide).
