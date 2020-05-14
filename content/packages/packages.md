---
layout: default
title: Packages Details
permalink: packages
has_children: true
nav_order: 3
---

# Packages
Hyperionix packages are modules written on modified and improved version of LuaJIT 2.1.0-beta3. These modules are loaded by Hyperionix Agent and used for getting information from the target system, changing its behaviour, monitoring its activity and generating events. 
## Packages Types
Pages could be one of the following types:
* [Lib](lib-details) - a helper library package;
* [Hook](hook-details) - describe a physical hook location;
* [Probe](probe-details) - describe a logic built on top of one or multiple hooks;
* [ESM](esm-details) (Entity State Machine) - describe a package with asynchronous post processing of all or certain events, used for filtering events, combining multiple events into bigger one etc;
* [Scheduled probe](scheduled-probes-details) - describe a callback which is executed within a given period, used for periodic collecting of system information.

## Packages Engines
In addition packages are executed by different engines:
* `sensor` - engine inside all processes in the target system. Every process has its own instances of packages which aren’t linked with others. In this engine hook and probe packages are executed.
* `service` - engine inside Hyperionix Service. Typically used for ESM and Scheduled Probes execution.

