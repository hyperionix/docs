---
layout: default
title: Development Techniques
permalink: development
# has_children: true
nav_order: 4
---

# Development Techniques

## Logging
There are two global loggers available in all [probe callbacks](probe-details#probe-callbacks): `LOG` and `DBG`. Both have the same methods but the first one outputs logs to a process default console and the second one outputs logs to system debug output. To view such logs you need to have a thirdparty tool. We recommend [DebugViewPP](https://github.com/CobaltFusion/DebugViewPP). 

Use logging only for testing and debugging, don't leave it in your final production ready code. 

**Usage:**
```lua
LOG:dbg("debug")
LOG:wrn("warning")
LOG:err("error")
LOG:pp({a = 5}) -- print a table
LOG:pp({a = 5}, 1) -- print a tables with depth
```
